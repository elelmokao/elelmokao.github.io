---
title: Git Cheat Sheet
date: 2023-12-03 00:00:00 +0800
categories: [CheatSheet, Git]
tags: [git, cheatSheet]
# TAG names should always be lowercase
---
Update Date: 2023-12-03

## Refresh .gitignore
```bash
vim .gitignore
git rm -r --cached .
git add .
git commit -m "comments"
```
If there is some files you commited and not push yet, and you want to add this type of files to `.gitignore`.\\
> Ref: <https://sigalambigha.home.blog/2020/03/11/how-to-refresh-gitignore/>

## Git Stack
```bash
git stash       # store current 
git stash list  # browse git stash list
git stash pop   # recover stash
git stash drop  # clean newest stash
git stash clear # clean all stashes
```
If you want to store some files that modified but do not want to commit and change to other branches.\\
> Ref: <https://gitbook.tw/chapters/faq/stash>