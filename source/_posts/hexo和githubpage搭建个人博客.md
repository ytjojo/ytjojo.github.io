---
title: Hexo和GitHub Pages建立个人博客
date: 2017-04-18 17:32:15
categories: hexo
description: 
tags: [Hexo,git,git配置,ssh,github page,建立个人博客]
---
#Hexo和GitHub Pages建立个人博客

>作为一个程序员，一直想写点技术文章，一来记录下自己的工作学习心得体会，二是遇到棘手问题或者深层理论可以理清楚思路头绪，记录下来方便以后查找，三是锻炼自己表达和整理能力，四分享给同行业需要的人，帮别人解决共同问题，如果内容比较精髓，可以增加自己的影响力。总之好处多多。但是去哪写是个问题，大部分人选择简书，CSDN，cnblogs，LOFTER，在这些平台写博客方便，但是自定义功能太差，界面千篇一律不够极客，所以我选择了自建。自建的方式有WordPress、Jekyll和Octopress，但是配置什么的比较麻烦，最后选择了HEXO


### 简介
hexo出自台湾大学生[tommy351](https://twitter.com/tommy351)之手，是一个基于Node.js的静态博客程序，其编译上百篇文字只需要几秒。hexo生成的静态网页可以直接放到GitHub Pages，BAE，SAE等平台上。官方中文文档在[这里](https://hexo.io/zh-cn/docs/index.html)

### 环境准备

### 安装Node.js

到Node.js的[官网](https://nodejs.org/en/)下载对应平台的最新版本，有两个可选项，**Recommended for most User** 和 **Latest Features**,我下载的是**Recommended for most User** V6.10.2LTS	版本。一路安装，自定义安装的话注意配置环境变量。[配置教程看这里](http://www.cnblogs.com/zhouyu2017/p/6485265.html)
 
### 安装Hexo

打开命令行，键入命令

```
	npm install -g hexo
```

### 安装git

去git官方网站下载页选择对应平台下载，[安装教程在这](http://blog.csdn.net/renfufei/article/details/41647875/)
设置下环境变量。

### Github注册和GitHubPage建立


+ 注册github账号，已有忽略。

+ 建立与你账户名对应的仓库，仓库名必须是[your_name.github.io],一个账号下只能建一个。

+ 生成SSH Keys

首先设置你的用户名密码：

	git config --global user.email "你的邮箱"
	git config --global user.name "github用户名"

	
	cd ~/.ssh #检查本机的ssh密钥

如果提示：No such file or directory 说明你是第一次使用git

生成ssh key，替换你的邮件地址

	ssh-keygen -t rsa -C "邮件地址@youremail.com"

提示

	Generating public/private rsa key pair.
	Enter file in which to save the key (/Users/Administrator/.ssh/id_rsa):#<回车就好>

然后系统会要你输入密码：
	
	Enter passphrase (empty for no passphrase):<输入加密串>
	Enter same passphrase again:<再次输入加密串>
**注意**，你可能疑惑输入密码命令行窗口却不显示，不要惊慌，只要正常输入就好了。
+ 添加SSH key 到github
  用记事本打开本地C:\Documents and Settings\Administrator.ssh\id_rsa.pub文件，完全复制文本内容

  登陆github系统。点击右上角的 Account Settings--->SSH Public keys ---> add another public keys
	
  你本地生成的密钥复制到里面（key文本框中）， 点击 add key 就ok了

**测试github ssh key**
键入命令

	ssh -vT git@github.com
如果看到

	The authenticity of host 'github.com (207.97.227.239)' can't be established.
	RSA key fingerprint is 16:27:ac:a5:76:28:2d:36:63:1b:56:4d:eb:df:a6:48.
	Are you sure you want to continue connecting (yes/no)?

输入yes就好，最后如果看到

	Hi 【你的用户名】! You've successfully authenticated, but GitHub does not provide shell access.

就表示成功了

### 搭建Hexo工作空间



在电脑上建立一个处理博客的文件夹如E:\blog_github\hexo，鼠标右键Git Bash Here 打开命令行，输入：
		
	hexo init
	npm install #安装所需要的依赖包

执行完成后看到目录结构

	.
	├── _config.yml
	├── package.json
	├── scaffolds
	├── source
	|   ├── _drafts
	|   └── _posts
	└── themes

结构说明：

_config.yml：可以在这里配置需要的配置。

package.json：应用程序数据。EJS, Stylus 和 Markdown渲染器会自动安装。如果想要卸载他们，也是可以的。

scaffolds：当你创建一个新文章时，用到的模板文件。

themes：主题。


---
您可以在 `_config.yml` 中修改大部份的配置。

#### 网站

参数 | 描述
--- | ---
`title` | 网站标题
`subtitle` | 网站副标题
`description` | 网站描述
`author` | 您的名字
`language` | 网站使用的语言
`timezone` | 网站时区。Hexo 默认使用您电脑的时区。[时区列表](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones)。比如说：`America/New_York`, `Japan`, 和 `UTC` 。

其中，`description`主要用于SEO，告诉搜索引擎一个关于您站点的简单描述，通常建议在其中包含您网站的关键词。`author`参数用于主题显示文章的作者。

#### 网址

参数 | 描述 | 默认值
--- | --- | ---
`url` | 网址 |
`root` | 网站根目录 |
`permalink` | 文章的 [永久链接](permalinks.html) 格式 | `:year/:month/:day/:title/`
`permalink_defaults` | 永久链接中各部分的默认值 |

网站存放在子目录
如果您的网站存放在子目录中，例如 `http://yoursite.com/blog`，则请将您的 `url` 设为 `http://yoursite.com/blog` 并把 `root` 设为 `/blog/`。


#### 目录

参数 | 描述 | 默认值
--- | --- | ---
`source_dir` | 资源文件夹，这个文件夹用来存放内容。 | `source`
`public_dir` | 公共文件夹，这个文件夹用于存放生成的站点文件。 | `public`
`tag_dir` | 标签文件夹 | `tags`
`archive_dir` | 归档文件夹 | `archives`
`category_dir` | 分类文件夹 | `categories`
`code_dir` | Include code 文件夹 | `downloads/code`
`i18n_dir` | 国际化（i18n）文件夹 | `:lang`
`skip_render` | 跳过指定文件的渲染，您可使用 [glob 表达式](https://github.com/isaacs/node-glob)来匹配路径。 |

提示
如果您刚刚开始接触Hexo，通常没有必要修改这一部分的值。


#### 文章

参数 | 描述 | 默认值
--- | --- | ---
`new_post_name` | 新文章的文件名称 | :title.md
`default_layout` | 预设布局 | post
`auto_spacing` | 在中文和英文之间加入空格 | false
`titlecase` | 把标题转换为 title case | false
`external_link` | 在新标签中打开链接 | true
`filename_case` | 把文件名称转换为 (1) 小写或 (2) 大写 | 0
`render_drafts` | 显示草稿 | false
`post_asset_folder` | 启动 [Asset 文件夹](asset-folders.html) | false
`relative_link` | 把链接改为与根目录的相对位址 | false
`future` | 显示未来的文章 | true
`highlight` | 代码块的设置 |

相对地址
默认情况下，Hexo生成的超链接都是绝对地址。例如，如果您的网站域名为`example.com`,您有一篇文章名为`hello`，那么绝对链接可能像这样：`http://example.com/hello.html`，它是**绝对**于域名的。相对链接像这样：`/hello.html`，也就是说，无论用什么域名访问该站点，都没有关系，这在进行反向代理时可能用到。通常情况下，建议使用绝对地址。


#### 分类 & 标签

参数 | 描述 | 默认值
--- | --- | ---
`default_category` | 默认分类 | `uncategorized`
`category_map` | 分类别名 |
`tag_map` | 标签别名 |

#### 日期 / 时间格式

Hexo 使用 [Moment.js](http://momentjs.com/) 来解析和显示时间。

参数 | 描述 | 默认值
--- | --- | ---
`date_format` | 日期格式 | `YYYY-MM-DD`
`time_format` | 时间格式 | `H:mm:ss`

#### 分页

参数 | 描述 | 默认值
--- | --- | ---
`per_page` | 每页显示的文章量 (0 = 关闭分页功能) | `10`
`pagination_dir` | 分页目录 | `page`

#### 扩展

参数 | 描述
--- | ---
`theme` | 当前主题名称。值为`false`时禁用主题
`deploy` | 部署部分的设置




现在已经搭建本地Hexo博客了，执行以下命令，然后在浏览器输入<http://localhost:4000>

	hexo g #生成博客依赖的html css js文件
	hexo s #本地提供博客服务

就会看到如下界面
![](https://raw.githubusercontent.com/CoderTian/CoderTian.github.io/master/2015/11/26/github-hexo-blog/success.png)

### 更改网站描述信息
打开你hexo工作目录如E:\blog_github\hexo，找到_config.yml文件打开，配置以下参数

	title: 张三的博客 
	subtitle: 学习笔记
	description: That's all, thank you！
	author: 张三
	language: zh-CN
	timezone: Asia/Shanghai

注意冒号之后有一个空格的，不然解析报错

### 关联github仓库
打开你hexo工作目录如E:\blog_github\hexo，找到_config.yml文件打开
	
	deploy:
  	type: git
  	repo: git@github.com:ytjojo/ytjojo.github.io.git
  	branch: master

### 将Hexo系统绑定到你的github page网址
打开你hexo工作目录如E:\blog_github\hexo，找到_config.yml文件打开，找到# URl这一项，更改url: 和root: 内容，如

	# URL
	## If your site is put in a subdirectory, set url as 'http://yoursite.com/child' and root as '/child/'
	url: https://github.com/ytjojo/ytjojo.github.io
	root: /
	permalink: :year/:month/:day/:title/
	permalink_defaults:

由于我Hexo是直接初始化在blog根目录文件夹下的，因此我的url和root没有改动，如果是初始化在子目录下的(如：blog中的hexo目录)就需要改为
	url: https://github.com/ytjojo/ytjojo.github.io/hexo
	root: /hexo/

配置完成，清除生成静态文件

	hexo clean

重新生成静态文件
	
	hexo g #hexo generate的简化版

部署到your_name.github.io.

	hexo d #hexo deploy的简化版，部署到github远程仓库

如果发布时候出现错误

	ERROR Deployer not found: git

执行

	npm install hexo-deployer-git --save

出现

	hexo-site@0.0.0 E:\blog_github\hexo
	`-- hexo-deployer-git@0.2.0

表示正常安装，默认是0.2.0版本，默认也可以指定高版本的，

	npm install hexo-deployer-git@0.4.0 --save

直到命令行出现

	Branch master set up to track remote branch master from git@github.com:ytjojo/ytjojo.github.io.git.
	To github.com:ytjojo/ytjojo.github.io.git
   	ad4d091..817debc  HEAD -> master
	INFO  Deploy done: git
	


打开你的主页,<https://ytjojo.github.io/>，就会看到你的博客了

### 添加文章

1. 如果已经有写好的markdown文件
打开文件，头部加入下面描述。已经有就忽略
	
	title: postName #文章页面上的显示名称，可以任意修改，不会出现在URL中
	date: 2013-12-02 15:30:16 #文章生成时间，一般不改，当然也可以任意修改
	categories: example #分类
	tags: [tag1,tag2,tag3] #文章标签，可空，多标签请用格式，注意:后面有个空格
	description: 附加一段文章摘要，字数最好在140字以内。
复制到source\_posts文件夹下面
2. 如果是新建文章
	
	hexo new [layout] "postName" #新建文章

其中layout是可选参数，hexo提供了3种默认的布局， post 、 page 和 draft ，路径分别为： source/_posts 、 source 、 source/_draft 。如果你将在文章前置申明中，将layout设置为false，那么这篇文章将不会有任何的布局。如果不写layout类型，默认到source/_post文件夹，如果想放入草稿文件夹，输入
	
	hexo new draft "我新建的草稿文章"

找到source/_draft文件夹对应的文件打开，发现少了categories，date
等，可以手动添加，也可以配置下，下次执行  `hexo new "新的文章"` 时候默认就会有这些信息
配置方法：在你的hexo工作空间打开scaffolds\post.md或者scaffolds\draft.md,输入


	---
	title: {{ title }}
	date: {{ date }}
	categories: 
	tags: 
	description: 
	---

新建文章之后，就可以用自己喜欢的markdown 编辑器写文章了

**注意**：source/_posts是网站中显示的文章source/_draft是草稿文件夹，有些主题默认不显示的，如果你在草稿里已经完成了文章，请将文章复制到source/_posts文件夹下

### 将新建的文章发布到博客里
	
	hexo clean
	hexo g
	hexo d
	
过一会儿打开你的博客网址<https://ytjojo.github.io/>,就会看到你写的博客了

### fancybox
![](http://ww1.sinaimg.cn/large/c1ff19eagy1feqnavr0daj20l40op40u.jpg)

可能有人对这个[Reading](http://bruce-sha.github.io/reading/)页面中图片的fancybox效果感兴趣，这个是怎么做的呢。
很简单，只需要在你的文章*.md文件的头上添加photos项即可，然后一行行添加你要展示的照片：

	layout: photo
	title: 我的阅历
	date: 2085-01-16 07:33:44
	tags: [hexo]
	photos:
	- http://bruce.u.qiniudn.com/2013/11/27/reading/photos-0.jpg
	- http://bruce.u.qiniudn.com/2013/11/27/reading/photos-1.jpg

如果不想每次添加，打开您的scaffolds\photo.md，没有的话新建一个。

	layout: { { layout } }
	title: { { title } }
	date: { { date } }
	tags: 
	photos: 
	- 
	---

然后每次可以执行带layout的new命令生成照片文章：

	hexo new photo "photoPostName" #新建照片文章

### 文章摘要
在需要显示摘要的地方添加如下代码即可：

	以上是摘要
	<!--more-->
	以下是余下全文

用`<!--more-->`分割摘要和正文,

more以上内容即是文章摘要，在主页显示，more以下内容点击『> Read More』链接打开全文才显示。

### 参考文章

  1. <http://ibruce.info/2013/11/22/hexo-your-blog/>
  2. <http://www.jianshu.com/p/05289a4bc8b2>
  3. <http://www.jianshu.com/p/a4b74cc9ff28>
  4. <http://blog.csdn.net/u014595668/article/details/51854259>
  5. hexo官方中文文档<https://hexo.io/zh-cn/docs/>
	



 
	
	
	








 




