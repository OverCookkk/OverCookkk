---
title: Git命令基础
---

## 创建版本库
首先创建一个文件夹，然后cd到该文件夹下，输入以下命令：
```text
git init
```
当前目录下会多了一个.git的目录，这个目录是Git来跟踪管理版本库的。


## 版本穿梭

### 工作区和暂存区
提交一条信息命令：

```text
git add 文件名	（*代表全部文件）
git commit -m ""
```

git add把文件添加进去，实际上就是把**工作区**文件修改添加到**暂存区**；
git commit是一次性提交暂存区的全部更改，实际上就是把**暂存区**的所有内容提交到**本地的当前分支（master）**。

![](https://raw.githubusercontent.com/OverCookkk/PicBed/master/blogImg/git%20img4.png?token=AVYDJJBU26LC65VC4VPKESTBKGXDM)


### 版本回退与撤销修改

穿梭前，用`git log`可以查看提交历史，以便确定要回退到哪个版本。（只能看到当前版本之前的历史版本）
要重返未来，用`git reflog`查看命令历史，以便确定要回到未来的哪个版本。（可以看到全部版本）

（1）当改乱了工作区某个文件的内容，想直接丢弃工作区的修改时，用命令`git checkout -- file`。
（2）当不但改乱了工作区某个文件的内容，还添加到了暂存区时（add），想丢弃修改，分两步，第一步用命令`git reset HEAD <file>`，就回到了（1），第二步按（1）操作。
（3）已经提交了不合适的修改到版本库时，想要撤销本次提交，使用命令`git reset --hard commit_id`，不过前提是没有推送到远程库。

<u>注：`git checkout -- file`也可以用来恢复工作区删除了的文件，前提是该文件已经提交到版本库。</u>


## 远程仓库



## 分支

### 创建分支
```text
创建分支: git branch 分支名
切换分支: git checkout 分支名
查看所有分支: git branch -a
分支合并: git merge 分支名 (把分支合并到当前分支)
```

创建分支后，由于新建立的分支还未在git上，所以push的时候，需要用命令：git push --set-upstream origin hexo

### 删除分支

```text
首先使用git branch -a查看当前所有分支。
删除本地分支：git branch -d 分支1[,分支名2,分支名3...]
删除远程分支: git push origin --delete 分支名1[,分支命2,分支名...]
```

### 缓存分支修改
​		有时候，我们可能会同时在几个分支上进行开发，在切换分支的时，如果当前分支有修改，这个时候功能还没有全部开发完成，你并不想马上提交到版本库的话，我们可以使用Git提供的缓存修改的命令，把这部分修改暂时缓存起来，切换回这个分支的时候，再把它取出来。

```text
缓存修改命令：git stash 
```
【注：这个命令会把提交到暂存区，就是把使用git add提交之后的代码缓存起来,对未经Git管控的文件修改并不做缓存】
```text
从缓存中恢复修改并删除缓存内容: git stash pop
查看缓存列表:git stash list
```



## 拉取

&emsp; git分为本地仓库和远程仓库，我们一般情况都是写完代码，commit到本地仓库（生成本地仓的commit ID，代表当前提交代码的版本号），然后push到远程仓库（记录这个版本号）。
&emsp; 假设我们本地仓库的 master 分支上 commit ID =1 ，orign/master中的commit ID =1 ;这时候远程仓库别人更新了github ogirn库中master分支上的代码，新的代码版本号commit ID =2 ,那么在github上 orign/master的commitID=2，然后我们要更新代码。git流程示意图如下：
![](https://raw.githubusercontent.com/OverCookkk/PicBed/master/blogImg/git%20img1.png?token=AVYDJJGKX2WHNUIMLL6KWX3BKGIQM)

### git fetch
- 使用git fetch更新代码，本地的库中master的commitID不变，还是等于1。但是与git上面关联的那个orign/master的commit ID变成了2。这时候本地相当于存储了两个代码的版本号，还要通过merge去合并这两个不同的代码版本，如果这两个版本都修改了同一处的代码，这时候merge就会出现冲突，然后我们解决冲突之后就生成了一个新的代码版本。
- 这时候本地的代码版本可能就变成了commit ID=3，即生成了一个新的代码版本。
![](https://raw.githubusercontent.com/OverCookkk/PicBed/master/blogImg/git%20img2.png?token=AVYDJJF4VVCENQRGVRESUW3BKGIS2)

### git pull
- 使用git pull会将最新代码更新到本地，head和remotes中的commitID同时被更新为远程分支最新的commitID。
![](https://raw.githubusercontent.com/OverCookkk/PicBed/master/blogImg/git%20img3.png?token=AVYDJJBVHY2YON2NZABOZQ3BKGIYI)