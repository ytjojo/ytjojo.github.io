---
title: git命令与场景使用
date: 2017-04-25 16:19:20
categories: git
tags: [git,git命令,git教程,git详解,git学习笔记]
---

![](https://segmentfault.com/img/remote/1460000006774229)

## 简写说明

	git checkout -b # b -> browse
	git branch -r # r -> remote
	git branch -a # a -> all
	git config -l # l -> list
	git commit -m # m -> message

## git 配置

	#查看目前git仓库中有哪些配置。
	
	git config --list

	#全局设置用户名和邮箱

	git config --global user.name "Your Name"
	git config --global user.email "email@example.com"

	#系统级配置,/etc/gitconfig

	git config --system user.name "Your Name"

	git config --system user.email "email@example.com"

	#仓库级配置,~/.gitconifg

	git config --local user.email "email@example.com"

## git 仓库初始化

	#在当前目录下新建一个git仓库
	git init
	# 新建一个目录，将其初始化为Git代码库
	git init androidDemo
	git remote add origin git@github.com:YotrolZ/helloTest.git

	# 下载一个项目和它的整个代码历史
	git clone https://github.com/521xueweihan/git-tips.git
	
	# 在androidDemo目录下载一个项目和它的整个代码历史
	git clone [url] androidDemo
	
	#修改远程仓库的地址，把<URL>替换成新的url地址。
	git remote origin set-url <URL>

	#三种方式修改远程仓库地址
	#1. 修改命令
	git remte origin set-url URL

	#2.先删后加
	git remote rm origin 
	git remote add origin git@github.com:Liutos/foobar.git 

	# 3. 直接修改config文件
	
	
## 增加/删除文件

	# 添加指定文件到暂存区
	git add test.txt

	# 添加多个文件，中间用空格隔开
	git add test1.txt test2.txt

	#添加带有路径的文件
	git add 2017/04/hexo和githubpage搭建个人博客.md

	#添加当前目录下所有文件到暂存区
	git add .
	#同样功能的命令，添加当前目录所有文件到暂存区
	git add --all
	
	#以“块”形式暂存你的改动
	#添加每个变化前，都会要求确认
	#对于同一个文件的多处变化，可以实现分次提交
	git add -p [path/file]

	#删除工作区文件，并将这次删除放入暂存区
	git rm [filename]

	#删除暂存区文件，停止追踪文件，但是该文件会保留在工作区
	git rm --cached [file]
	
### 关于`git add -p filename`这个命令

![](http://ww1.sinaimg.cn/large/c1ff19eagy1fezwr1a0ayj20go06bt8u.jpg)

途中看到我更改了三个区块，删除了import和main两行，增加了aaa这一行，增加了abc和ddd这两行。图中有这一行

	
	Stage this hunk [y,n,q,a,d,/,s,e,?]? 

让我选择命令，详细的命令解释如下。我输入了s，表示分割，最后分割为三块，

* y - stage this hunk
* n - do not stage this hunk
* q - quit; do not stage this hunk or any of the remaining ones
* a - stage this hunk and all later hunks in the file
* d - do not stage this hunk or any of the later hunks in the file
* g - select a hunk to go to
* / - search for a hunk matching the given regex
* j - leave this hunk undecided, see next undecided hunk
* J - leave this hunk undecided, see next hunk
* k - leave this hunk undecided, see previous undecided hunk
* K - leave this hunk undecided, see previous hunk
* s - split the current hunk into smaller hunks
* e - manually edit the current hunk
* ? - print help

然后每块的操作都要询问我，让我输入命令

![](http://ww1.sinaimg.cn/large/c1ff19eagy1fezwxp4l6kj20ev07q74g.jpg)

如图可以看到最后提交到暂存区的只有第一块


	
	

## 重命名/移动文件和文件夹

	# 改名文件，并且将这个改名放入暂存区
	git mv [file-original] [file-renamed]

	#移动文件到文件夹
	git mv xx1.js js/
	
	#将文件夹移动到另一个文件夹下,其实相当于文件夹重命名
	git mv app/ androidDemo/app
	


## 提交

	# 提交暂存区到仓库区
	$ git commit -m [message]
	
	# 提交暂存区的指定文件到仓库区
	$ git commit [file1] [file2] ... -m [message]
	
	# 提交工作区自上次commit之后的变化，直接到仓库区，
	# Git 就会自动把所有已经跟踪过的文件暂存起来一并提交，跳过 git add。
	$ git commit -a
	
	# 提交时显示所有diff信息
	$ git commit -v
	
	# 使用一次新的commit，替代上一次提交
	# 如果代码没有任何新变化，则用来改写上一次commit的提交信息
	$ git commit --amend -m [message]
	
	# 重做上一次commit，并包括指定文件的新变化
	$ git commit --amend [file1] [file2] ...

## 分支

	# 列出所有本地分支
	git branch
	
	# 列出所有远程分支
	git branch -r
	
	# 列出所有本地分支和远程分支
	git branch -a
	
	# 新建一个分支，但依然停留在当前分支
	git branch [branch-name]
	
	# 新建一个分支，并切换到该分支
	git checkout -b [branch]
	
	# 新建一个分支，指向指定commit
	git branch [branch] [commit]
	
	# 新建一个分支，与指定的远程分支建立追踪关系
	git branch --track [branch] [remote-branch]
	
	# 本地没有master分支，新建master 分支，与远程master分支建立追踪关系
	git checkout --track -b master origin/master
	
	# 切换到指定分支，并更新工作区
	git checkout [branch-name]

	#对比分区的区别
	git diff branch1 branch2
	
	# 切换到上一个分支
	git checkout -
	
	# 建立追踪关系，在现有分支与指定的远程分支之间
	git branch --set-upstream [branch] [remote-branch]
	
	# 合并指定分支到当前分支
	git merge [branch]
	
	# 选择一个commit，合并进当前分支
	git cherry-pick [commit]
	
	# 删除分支,如果分支没有合并，删除失败
	git branch -d [branch-name]

	# 删除分支，即使没有合并，仍然可以删除
	git branch -D [branch-name]
	
	# 删除远程分支
	git push origin --delete [branch-name]
	git branch -dr [remote/branch]

	#重命名分支,不会覆盖已经存在的同名分支
	git branch -m original-branch newbranch
	
	#重命名分支，会覆盖已经存在的同名分支
	git branch -M original-branch newbranch
	
	#基于哈希创建新分支
	#可用于查看某个历史断面
	git branch emputy bfe57de0	
	
	#基于标签创建新分支
	git branch [tag] emputy

	#查看合并的分支
	git branch --merged
	#查看未合并的分支
	ggit branch --no-merged
	
## 标签

	# 列出所有tag
	git tag

	# 新建一个tag在当前commit
	git tag [tag]
	
	#缩略commitID并单行显示提交信息
	git log --pretty=oneline --abbrev-commit
	# 新建一个tag在指定commit
	git tag [tag] [commitID]
	
	#创建带有说明的标签，-a指定标签名，-m指定说明文字。
	git tag -a v0.1 -m "version 0.1 released" [commitID]

	
	# 删除本地tag
	git tag -d [tag]
	
	# 删除远程tag
	git push origin :refs/tags/[tagName]
	
	# 查看tag信息
	git show [tag]
	
	# 提交指定tag
	git push [remote] [tag]
	
	# 提交所有tag
	git push [remote] --tags
	
	# 新建一个分支，指向某个tag
	git checkout -b [branch] [tag]

	# 推送标签1.0到远程
	git push origin v1.0 

	# 推送所有的标签到远程
	git push origin --tags 
	
	# 删除远程标签，但是前提是要先在本地删除对应标签。
	git push origin :refs/tags/v0.9 

## 查看信息

	# 显示有变更的文件
	$ git status
	
	#简短方式
	git status -s
	
	# 显示当前分支的版本历史
	$ git log
	
	# 显示commit历史，以及每次commit发生变更的文件
	$ git log --stat
	
	# 搜索提交历史，根据关键词
	$ git log -S [keyword]
	
	# 显示某个commit之后的所有变动，每个commit占据一行
	$ git log [tag] HEAD --pretty=format:%s
	
	# 显示某个commit之后的所有变动，其"提交说明"必须符合搜索条件
	$ git log [tag] HEAD --grep feature
	
	# 显示某个文件的版本历史，包括文件改名
	$ git log --follow [file]
	$ git whatchanged [file]
	
	# 显示指定文件相关的每一次diff
	$ git log -p [file]
	
	# 显示过去5次提交
	$ git log -5 --pretty --oneline
	
	# 显示所有提交过的用户，按提交次数排序
	$ git shortlog -sn
	
	# 显示指定文件是什么人在什么时间修改过
	$ git blame [file]

	# 显示今天你写了多少行代码
	$ git diff --shortstat "@{0 day ago}"
	
	# 显示某次提交的元数据和内容变化
	$ git show [commit]
	
	# 显示某次提交发生变化的文件
	$ git show --name-only [commit]
	
	# 显示某次提交时，某个文件的内容
	$ git show [commit]:[filename]
	
	# 显示当前分支的最近几次提交
	$ git reflog

### log支持的选项参考
|选项|说明|
|:-----|----------:|
|-p|按补丁格式显示每个更新之间的差异|
|--word-diff|按 word diff 格式显示差异。|
|--stat|显示每次更新的文件修改统计信息。|
|--shortstat只显示 --stat 中最后的行数修改添加移除统计。|
|--name-only|仅在提交信息后显示已修改的文件清单。|
|--name-status|显示新增、修改、删除的文件清单。|
|--abbrev-commit|仅显示 SHA-1 的前几个字符，而非所有的 40 个字符。|
|--graph	显示|ASCII 图形表示的分支合并历史。|
|--relative-date|使用较短的相对时间显示（比如，“2 weeks ago”）。|
|--pretty|使用其他格式显示历史提交信息。可用的选项包括 oneline，short，full，fuller 和 format（后跟指定格式）。|
|--oneline|--pretty=oneline --abbrev-commit 的简化用法。|


### diff命令详解

![](http://img0.tuicool.com/Vb2qAn.png!web)
	
	# 显示暂存区和工作区的差异
	$ git diff
	
	# 显示暂存区和上一个commit的差异
	$ git diff --cached [file]
	
	# 显示工作区与当前分支最新commit之间的差异
	$ git diff HEAD
	
	# 显示两次提交之间的差异
	$ git diff [first-branch]...[second-branch]

	#显示出所有有差异的文件列表
	$ git diff branch1 branch2 --stat 
	
	#将两个分支不同内容输出到foo.diff 文件中，用notepad++/sublime 之类的编辑器打开，高亮颜色
	$ git diff branch1 branch2 --color > foo.diff  

	#显示当前目录下的lib目录和上次提交之间的差别（更准确的说是在当前分支下）
	$ git diff HEAD -- ./lib

	#、比较上次提交commit和上上次提交
    $ git diff HEAD^ HEAD

	#直接将两个分支上最新的提交做diff
    $ git diff topic master 或 git diff topic..master

	#如果你想查看将要合并的某个分枝会有什么样的变化，将branch替换为你想要合并的分枝名即可
	$ git diff ...(branch)
	

## 远程仓库同步
	
	#查看远程仓库
	git remote -v
	
	#添加远程仓库
	git remote add [name] [repository-url]

	#删除远程仓库
	git remote rm [name]
	
	#修改远程仓库地址：
	git remote set-url origin new-repository-url

	#拉取远程仓库： 
	#pull与fetch的区别是pull会自动merge，fetch不会
	git pull [remoteName] [localBranchName]

	#推送远程仓库： 
	git push [remoteName] [localBranchName] 

	#将当前分支推送到远端master分支
	git push -u orgin master 

	#将本地 test 分支提交到远程 master 分支
	#(把本地的某个分支 test 提交到远程仓库，并作为远程仓库的 master 分支) 
	git push origin test:master 

	#提交本地 test 分支作为#远程的 test 分支 :
	git push origin test:test

	# 强行推送当前分支到远程仓库，即使有冲突
	git push [remote] --force

	# 推送所有分支到远程仓库
	git push [remote] --all

## 撤销错误的修改和提交

	#撤销一个已经提交的快照
	#需要修复一个公共提交，git revert命令正是被设计来做这个的。
	#确保你只对本地的修改使用git reset
	git revert [commitID]
	
	#checkout 有三个不同作用， 检出文件、检出提交和检出分支
	#检出最新版文件
	git checkout HEAD [filename]

	#检出某次提交的文件，会覆盖工作区同名文件，并显示除待提交状态
	git checkout [commitID] <file>

	#检出某次提交所有文件，相当于查看历史版本的文件
	#工作区和暂存区内容和那次提交之后内容相同
	git checkout [commitID]

	#检出打标签时文件内容
	git checkout [tag]

	#如果查看完历史版本文件内容，想回到最新提交文件内容
	git checkout [branchName]

	# 恢复暂存区的所有文件到工作区
	git checkout .
	
	#比如你删除了index.html，使用以下命令恢复文件，也可以恢复之前修改过的
	git checkout  -- index.html 
	

	#git reset 仅仅用于未提交远程仓库的撤销操作
	#如果已经提交远程仓库请用 git revert

	#git reset --hard HEAD^  reset index and working directory , 以来所有的变更全部丢弃，并将 HEAD 指向
	#git reset --soft HEAD^  nothing changed to index and working directory ,仅仅将 HEAD 指向 ，所有变更显示在 “changed 						to be committed”中
	#git reset --mixed HEAD^ default,reset index ,nothing to working directory 默认选项，工作区代码不改动，添加变更到index区

	# 重置暂存区的指定文件，与上一次commit保持一致，但工作区不变
	#有时我们会不小心git add,取消某些add的文件。(还原暂存区)
	git reset [file]
	
	# 重置暂存区与工作区，与上一次commit保持一致
	# 相当于删除了未提交的更改，而且是同时删除了暂存区和工作区
	git reset --hard

	#重设缓冲区，匹配最近的一次提交，但工作目录不变
	git reset
	
	# 重置当前分支的指针为指定commit，同时重置暂存区，但工作区不变
	git reset [commit]

	# 重置当前分支的HEAD为指定commit，同时重置暂存区和工作区，与指定commit一致
	git reset --hard [commit]
	
	#只更改引用的指向，不改变暂存区和工作区
	git reset --soft <commit>

	# 重置当前HEAD为指定commit，但保持暂存区和工作区不变
	git reset --keep [commit]


### git reset

git reset 有3个选项.

* --soft 不会影响到工作目录还有暂存区里的东西
* --hard 工作目录，暂存区直接重置到指定的提交状态
* --mixed 默认选项，会把暂存区里的东西重置到指定提交状态，并且指针指向这个提交。

一般情况, 如果你发现commit文件是存在bug情况，你只需要修改文件代码，那就用默认的mixed，hard会重置文件的内容到指定的commit，也就是说你的之前写的代码会被重置删除掉，切记。

 	git reset [--soft | --mixed | --hard] [-q] [<commit>]




![](http://img0.tuicool.com/Mra6Rj.png!web)

## 其他命令
	
	git blame -w  # 忽略移除空白这类改动
	git blame -M  # 忽略移动文本内容这类改动
	git blame -C  # 忽略移动文本内容到其它文件这类改动
	
	# 执行一次git clean的『演习』。它会告诉你那些文件在命令执行后会被移除，而不是真的删除它。
	git clean -n

	# 移除当前目录下未被跟踪的文件
	git clean -f

	#移除未跟踪的文件，但限制在某个路径下。
	git clean -f <path>

	
	#移除未跟踪的文件，以及目录。
	git clean -df

	#移除当前目录下未跟踪的文件，以及Git一般忽略的文件。
	git clean -xf


## 保存修改恢复进度文件

	#git stash 是保存的暂存区文件，如果你不将工作区修改文件add进暂存区
	#这些未添加的文件不会被保存，或者显示`No local changes to save`
	#git stash保存多个的时候，新增的一个Index永远是0，由近及远index递增。

	# 将工作区现场储藏起来，等以后恢复后继续工作。通常用于处理更为着急的任务时，例如：bug。
	git stash
	
	#查看保存的工作现场
	git stash list
	
	#查看保存的工作现场详情
	$ git stash show -p stash@{0}

	#恢复工作现场,但不删除
	git stash apply
	
	#删除stash内容,只清除一条，而且是最新添加的，也就是index为0的那条
	git stash drop 
	
	#恢复的同时直接删除stash内容
	git stash pop 

	#恢复指定的工作现场，当你保存了不只一份工作现场时。
	git stash apply stash@{0}

	#就是清空所有暂存区的记录
	git stash clear

## 别名

使用如下方法为命令设置别名

	git config --global alias.(name) "(command)"


别名进行了一些整理修改，这是我现在的.gitconfig里的别名配置：


	[alias]
	st = status -sb
	co = checkout
	br = branch
	mg = merge
	ci = commit
	ds = diff --staged
	dt = difftool
	mt = mergetool
	last = log -1 HEAD
	latest = for-each-ref --sort=-committerdate --format=\"%(committername)@%(refname:short) [%(committerdate:short)] %(contents)\"
	ls = log --pretty=format:\"%C(yellow)%h %C(blue)%ad %C(red)%d %C(reset)%s %C(green)[%cn]\" --decorate --date=short
	hist = log --pretty=format:\"%C(yellow)%h %C(red)%d %C(reset)%s %C(green)[%an] %C(blue)%ad\" --topo-order --graph --date=short
	type = cat-file -t
	dump = cat-file -p
	lg = log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit --date=relative
	findstr = git rev-list --all | xargs git grep -F 


## 组合拳

### 删除已经合并到master的分支

	git branch --merged master | grep -v '^\*\|  master' | xargs -n 1 git branch -d
	
### 忽略一个已经提交的文件



	#假设你不小心上传local.properties这个文件
	#忽略对这个文件的追踪，就要用到下面的命令，同时在.gitignore中添加忽略
	git update-index --assume-unchanged (path/file)

### 创建一个新的空分支
这里以github的操作为例，下面试图创建一个名为gh-pages的空分支

	$cd repo

	$ git checkout --orphan gh-pages
	# 创建一个orphan的分支，这个分支是独立的
	Switched to a new branch \'gh-pages\'

	git rm -rf .
	# 删除原来代码树下的所有文件
	rm \'.gitignore\'
注意这个时候你用git branch命令是看不见当前分支的名字的，除非你进行了第一次commit。

下面我们开始添加一些代码文件，例如这里新增了一个index.html
	
	
	$ git add -A
	$ git commit -a -m \"First pages commit\"
	$ git push origin gh-pages
在commit操作之后，你就可以用git branch命令看到新分支的名字了，然后push到远程仓库

### merge和rebase的区别和使用场景

参考这篇文章 [2.7-重写项目历史](https://github.com/geeeeeeeeek/git-recipes/wiki/2.7-%E9%87%8D%E5%86%99%E9%A1%B9%E7%9B%AE%E5%8E%86%E5%8F%B2)

### 把A分支的某一个commit，放到B分支上

这个过程需要cherry-pick命令，[参考](http://sg552.iteye.com/blog/1300713#bc2367928)

git checkout <branch-name> && git cherry-pick <commit-id>
### 压缩多个Commit
压缩两个commit `git rebase -i HEAD~2`

### 永久删除文件(包括历史记录)
如果我们在git远程仓库不小心保存了账号密码或者其他机密或敏感信息文件，我们可以用以下步骤来进行补救

1.重写每个分支的历史 

	git filter-branch --force --index-filter 'git rm --cached --ignore-unmatch path-to-your-remove-file' --prune-empty --tag-name-filter cat -- --all

如果看到类似下面这样表示成功了

	Rewrite 48dc599c80e20527ed902928085e7861e6b3cbe6 (266/266)
	# Ref 'refs/heads/master' was rewritten

如果看到这样，表示路径有问题，删除失败了

	WARNING: Ref 'refs/heads/master' is unchanged
	WARNING: Ref 'refs/remotes/origin/master' is unchanged
	WARNING: Ref 'refs/remotes/origin/master' is unchanged
	WARNING: Ref 'refs/stash' is unchange

2.添加忽略文件

用代码，或者手动在.gitignore添加都可以

	echo (filename) >> .gitignore
	git add .gitignore
	git commit -m "Add sensitive (filename) file to gitignore"

3.强制push到远程仓库

	git push origin master --force

4.清理和回收空间
虽然上面我们已经删除了文件, 但是我们的repo里面仍然保留了这些objects, 等待垃圾回收(GC), 所以我们要用命令彻底清除它, 并收回空间.

	rm -rf .git/refs/original/
	git reflog expire --expire=now --all
	git gc --prune=now
	git gc --aggressive --prune=now

### 输出最后一次提交的改变到压缩包中

	git archive -o ../updated.zip HEAD $(git diff --name-only HEAD^)

	#输出两个提交间的改变
	git archive -o ../latest.zip NEW_COMMIT_ID_HERE $(git diff --name-only OLD_COMMIT_ID_HERE NEW_COMMIT_ID_HERE) 

### 检测你的分支的改变是否为其它分支的一部分

cherry命令让我们检测你的分支的改变是否出现在其它一些分支中。它通过+或者-符号来显示从当前分支与所给的分支之间的改变：是否合并了(merged)。.+ 指示没有出现在所给分支中，反之，- 就表示出现在了所给的分支中了。这里就是如何去检测：

	git cherry -v OTHER_BRANCH_NAME_HERE
	#例如: 检测master分支
	git cherry -v master
### 使用rebase推送而非merge

如果您正在团队中工作并且整个团队都在同一条branch上面工作，那么您就得经常地进行fetch/merge或者pull。Git中，分支的合并以所提交的merge来记录，以此表明一条feature分支何时与主分支合并。但是在多团队成员共同工作于一条branch的情形中，常规的merge会导致log中出现多条消息，从而产生混淆。因此，您可以在pull的时候使用rebase，以此来减少无用的merge消息，从而保持历史记录的清晰。

	git pull --rebase

您也可以将某条branch配置为总是使用rebase推送：

	git config branch.BRANCH_NAME_HERE.rebase true	 

### 远程分支回滚到某个commit

	$ git checkout the_branch

	$ git pull
	
	$ git branch the_branch_backup //备份一下这个分支当前的情况
	
	$ git reset --hard the_commit_id //把the_branch本地回滚到the_commit_id
	
	$ git push origin :the_branch //删除远程 the_branch
	
	$ git push origin the_branch //用回滚后的本地分支重新建立远程分支
	
	$ git push origin :the_branch_backup //如果前面都成功了，删除这个备份分支

### git将单个文件恢复到历史版本的正确方法如下：

	$ git reset commit_id 文件路径
	$ git checkout -- 文件路径	

### git reflog

Git reflog 可以查看所有分支的所有操作记录（包括（包括commit和reset的操作），包括已经被删除的commit记录，git log则不能察看已经删除了的commit记录
commit1: add Test1.java
commit2: add Test2.java
commit3: add Test2.java
	
	#删除commit3的提交
	$ git reset --hard HEAD~1
	#如果恢复commit3的提交，就要查看commitid
	$ git reflog
	
	3e94c18ae HEAD@{0}: commit: add Test3.java
	e55c2974f HEAD@{1}: commit: add Test2.java
	3e94c18ae HEAD@{2}: commit: add Test1.java
	#恢复删除的commit3 有两种方式 
	$ git reset --hard 3e94c18ae
	#第二种方式
	$ git cherry-pick 3e94c18ae
	


	

## 参考文章

1. <https://www.ibm.com/developerworks/cn/devops/d-learn-workings-git/index.html>
2. git学习圣经<https://git-scm.com/book/zh/v2>
3. 常用 Git 命令清单 阮一峰的网络日志 <http://www.ruanyifeng.com/blog/2015/12/git-cheat-sheet.html>
4. <https://juejin.im/post/58f817b55c497d0058e0926a>
5. <https://github.com/xirong/my-git/blob/master/useful-git-command.md>
6. <https://github.com/geeeeeeeeek/git-recipes/wiki>
7. 试试Git – 15分钟的Git交互教程<https://try.github.io/levels/1/challenges/1>
8. git奇技淫巧<https://github.com/521xueweihan/git-tips>
9. Git Community Book 中文版<http://gitbook.liuhui998.com/index.html>
10. <http://iissnan.com/progit/>
11. <https://git-scm.com/book/zh/v2/>
12. <http://gitbook.liuhui998.com/index.html>
13. <https://github.com/ruijun/Android-Dev-Favorites/blob/master/Git/Git.md>
14. <http://www.jianshu.com/p/da3ee7d07a03>
15. commit是如何填message<http://mp.weixin.qq.com/s?__biz=MzAwNDYwNzU2MQ==&mid=401622986&idx=1&sn=470717939914b956ac372667ed23863c&scene=2&srcid=0114ZcTNyAMH8CLwTKlj6CTN&from=timeline&isappinstalled=0#wechat_re
16. 
17. 
18. direct>

