---
title: "Git 常用命令总结"
date: 2022-04-21
tags: ["Git",""]
description: "记录一下常用的 git 命令和操作流程"
draft: false
---

> [Git Documentation](https://git-scm.com/book/zh/v2)
>
> [Git 教程 - 廖雪峰](https://www.liaoxuefeng.com/wiki/896043488029600)

# Git 常用命令

- `git init` 初始化本地 git 环境
- `git clone <repository>` 克隆一份代码到本地仓库
- `git pull` 把远程仓库代码更新到本地，等于 `git fetch + git merge`
- `git pull --rebase origin master` 强制把远程仓库的代码更新到当前分支上面
- `git fetch` 把远程库的代码更新到本地
- `git add .`  把本地改动过的文件添加到暂存区中
- `git commit -m '<commit message>'` 把暂存区中的修改提交到本地库
- `git push` 把本地库的修改提交到远程库中
- `git push origin <branch name>` 提交一个分支到远程库中
- `git branch -r/-a` 查看远程分支 / 全部分支
- `git checkout master/bugfix` 切换到某个分支
- `git checkout -b bugfix` 新建 bugfix 分支
- `git checkout -d bugfix` 删除 bugfix 分支
- `git merge master` 假设当前在 bugfix 分支上，把 master 分支上的修改同步到 bugfix 分支上
- `git merge tool` 调用 merge 工具
- `git stash` 把未完成的修改保存起来
- `git stash list` 查看所有保存列表
- `git stash pop` 恢复本地分支到缓存状态
- `git blame <file name>` 查看某个文件每一行的修改记录，谁在什么时候修改的
- `git status` 查看当前分支有哪些修改
- `git log` 查看当前分支上面的日志信息
- `git diff` 查看当前没有 add 的内容
- `git diff --cached` 查看已经 add 但是没有 commit 的内容
- `git diff HEAD` 上面两个命令显示内容的合并
- `git reset --hard HEAD` 撤销本地修改

# 团队协作 Git 流程

## 克隆新项目，完成功能并提交

1. `git clone <repository>` 克隆代码仓库
2. `git checkout -b <branch name>` 新建分支
3. `<modify your code>` 完成功能的开发，代码的修改
4. `git add .` 把修改添加到暂存区
5. `git commit -m '<commit message>'` 提交修改到 bugfix 分支
6. `<review 代码>`
7. `git checkout master` 切换到 master 分支
8. `git pull` 更新代码
10. `git merge <branch name>` 将新建分支合并到 master
11. `git push origin <branch name>` 把新建分支的代码 push 到远程仓库

## 正在新功能分支开发，需要紧急修复 bug

适用于新功能正在开发还不想提交的情况

1. `git add .` 将当前代码添加暂存
2. `git stash` 保存修改
3. `git checkout -b <bugfix>` 新建 bugfix 分支
4. `git pull --rebase origin master` 主分支代码更新到当前分支
5. `<fix the bug>` 修复 bug
6. `git add .` 添加暂存
7. `git commit -m '<commit message>'` 提交代码
8. `git push origin <bugfix>` 推动 bugfix 分支到远程仓库
9. `git checkout <new feature branch>` 回到新功能开发分支
10. `git stash pop` 恢复修改
11. `<continue develop>` 继续开发

