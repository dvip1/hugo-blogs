+++
title = 'Getting Started with Git'
date = 2026-02-01T12:52:20+05:30
draft = true
+++


## Initial Process

### 1. GitHub Account

Make sure you already have a GitHub company account. If you don’t ask your supervisor to provide it to you.  

### 2. Install Dependencies

Check if you have git installed from this command in PowerShell. 
`git —version`

If it’s not installed, then install it as such

`winget install --id Git.Git -e --source winget` 

To check if it’s installed, run the command `git --version` once again in a new PowerShell window or tab.

### 3. Sign in

The next step is to sign in using your credentials.

Continue on PowerShell.  

- configure name: 
`git config --global user.name "firstname lastname"`
- configure email (same as GitHub mail):
`git config --global user.email "company-mail@email.com"`

And then sign in to your GitHub account itself. 
Now there are multiple ways to do this. 

**GITHUB CLI (METHOD 1):**

One of the easiest is to install GitHub cli on PowerShell. 

With the command below:
`winget install -e --id GitHub.cli`

Open up a new window of PowerShell or a new tab of your shell.

`gh --version` if it spits out version without giving any error then it’s successfully installed.

and login by `gh auth login`

It would give you several prompts, and follow the instruction down below if you can’t navigate. 

select  > GitHub

      > HTTPS

      > ‘Y’ for “Authenticate Git with your GitHub credentials" prompt
      > Login with a web browser 

copy paste the link and login with the otp given on your terminal window. 
And there you have it, no complication of gpg keys or ssh keys. 

GITHUB CLI (gh) gives you flexibility, as it allows you to quickly switch between accounts, quick  login and logout and a great user friendly experience if you use GitHub. 

**SSH (METHOD 2):**

This is another great method. But this is more advanced method, so avoid if you don’t already know what SSH is. 

Follow the link below, it explains things pretty well:
[https://dev.to/thevinitgupta/connecting-to-github-with-ssh-windows-2oj3](https://dev.to/thevinitgupta/connecting-to-github-with-ssh-windows-2oj3)

## Usage 

### Git Clone

Once you’re logged in and ready to go. Clone the project you want to work on, using `git clone <url>` command. 
Open the repository that you want to clone on your browser,
copy the URL.

Cloning to new directory 

`git clone <copied-url>`

this would by default make a new directory containing the project. 

Cloning in current directory 

`git clone <copied-url> .`

would clone the project in current directory without creating a new one. Helpful when you have multiple instances of the same project and want to name each folder differently.

### Git Branches

Here’s some basics about git branches: [Link](https://www.w3schools.com/git/git_branch.asp)

List Branches using:

`git branch`

to get updated list of branches from origin:

`git fetch origin`

to create new branch: 

`git checkout -b {new-branch-name}`

to change branch use 

`git checkout {branch-name}`

### Basic Workflows

I’d list a basic workflow of how you’d work
once you have cloned and changed to the required branch. 

1. Pull changes from your peers. This would merge code from the given branch to yours.
`git pull {branch-name}` 
2. Check the status, which gives bunch of useful information
`git status`
3. Stage your current changes 
`git add {file-name or directory-name or .}` , ‘.’ stages all the changes.
4. Commit your changes
`git commit -m “{describe changes you made}”`
5. Push the changes 
`git push origin {branch-name}` 

Using graphical interface (vscode):
vscode has this features built-in, do check them out if you like to do things graphically: 
[https://code.visualstudio.com/docs/sourcecontrol/overview](https://code.visualstudio.com/docs/sourcecontrol/overview)
They are much more simple and easy to use. But knowing the commands might come in handy when you don’t have access to vscode. 

## Explore more

Git and GitHub has ton of features that I haven’t covered here. Please checkout the link below if you want to learn them in detail. Learning basic Merge and merge conflicts are essential for working in team environment.

[https://www.w3schools.com/git/default.asp?remote=github](https://www.w3schools.com/git/default.asp?remote=github)
