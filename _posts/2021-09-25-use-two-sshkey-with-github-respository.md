---
title: Use Two sshkey with Github Repository
date: 2021-09-25 01:30:00 +0800
categories: [Github]
tags: [github, tech]
# TAG names should always be lowercase
---
# Goal
If we want have two respository connecting to two folder for one account in linux server, we must have two sshkey.
# Operation
## 1. Create Two Project First
Here, we name them as `repo_A` and `repo_B` in one github account.

## 2. Generate Two SSHkey at linux server
* Make a folder saving data for ssh key if nonexist:
`mkdir ~/.ssh`
* Generate ssh keys respectively for `repo_A` & `repo_B`:
    * Repo_A: `$ssh-keygen -t rsa -C "your_email@example.com"`<br>
    This would generate SSH key called **id_rsa.pub & id_rsa**.
    * Repo_B: `$ssh-keygen -t rsa -f ~/.ssh/sshkeyB -C "your_email2@example.com"`<br>
    This generate SSH key called **sshkeyB.pub & sshkeyB**.<br>

All public keys (id_rsa.pub, sshkeyB.pub) and private key (id_rsa, sshkeyB) are saved in `~/.ssh/`. We should only tell others the public keys.

## 3. Write a Config 
To tell which SSH key is deployed at somewhere, we have to write `config` in `~/.ssh folder`:
Build config file: `vim config` @ the folder `~/.ssh/`
```config
Host Repo_A-github             // ssh 要連的代號名稱
    HostName github.com        //IP 或 Domain Name
    User git                   //使用者名稱
    IdentityFile ~/.ssh/id_rsa // ssh key（絕對位置）

Host Repo_B-github
    HostName github.com
    User git
    IdentityFile ~/.ssh/sshkeyB
```
Note that we write `User git` since we will push to github as the command line `git@github.com/<github_name>/<repo_name>.git`.<br>
Then execute `touch ~/.ssh/config`.

## 4. Add new key (sshkeyB) to the agent:
Execute `ssh-add ~/.ssh/sshkeyB`.

If it pops out the error message: *Could not open a connection to your authentication agent.*<br>
Execute `ssh-agent bash` first.

## 5. Add new key (sshkeyB) to github repository:
Copy the **public key** (A as example):
```console
$ cat ~/.ssh/id_rsa.pub
```
And copy what server returns.<br>
Then, go the github page, as `repo_A` -> [Settings] -> [Deploy keys] -> [Add deloy key] to paste the text.<br>
Same as the `repo_B`.


## 6. Connect Folder to Github Repository
In local, we have two folders called `folder_A` and `folder_B`. <br>
We should initialize `.git` first: (Just follow the given command lines from the new repository):<br>
Use `folder_A` to connect `repo_A` as example:
```console
$ git init
$ git add .
$ git commit -m "initialize"
$ git branch -M main
$ git remote add origin git@github.com:<user_name>/repoA.git
$ git push -u origin main
```
If fail at the command `git push -u origin main` in either:
A as example:
```console
$ git config --global user.name "<github_name>"
$ git config --global user.email <github_email>
```
Then check `folder_A/.git/config` or `folder_B/.git/config`. <br>
A as example:
```config
[core]
    repositoryformatversion = 0
    filemode = true
    bare = false
    logallrefupdates = true
[user]
    email = <github_email>
    name = <github_name>
[remote "origin"]
    url = git@github.com:neko2048/repo_A.git
    fetch = +refs/heads/*:refs/remotes/origin/*
```
If correct in both cases, then it should work.
# Reference
* [SSH key CONFIG setting](https://askie.today/lets-set-ssh-config/): <br>https://askie.today/lets-set-ssh-config/
* [SSH key CONFIG setting](https://blog.alantsai.net/posts/2016/03/ssh-config-ssh-agent-passphrase-management): <br> https://blog.alantsai.net/posts/2016/03/ssh-config-ssh-agent-passphrase-management
* [SSH add key error](https://hoohoo.top/blog/to-resolve-ssh-add-appears-could-not-open-a-connection-to-your-authentication-agent/): <br>https://hoohoo.top/blog/to-resolve-ssh-add-appears-could-not-open-a-connection-to-your-authentication-agent/
* [Connecting to GitHub with SSH](https://docs.github.com/en/authentication/connecting-to-github-with-ssh): <br>https://docs.github.com/en/authentication/connecting-to-github-with-ssh
* [Best way to use multiple SSH private keys on one client [closed]](https://stackoverflow.com/questions/2419566/best-way-to-use-multiple-ssh-private-keys-on-one-client): <br>https://stackoverflow.com/questions/2419566/best-way-to-use-multiple-ssh-private-keys-on-one-client
* [Token](https://blog.csdn.net/weixin_41010198/article/details/119698015): <br>https://blog.csdn.net/weixin_41010198/article/details/119698015