---
layout: post
title: "Git初学不得不知的一些命令"
description: "Git 的命令行其实已经足够强大了，这里举例一些常用的"
category: scm
tags: [git]
---
{% include JB/setup %}
## 常用命令
{% highlight html %}
git add *.txt // 添加需要提交的文件到暂存区
git commit -m “test” *.txt // 提交暂存区的文件
git commit -a -m 'test' // 直接提交到仓库
git rm readme // 从仓库中删除文件
git rm --cached readme.txt // 从仓库中删除文件， 并保留开发目录下的文件 
git rm \*.log // 递归删除当前目录下面所有*.log的文件
git rm *.log // 仅删除当前目录下面的*.log文件
git checkout path/to/file // 只是工作目录中文件被删除，直接checkout该文件即可
git checkout pre_commit_id path/to/file //删除了文件并已commit，可分根据历史版本id恢复
git mv file_from file_to // 改名
gitk // 图形化工具
git commit --amend // 修改最近提交的评述
git remote -v // 显示远程仓库列表
git fetch [remote-name] // fetch 命令只是将远端的数据拉到本地仓库,并不自动合并到当前工作分支 
git pull // 从远端仓库抓取数据，并合并到工作目录中当前分支
git push origin master // 推送数据到远程仓库 
git diff // 比较工作目录中当前文件和暂存区域快照之间的差异
git diff --cached // 比较已经暂存起来的文件和上次提交时的差异
git diff commit_id_1:path/to/file commit_id_2:path/to/file // 比较两次提交历史中指定文件的差异
git branch -av // 查看本地仓库的分支信息
git fetch origin // 获取远程origin所有仓库信息
git merge origin/dev // merge
git push origin dev // 上传到origin/dev远程仓库分支
git checkout dev // 切换本地分支
git checkout master // 切换主干
git branch -d dev // 删除分支
git branch -D dev // 删除分支(强制执行)
git remote -v // 显示远程仓库信息
git branch dev // 创建分支
git branch --merged // 查看哪些分支已被并入当前分支
git branch --no-merged // 查看尚未合并的分支
{% endhighlight %}