+++
title = "How I Handled 20k Writes per Second on a $12 Lightsail Server"
date = 2026-11-23T23:30:00+05:30
draft = false
description = "How Dvip Patel built a Go and SQLite logger server that ingests encrypted readings from 20,000+ devices per second on a $12 AWS Lightsail box (2 vCPU, 2 GB RAM) and survives kill -9."
summary = "A write-heavy backend on one tiny box: group commit into append-only segment files, an in-memory hot tier, once-a-minute compaction into SQLite blocks, and crash recovery that survives kill -9."
tags = ["Go", "SQLite", "Performance", "Databases", "Backend"]
categories = ["Projects", "Architecture"]
keywords = ["Dvip Patel", "20k writes per second", "SQLite write performance", "group commit", "Go high throughput ingest", "AWS Lightsail benchmark", "append-only log", "survive kill -9"]

[cover]
image = "/images/posts/20k-writes-per-second-12-dollar-lightsail-server/architecture.png"
alt = "Architecture diagram: logger devices send packets to an ingest handler, a write queue and a single writer goroutine that appends to segment files and device buffers, which compact into SQLite blocks every 60 seconds"
hidden = true
+++

On a simulated $12 AWS Lightsail box (2 vCPUs, 2 GB of RAM), my logger server takes encrypted readings from **22,500 devices every second** and passes every correctness check. It's one Go binary and one SQLite file. No message queue, no cache server, no managed database.

