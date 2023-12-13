# Links
https://proglib.io/p/git-cheatsheet

https://kbroman.org/github_tutorial/
---
# Usual workflow
## Create repo
- git init
- git add .
- git commit -m "Started"
- %%stuff on github website%%
- git branch -M main
- git remote add origin git@github.com:vsouth/study-api.git
- git push -u origin main
## Or download
- ?
## Saving changes
- git add .
- git commit -m "Fixed"
## Branches
- ?

---
# Notes
==DO NOT FORGET ABOUT .GITIGNORE!==
SSH key for private repos - https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent

---
# Commands
## To start 
- git init
%% stuff on github website %%
- git branch -M main
- git remote add origin git@github.com:vsouth/study-api.git
- git push -u origin main
## To save changes 
- git add .
- git commit -m "Started" # or smth like that
## To cancel changes
- git restore .
## To work with branches
- ?
## .gitignore
```
*.pyc
*.egg-info
# Настройки и хранение секретных данных 
/.env
```
## To download
- ?
- 