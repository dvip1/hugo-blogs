---
title: "From Prompt to Dashboard: Building a Secure Text-to-SQL Analytics Bot"
date: 2026-05-15
draft: false
---

Let me define the problem statement first:  

> Users/managers constantly ask for slightly customized metrics or weird date ranges that standard dashboards don't cover. Hardcoding a new UI chart for every edge-case query is unscalable. 
  
Sounds simple enough problem, right ?
  
We could just make some chatbot API.     
  
Well it is simple if you ignore the security risk that it could be, plus how could a chatbot possibly provide visualizations?

I was asking these questions to myself as well.  
So let's start from the very beginning and walk our way through it.  

### Choosing a LLM
As this was experimental project, I wanted something that is cheap and dependable if the project works.  
I could've gone with something like "Open Router" which can provide multiple models and I could switch between them.  
But I choose Google, specifically because of their open gemma4 models.   
Those models weren't only cheap but also very reliable with my testing.  
Not only that but Gemini can provide JSON formatted Data in the format I want which is a game changer but competitors like openai can also do the same. 

### Sanitizing the data
For this I wrote a strict prompt which should block most prompt injection, I also wrote some regex which blocks harmful prompts such as `DROP`, `INSERT` and such.  
But regex alone isn't very reliable.  
I made a new user which only has read access to the tables that are useful to LLM, so even if bad actor manages to skip every check before, they still won't be able to run anything malicious.  

### Storing the Data
So we've integrated chatbot, and we've sanitized the data. Now how do we store it ?   
I designed a structure which is very extensible.  
  
First a Chat Room table to store History of all the chats
```sql
CREATE  TABLE chat_bot.chatroom (  
id UUID PRIMARY  KEY  DEFAULT gen_random_uuid(),   
  
chat_title TEXT,  
  
created_at TIMESTAMP  NOT  NULL  DEFAULT NOW(),  
  
updated_at TIMESTAMP  NOT  NULL  DEFAULT NOW(),  
  
is_share BOOLEAN  NOT  NULL  DEFAULT  FALSE
);
```
And storing messages itself I did this: 
```sql
CREATE  TABLE chat_bot.chatmessage (  
id UUID PRIMARY  KEY  DEFAULT gen_random_uuid(),  
  
room_id UUID NOT  NULL,  
  
role  VARCHAR(10) NOT  NULL,  
  
content TEXT NOT  NULL,  
  
sql_query TEXT,  

  
created_at TIMESTAMP  NOT  NULL  DEFAULT NOW(),  
  
which_type TEXT,  
  
is_shared BOOLEAN,  
  
is_saved BOOLEAN  DEFAULT  FALSE
)
```  
- Here role is either user/bot
-  content is the text output from bot or request from user
- sql_query: is query from response from bot
- we'll later discuss is_saved & which_type

### Visualizing the Data

This is everything that client sees, for this I won't be going into UI but rather how I'm getting information.  
  
 `which_type` from our database discussion gives us type of data to show in client.  
 say, a table, a line graph, a pie chart, all of which comes directly from gemma.  
   
 And the is_saved bit also very interesting,  
 This flag allows user to place all the things that they gathered from the bot and save it in the dashboard and because I'm saving everything in SQL Query, every chart and tables are up to date.

### Future Scope & Conclusion 
While gemma-26B is relatively cheap, it still costs real money and isn't very easy to scale up. So as we gather more data relying on vector database would be much more feasible and would be very cheap. But that's a far thought for now.  
Reach my DMs if you have anything particular to ask!


