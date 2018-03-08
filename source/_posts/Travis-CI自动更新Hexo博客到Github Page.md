---
title: Travis-CI自动更新Hexo博客到Github Page 
date: 2017-04-20 17:34:15
categories: hexo
description: 
tags: [Travis-CI,自动更新Hexo,多端同步,持续集成]
---


# Travis-CI自动更新Hexo博客到Github Page

## 前言

如过你是用Hexo和github page来构建自己的独立博客，你一定被多端同步困扰过，如果你的公司安装了Hexo构建环境，但是家里电脑却没有，如果想在家更新博客就变得麻烦了，你需要把你公司博客源文件拷贝到你家里电脑里，然后安装Node、Hexo、Git，生成Ssh等等。有人就考虑用dropbox等云盘或者git仓库进行同步，但是总体流程不够顺畅。有没有更好的方式处理呢？答案是肯定的的。如何操作？

1. githubPage仓库下建立另一个分支，命名blog-source，用来保存我们的博客源代码
2. 用持续集成工具关联github仓库，我的是https://github.com/ytjojo/ytjojo.github.io.git，并监听blog-source分支代码改动。
3. 将博客源代码push到仓库的blog-source分支，
4. 持续集成工具监听到push，运行脚本
5. 持续集成工具将生成public文件夹下的博客静态文件push到仓库的主分支master上。
6. 打开https://ytjojo.github.io/看到改动。


## Travis CI

>[Travis CI](https://travis-ci.org) 是目前新兴的开源持续集成构建项目，它与jenkins，GO的很明显的特别在于采用yaml格式，同时他是在在线的服务，不像jenkins需要你本地打架服务器，简洁清新独树一帜。目前大多数的github项目都已经移入到Travis CI的构建队列中，据说Travis CI每天运行超过4000次完整构建。对于做开源项目或者github的使用者，如果你的项目还没有加入Travis CI构建队列，那么我真的想对你说out了。
### 登录
用github账号登录Tavis CI

![](http://ww1.sinaimg.cn/large/c1ff19eagy1fet7j6sh6oj20jh06xgm8.jpg)

登录后就会看到

![](http://ww1.sinaimg.cn/large/c1ff19eagy1fet8d4n13uj21gz0hz3yp.jpg)

点击加号

![](http://ww1.sinaimg.cn/large/c1ff19eagy1fet8ffqwoyj20zc0e7q36.jpg)

找到你的github仓库，将开关打开

![](http://ww1.sinaimg.cn/large/c1ff19eagy1fet8gc6vp2j20ng0903ye.jpg)

开启后需要配置下，点击红框的那个菜单按钮

![](http://ww1.sinaimg.cn/large/c1ff19eagy1fet8lol5asj21gr09amx5.jpg)

就会出现这样的下拉菜单，我们选择设置，来到这个界面，我们按照如下勾选

![](http://ww1.sinaimg.cn/large/c1ff19eagy1fet8sz4j5fj20k804xmx1.jpg)

Build only if .travis.yml is present：是只有在.travis.yml文件中配置的分支改变了才构建
Build pushes：当推送完这个分支后开始构建

到这一步，Tavis 会监听到blog-source 分支下pushes，并进行构建。但是如何构建，还有构建完成后如何将hexo生成要public文件夹内容推送到仓库 master分支上这些问题还没解决。

### 配置githhub的Access Token

打开github首页，右上角 打开下拉菜单 Setting，进入之后点击Personal access tokens，点击右上角的Generate new token按钮，

除了delet repo全部勾选上

生成完成后拷贝下来，
在Trav Ci页面，添加环境变量GH_TOKEN，value就是复制的github access token

![](http://ww1.sinaimg.cn/large/c1ff19eagy1fet9bo9y4aj217k0mnmxh.jpg)

## 初始化blog—source分支

本地磁盘新建文件夹，将远程仓库blog-source分支拉下来，没有就新建
git命令
	
	git init
	# 添加自己的项目
	git remote add origin git@github.com:ytjojo/ytjojo.github.io.git
	# 新建并切换分支
	git checkout --orphan blog-source
	git add -A
	git commit -m "Travis CI"
	git push 

将hexo源码复制到这个git仓库目录下
必须的文件如下

	.
	├── _config.yml*
	├── db.json*
	├── node_modules
	├── package.json*
	├── scaffolds*
	├── source*
	│   ├── CNAME*
	│   ├── _posts
	│   ├── about
	│   ├── categories
	│   ├── img
	│   ├── media
	│   └── tags
	└── themes


同时在当前目录下新建.travis.yml

	language: node_js
	node_js: stable
	
	# S: Build Lifecycle
	install: 
	- npm install 
	- yarn
	
	
	#before_script: 
	#- npm install -g gulp
	
	 
	 # 缓存，可以节省集成的时间，这里我用了yarn，如果不用可以删除
	cache:
	  apt: true
	  yarn: true
	  directories:
	    - node_modules
	    
	# tarvis生命周期执行顺序详见官网文档
	before_install:
	- git config --global user.name "ytjojo" #替换自己的
	- git config --global user.email "ytjojo@163.com" #替换自己的
	# 由于使用了yarn，所以需要下载，如不用yarn这两行可以删除
	- curl -o- -L https://yarnpkg.com/install.sh | bash
	- export PATH=$HOME/.yarn/bin:$PATH
	- npm install -g hexo
	- git clone https://github.com/raytaylorlin/hexo-theme-raytaylorism.git themes/raytaylorism #这个要替换你自己的主题
	
	script:
	  - hexo clean
	  - hexo g
	after_success:
	  - cd ./public
	  - git init
	  - git add --all .
	  - git commit -m "Update docs"
	  - git push --quiet --force https://$GH_TOKEN@$GH_REF  #注意没有双引号，我被坑过  
	    master
	# E: Build LifeCycle
	
	branches:
	  only:
	    - blog-source 
	env:
	 global:
	   - GH_REF: github.com/ytjojo/ytjojo.github.io.git #替换自己的

## 提交blog-source分支
分支文件结构如图

![](http://ww1.sinaimg.cn/large/c1ff19eagy1fetapb1g3rj20l007wt9d.jpg)

最后将本地blog-source 分支提交到远程，Travis 就会自动构建然后push到你github page master分支上

## 参考文章

1. <https://github.com/ytjojo/ytjojo.github.io.git>
2. <http://kchen.cc/2016/11/12/hexo-instructions/>



















 






