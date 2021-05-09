---
title: "Git 命令"
date: 2019-10-05T20:02:27+08:00
summary: "记录常用 Git 命令"
tags: git
---
记录常用 Git 命令

## 配置用户名和邮箱

### 添加用户和邮箱

```bash
$ git config --global user.name "your_name"
$ git config --global user.email "example@example.com"
```

### 查看已设置的用户

```bash
$ git config user.name
$ git config user.email
```

## 创建新的仓库

```bash
$ echo "# Introduction" >> README.md
$ git init
$ touch .gitignore
$ git add .
$ git commit -m "log"
$ git remote add origin [repository_address]
$ git push -u origin master
```

## 远程仓库相关

```bash
$ git clone [repo]                              #克隆到本地
$ git remote -v                                 #查看远程仓库
$ git remote add [name] [repo]                  #添加远程仓库
$ git remote rm [name]                          #删除远程仓库
$ git remote set-url --push [name] [new_repo]   #修改仓库
$ git pull [remote_repo] [local_branch]         #拉取远程仓库
$ git push [remote_repo] [local_branch]         #推送到远程仓库
```

## 添加删除缓存区

```bash
$ git add [file]                #添加将要合并的文件
$ git add .                     #添加所有
$ git rm --cached [file]        #删除缓存的文件
$ git rm -r --cached .          #清楚所有缓存的文件
```