- **Challenge:** [LoggerBench challenge spec](https://github.com/dvip1/loggerbench_challenge/blob/master/template.md)
- **Code:** [dvip1/super_optimized_logger_server](https://github.com/dvip1/super_optimized_logger_server)
- **Stack:** Go, SQLite (WAL mode)

This post covers the first design that fell over, the one that replaced it, and the small decisions that each removed a bottleneck.

## The challenge

Thousands of data loggers each send one encrypted reading per second, forever, and their owners read them back in a dashboard. Build the backend on one tiny box and make it carry as many loggers as possible. A judge program signs up users, registers devices, floods the server with readings, then reads every one back to check nothing was lost, corrupted or duplicated.

The constraints:

- **2 burstable vCPUs.** Sustained use is only 20% per vCPU; going above that spends CPU credits.
- **2 GB of RAM** for the OS, the server and any caching.
- **A 60 GB SSD** with a few thousand IOPS. Stored raw, 1,000 devices write about 23 GB a day.
- **It must survive `kill -9`.** The judge kills the server mid-load and checks that every acknowledged reading is still there after a restart.

## Picking the stack

At first I wasn't sure how to approach this at all. One thing I did know: I couldn't afford an interpreted language. Its runtime needs memory of its own, and I had 2 GB for everything. So it had to be a compiled language, and the options were Go, C and Rust.

I could probably have squeezed the most out of C, but I picked Go. It's easy to work with, and it still let me do nearly everything I wanted without managing memory by hand.

SQLite was the obvious choice for storage. I wanted plain SQL for the read side, and I didn't want a separate database process eating RAM.

## The packet

Every device does the same thing once a second: it sends a 301-byte packet over plain HTTP. The server has to authenticate and decrypt it, reject replays and tampering, store it durably, and answer `204` only once the reading can be read back.

`Content-Type: application/octet-stream`, fixed 301 bytes, all numbers little-endian:

| Offset | Size | Field |
| --- | --- | --- |
| 0 | 1 | version (`1`) |
| 1 | 8 | srno (the 16 hex chars as 8 raw bytes) |
| 9 | 12 | nonce |
| 21 | 280 | AES-128-GCM ciphertext (264) + tag (16) |

The ciphertext holds a timestamp plus 64 `float32` sensor values. One packet is tiny. But at 20,000 devices that's 20,000 decrypts, 20,000 durable writes and 20,000 HTTP round trips every second, on a box that makes each of those expensive. The question I kept coming back to was how few CPU cycles, disk operations and bytes of memory each reading could cost.

## First attempt: one SQLite row per reading

My first thought was: it's a write-heavy app, so just write everything as fast as possible. That idea turned into batching commits every 20 ms with SQLite in WAL mode and `synchronous = OFF`.

Every reading became a row in an `ingest_log` table, indexed on `(srno, ts)` so compaction and crash recovery could find a device's readings.

It looked fine at low load and collapsed at high load. The index was the problem. A batch of readings from thousands of devices touches roughly one index page per device, so each 20 ms commit rewrote far more pages than the data needed. `strace` counted 2.2 `pwrite64` calls per reading, and SQLite wrote about 15 times more pages than the readings themselves.

| Simulated Lightsail, 15,000 devices, 5 min, crash test | SQLite `ingest_log` | Segment files (current) |
| --- | --- | --- |
| Verdict | FAIL (39% of readings accepted) | PASS |
| Ingest p99 | 4.9 s | 221 ms |
| Query p99 | 4.7 s | 32 ms |
| Restart until healthy | 51 s | 0.25 s |
| Disk write operations | 2,021,032 | 27,841 |
| Storage per reading | 641 B | 275 B |

{{< aside "Where I learned this" >}}Fly.io's [SQLite internals: B-trees](https://fly.io/blog/sqlite-internals-btree/) explains why every index page you touch turns into a page write.{{< /aside >}}

The lesson: the disk is slow because of how many separate writes you make, and an index on the hot path multiplies them. This was my first real deep dive into SQLite internals.

## The architecture

The design splits storage into a hot tier and a cold tier. New readings go to an append-only file and an in-memory buffer. Once a minute they move to SQLite as one block per device.

![Architecture diagram: every reading is one append, and SQLite sees one block per device per minute. Logger device → ingest handler (decrypt, replay check) → write queue (Go channel, 65,536 deep) → writer goroutine (one write() per batch, then 204). The writer appends to segment files and adds to per-device buffers, which compact into SQLite blocks every 60 s. Dashboard reads come from SQLite up to the watermark plus buffered readings after it.](/images/posts/20k-writes-per-second-12-dollar-lightsail-server/architecture.png)

The top row runs for every packet and never touches SQLite. The bottom row runs once a minute, in the background, and is the only place SQLite gets written.

## Group commit: one writer, one `write()` per batch

SQLite only allows one writer at a time, so pushing lots of concurrent commits at it was never going to work. Funnelling everything through a single writer was the obvious move.

The replacement for `ingest_log` is a set of append-only segment files. Each record is a fixed 272 bytes (serial, timestamp, 64 sensors), written in arrival order, with no index and no B-tree.

Only one goroutine ever appends. Request handlers decrypt and validate a reading, push it onto a channel, and wait. The writer loop:

1. Block until one reading arrives.
2. Take everything else already waiting on the channel, without waiting for more.
3. Append the whole batch to the current segment with a single `write()`.
4. Add each reading to its device's in-memory buffer and its `latest` pointer.
5. Release every waiting handler, which then replies `204`.

```go
case row := <-writeQueue:
    batch = drainQueue(append(batch[:0], row))
    // encode every reading into one buffer, then:
    err := seg.append(out) // ONE write() for the whole batch
```

There's no timer and no batch size to tune. At low load a batch is one reading and latency is a few hundred microseconds. Under load, readings pile up while the previous `write()` runs, so the next batch is bigger and the cost per reading falls. That's why server CPU only rose from about 115% to 130% of a core while load went from 5,000 to 21,888 readings per second on my 4-core test machine.

The channel holds 65,536 readings. If it ever fills, handlers block on the send, which slows ingest down instead of dropping data.

## Keeping SQL off the hot path

A data packet never touches SQLite. At startup the server loads every device into a map keyed by its 8-byte serial number. Each entry holds a ready-made AES-128-GCM cipher, the owner and token, and the last accepted timestamp. Revoked tokens live in an in-memory set that's updated whenever a token is revoked.

So the ingest handler is a short, fixed sequence:

1. Read exactly 301 bytes into a stack buffer and check the size and version.
2. Look up the device by serial number.
3. Under that device's lock: check the token isn't revoked, decrypt in place, reject the reading if its timestamp isn't newer than the last one, then record the new timestamp.
4. Queue the reading for the writer, still under the lock, so each device's readings reach the writer in timestamp order.

Two details matter here. Building the cipher once per device instead of once per packet removes the AES key schedule from the hot path. And `aead.Open` decrypts into the ciphertext's own bytes, so a reading costs no extra allocation for its plaintext.

The lock is per device, so 20,000 devices never contend with each other. It also makes decrypt, replay check and timestamp update one step: a re-registration can't swap the key halfway through, and two copies of the same packet can't both pass the replay check.

## Compaction and the watermark

Segment files are fast to write but useless to query: finding one device's readings would mean scanning everything. So once a minute, or after 5 seconds without data, the server compacts:

1. **Rotate.** On the writer goroutine, open a new segment and note how many readings each device has buffered. Every one of them is in the older segments.
2. **Write blocks.** In the background, insert one row per device into `reading_blocks`: about 60 readings packed into one blob, in transactions of 500 devices.
3. **Trim.** After each commit, drop those readings from the device's buffer and move its **watermark** to the block's last timestamp.
4. **Clean up.** Delete the old segments and checkpoint the WAL so it doesn't grow.

Compacting every device at once is what lets the hot tier get away with no index: readings never have to be found or deleted one device at a time. And `reading_blocks` gets one row per device per minute instead of 60, so its `(srno, end_ts)` index stays small.

The watermark is what keeps reads correct while compaction runs. Readings at or before it are in SQLite; readings after it are in memory. A `/readings` query copies the buffer and the watermark together under one lock, reads blocks only up to the watermark, then appends the buffered readings. A block committed during the query only holds readings newer than the copied watermark, and those are already in the copied buffer. No reading is returned twice or skipped, and a query never blocks the writer.

## Surviving `kill -9`

The challenge requires surviving `kill -9`, and that turned out to be a much easier bar than surviving power loss. Once `write()` returns, the data is in the kernel's page cache, and the kernel keeps it when the process dies. So a reading is acknowledged as soon as its batch's `write()` returns, with no `fsync`. SQLite runs with `synchronous = OFF` for the same reason.

That makes durability almost free, as long as every step stays safe if the process dies in the middle of it:

- **A crash mid-append.** If a `write()` fails, the segment is truncated back to its last whole record. A partial record at the end of a file was never acknowledged, so replay stops there.
- **A crash mid-compaction.** A block is committed before its readings leave the buffer, and segments are deleted only after all blocks commit. Either the block exists or the segment does.
- **Restart.** Load devices, set each device's watermark from its newest block, then replay every segment, skipping records at or before the watermark. Committed readings are never loaded twice, and uncommitted ones are never lost.

The HTTP server starts listening only after replay finishes, so the health check can't report ready before every acknowledged reading is back. Restart until healthy is 0.25 s, compared with 51 s for the SQLite-only version.

## Results

On a faithful copy of the Lightsail box, the server passes up to **22,500 devices** while it has CPU credits, and up to **1,500** once the credits run out.

Passing means at least 99% of expected readings accepted, ingest p99 under 500 ms, query p99 under 1 s, and every audited reading back intact.

| Condition | Devices | Outcome |
| --- | --- | --- |
| With CPU credits | 22,500 | PASS |
| With CPU credits | 25,000 | FAIL: CPU pinned, ~22,700 readings/s max, latency over 1 s |
| Credits exhausted (20% per vCPU) | 1,500 | PASS, p99 139 ms |
| Credits exhausted (20% per vCPU) | 2,000 | FAIL, p99 580 ms |

*LoggerBench judge, 1-minute runs on a simulated Lightsail box, fresh server per run, November 2026.*

## Challenges and tradeoffs

**Testing setup.** One underrated challenge was setting up the networking to actually push this many requests at the server. My first local runs were capped at 21,888 readings per second, and that turned out to be the server's 100 Mbit network port, not the server. So I replicated Lightsail's hardware constraints locally on my laptop: server and judge on one machine over localhost, with the server in a cgroup shaped like Lightsail:

- 2 hyperthreads of one core, clocked down to 2.5 GHz
- 1.6 GB of memory (2 GB minus the OS), no swap
- An ext4 disk limited to 3,000 IOPS and 125 MB/s
- A CPU cap calibrated against a real Lightsail run: 2,000 devices used about 50% of a core there

The numbers matched the real box surprisingly closely.

**The filesystem mattered more than I expected.** On btrfs, compaction bursts stalled every write and 10,000 devices failed. On ext4 the same server passed 10,000 with a 10 ms p99.

**Keep-alive connections.** The benchmark relies on persistent HTTP keep-alive connections. Each logger reuses its TCP connection, so the server isn't paying for a handshake on every reading. Devices that reconnect for every packet would cost a lot more per request.

**No TLS on ingest.** I skipped TLS on the ingest endpoint since the packets are already encrypted and authenticated with AES-GCM. TLS still works on the other endpoints (sign-up, dashboard reads) with only a slight dip in performance.

**Durability is scoped to `kill -9`.** Skipping `fsync` is safe for a process crash, but a power loss or kernel panic could lose the last few seconds of acknowledged readings. Guarding against that would take a lot of `fsync` calls, and that cost is exactly what this design avoids.

**Burstable CPU.** 22,500 devices is the number while CPU credits last. Sustained, the box carries about 1,500. If this ran for real, that's the number to plan around.

## What's next

- **Compression.** Blocks are stored raw today. The planned codec finds the smallest decimal scale that reproduces each sensor column exactly, then stores zigzag deltas, bit-packed, with raw 32-bit fallbacks for NaN and -0. Sensor data changes slowly, so this should cut storage several times over.
- **Memory for idle devices.** After compaction a device's buffer is emptied but keeps its allocation, about 28 KB per device, even after the device goes silent. Freeing empty buffers and setting `GOMEMLIMIT` would keep memory flat as the device count grows.
- **HTTP timeouts.** The server uses the default `http.ListenAndServe` with no timeouts, which is fine for a benchmark but not for the open internet.

## What I learned

1. **Count operations, not bytes.** The disk and the CPU care about how many `write()` calls, syscalls and allocations you make. Batching cut disk write operations from about 2 million to about 28 thousand.
2. **Let batches size themselves.** Taking whatever is queued gives you low latency at low load and big batches at high load, with nothing to tune.
3. **Know exactly what durability you're promising.** Surviving `kill -9` and surviving power loss are different promises, and the gap between them is a lot of `fsync` calls.
4. **Measure on the real box, or a faithful copy of it.** My first local numbers were capped by a 100 Mbit port, and btrfs behaved differently from ext4. The server never changed; the measurements did.

The full code is on GitHub at [dvip1/super_optimized_logger_server](https://github.com/dvip1/super_optimized_logger_server).
