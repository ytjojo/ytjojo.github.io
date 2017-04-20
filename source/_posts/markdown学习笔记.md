---
title: Markdown学习笔记
date: 2017-04-15 15:58:15
tags: [markdown,markdown编辑器,markdown语法,图床]
categories: markdown
description: 
---

#Markdown学习笔记
[toc]


### 导语

markdown是一种轻量级的标记语言，宜读宜写，常被用了写文章，记日记，总结经验，也被程序员用了写技术文档，技术博客。markdown的语法比较简单，常用的也就不超过10个，学习成本也比较低，一旦熟悉这种语法规则，有一劳永逸的效果，对以后的工作学习都有很好的帮助。简书、github、Tumblr等平台都原生支持markdown语法，学习markdown更有助于写出简洁美观宜与阅读的文章。

### Markdown 官方文档

* [创始人 John Gruber 的 Markdown 语法说明](http://daringfireball.net/projects/markdown/syntax)

* [Markdown 中文版语法说明](http://wowubuntu.com/markdown/#list)

### Markdown工具推荐

**Mac Os X**

* [MOU](http://25.io/mou/)被很多人推荐
* 
![Mou](http://www.williamlong.info/upload/4319_14.jpg)

* [Ulysses](https://www.ulyssesapp.com/)一款由国外开发商 The Soulmen 制作的 Markdown 编辑器。与其它同类应用相比，Ulysses 最大的不同在于，它能根据内置的文件管理器，以及与 iCloud 云服务器的实时同步方案，达到最快捷的文章整理效率。


<img src="http://ww1.sinaimg.cn/large/c1ff19eagy1ferpraokkfj20py0gkamo.jpg" width = "90%"  alt="图片名称" align=center />

* [Mweb](http://zh.mweb.im/)界面简洁高效、功能强大全面支持 Github Flavored Markdown 语法如 TOC、Table、Fenced code block、LaTex、Task lists、Footnote 等。内置图床功能。

![](http://zh.mweb.im/asset/img/cn/1-1.jpg)


**Windows平台上**

* [MarkdownEditor](https://link.zhihu.com/?target=https%3A//github.com/chenguanzhou/MarkDownEditor/releases)Metro风格的markdown编辑器，号称功能最全，一个亮点是与七牛存储集成，可以直接将本地图片上传到服务器，将图片的URL地址嵌入到编辑器

![https://link.zhihu.com/?target=https%3A//www.microsoft.com/store/apps/9nblggh4q9rs](https://pic1.zhimg.com/b507d385b024c86efe3457c423bdf004_b.jpg)

* [MarkdownPad](http://www.markdownpad.com/)一款全功能的编辑器，被很多人称赞为windows 平台最好用的markdown编辑器

![MarkdownPad](http://www.williamlong.info/upload/4319_10.jpg)

* [MarkPad](http://code52.org/DownmarkerWPF/)开源的 Markdown 编辑器,与 Window 8 风格和谐友好的界面，可以直接在你的博客或者 GitHub 中打开、保存文档，直接将图片粘贴到 Markdown 文档中。

![MarkPad](http://code52.org/DownmarkerWPF/screenshot.png)
**web端**

* [简书](http://www.jianshu.com/),简书是一个将写作与阅读整合在一起的网络产品。集合文字的书写、编集、发布功能于一体的在线写作编辑工具。

* [StackEdit](https://stackedit.io/editor)好用的网页版编辑器

开源，免费，搭建在github page上，源码寄存在GitHub。整合Dropbox和Google Drive，自动同步（如果能够指定文件夹自动同步当然是最好的）支持一键发布到Google Blog，Tumblr等。可左右或者上下分栏，一边显示Markdown语言一边显示效果可以单击左右栏分界线切换至纯写作模式，同样可以收缩顶部工具栏。在网页顶部工具栏支持加粗、倾斜、超链接、撤消和还原 等等（注意：与Word处理方式有稍许不同）。不论在左栏还是右栏滚动页面另一侧也会同步。支持Markdown Extra 以GIst发布后支持分享（可以在线使用StackEdit阅读）多种保存格式详细的说明文档界面优美.

* [Markable.in](https://markable.in/)，支持实时预览，自动保存，保存到Dropbox，发布到Tumblr等。

![Markable.in](https://markable.in/static/images/screenshot-1.png)


**跨平台**

* [Cmd Markdown](https://www.zybuluo.com/mdeditor) 作业部落出品，也是一款不错的工具和博客平台兼顾的产品。全平台且提供web版
![Cmd Markdown](http://www.williamlong.info/upload/4319_6.jpg)

* [Haroopad](http://pad.haroopress.com/user.html)是一款覆盖三大主流桌面系统的编辑器，支持 Windows、Mac OS X 和 Linux。 主题样式丰富，语法标亮支持 54 种编程语言。最新版支持流程图和幻灯片

![Haroopad](http://img1.tuicool.com/fiaIVz.png!web)

* [小书匠编辑器](http://soft.xiaoshujiang.com/) 全平台覆盖并且有web版

![](http://ww1.sinaimg.cn/large/c1ff19eagy1fejmcv0wunj21gw0oq41q.jpg)

* [Typora](http://typora.io/) 有Windows 和Linux版本


![Typora](http://www.williamlong.info/upload/4319_15.jpg)

**文库集成类**


* GitBook: 集成GitHub

* Madoko: 集成GitHub, DropBox, OneDrive

* 马克飞象: 集成印象笔记
GitBook的火热程度如同GitHub，我所看到的很多软件帮助文档、技术教程，都已经在GibBook上发布。GitBook于2014年创办，已发布35500本书籍。

### 图床

Markdown作为纯文本格式，自然不能粘贴图像文件，只能嵌入图像的地址（URL或者本地地址）。所以插入图片需要预先将图片存储在网络。

![](http://ww1.sinaimg.cn/large/c1ff19eagy1fejm5pg7qwj20qq0c0tab.jpg)
在chrome搜图床就会出现这三个，都挺好用的。
另外一个图床墙裂推荐七牛，七牛是云服务提供商，注册就送10G云存储
如何使用看这篇文章<http://www.jianshu.com/p/5f0d5451ca01>
配合[MPic](http://mpic.lzhaofu.cn)，一款支持拖曳、复制、截图上传的七牛图床神器,用起来爽爆了。


### 截屏工具
windows平台[Faststone Capture](https://faststone-capture.en.softonic.com/)
    
体积小巧、功能强大。不但具有常规截图等功能，更有从扫描器获取图像，和将图像转换为 PDF 文档等功能,该软件拥有不规则抓图、滚动抓图、活动窗口抓图、图片简单处理、屏幕录制等很多很多实用的功能。

Mac OS X平台 [Snip](http://snip.qq.com/)

Snip是一款腾讯推出的一款截图工具，是Mac平台的截屏应用，支持自动识别窗口、图标标记再次编辑、关联QQ邮箱截屏、滚动截屏、邮件分享截图、支持Retina显示屏等。 


### 语法
#### 1.  **标题**

行首用1-6个#开头表示不同级别的标题，注意#号和文字之间有个空格，在有些编辑器或者网站上会自动生成文章目录
![](http://ww1.sinaimg.cn/large/c1ff19eagy1feiwg6ftxcj20wl0g70tq.jpg)

#### 2. **引用**

行首使用 > 加上一个空格表示引用一个段落，可嵌套

![](http://ww1.sinaimg.cn/large/c1ff19eagy1fejnie3clej21a007q3za.jpg)

#### 3. **分割线**

在一行连续三个或者三个以上_或者\*  
	___  
	***


![](http://ww1.sinaimg.cn/large/c1ff19eagy1fejnrtinbaj2197082aao.jpg)
途中分割线不是很明显。

#### 4. **代码区域**

代码区域内的文字不会被处理，按照原样输出。
每一行前边加入4个空格或者一个tab可以标记一个代码段落：

	
	int main(){
    	return 0
    }

效果如下

![](http://ww1.sinaimg.cn/large/c1ff19eagy1fejo70v647j20pc02ot8i.jpg)

还可以使用  \`这是代码块\` 来标记行内代码

如：在activity的初始化代码一般是\`  protected void onCreate(Bundle savedInstanceState){super.onCreate(savedInstanceState);}\`

效果如下

![](http://ww1.sinaimg.cn/large/c1ff19eagy1fejotkmz67j21bh056dge.jpg)


也可以用三个`表示代码块

	```javascript
	var a = "hello world";
	var b = "good luck";
	```

#### 5. **强调**
	*斜体*
	**粗体**
  
	
	_斜体_
	__粗体__
![](http://ww1.sinaimg.cn/large/c1ff19eagy1fejoqd53waj20uv06r74o.jpg)
#### 6. **链接**

Markdown有两种链接方式：Inline以及Reference

*  文字链接

    Inline:
    [谷歌](https://www.google.com) 
    Reference:
    [谷歌][google_url]
    [google_url]:https://www.google.com

![](http://ww1.sinaimg.cn/large/c1ff19eagy1fejp1f177bj20so05zdg2.jpg)

* 图片链接
	
		![](https://www.baidu.com/img/bd_logo1.png)
		![][baidu_logo]
		[baidu_logo]:https://www.baidu.com/img/bd_logo1.png

注意！第二种叹号后第一个[]容易漏掉，一定不要忘记哦。
当图片url地址含有(或)时的处理

	
	![](http://latex.codecogs.com/gif.latex?%5Cprod(n_%7Bi%7D+100))
	
	![][latex_img]
	[latex_img]:http://latex.codecogs.com/gif.latex?%5Cprod(n_%7Bi%7D+100)
	<img src="http://latex.codecogs.com/gif.latex?\prod(n_{i_1})+10000">
![](http://ww1.sinaimg.cn/large/c1ff19eagy1fejq7fbwi7j211m04qwey.jpg)

* 自动链接

使用尖括号<>包含住一段地址或者邮箱  


	<http://www.baidu.com>


**图片大小控制**
	
固定图片显示大小：


```<img src="http://img.blog.csdn.net/20151129213701642" width=256 height=256 alt="图片名称" align=center  />
```


百分比控制大小

```<img src="http://img.blog.csdn.net/20151129213701642" width="50%" alt="图片名称" align=center />```


给图片加脚注  

	<center>
	<img src="http://img.blog.csdn.net/20151129213701642" width="25%" alt="图片名称" align=center />
	Figure 1. Lena
	</center>```


<center>
<img src="http://img.blog.csdn.net/20151129213701642" width="25%" alt="图片名称" align=center />
Figure 1. Lena
</center>

增大脚注

	<center>
	<img src="http://img.blog.csdn.net/20151129213701642" width="25%" height="25%" />
	$ $
	Figure 1. Lena
	</center>

<center>
<img src="http://img.blog.csdn.net/20151129213701642" width="25%" height="25%" />
$ $
Figure 1. Lena
</center>

#### 7. **转义字符**


	\\ 反斜杠
	\` 反引号
	\* 星号
	\_ 下划线
	\{\} 大括号
	\[\] 中括号
	\(\) 小括号
	\# 井号
	\+ 加号
	\- 减号
	\. 英文句号
	\! 感叹号
	&gt; >	
	&amp; &
	&nbsp； 空格（non-breaking space）


#### 8. **列表**

* 无序列表

 一个*或者+或者- 加上一个空格
	
	* 无序列表
	+ 无序列表
	- 无序列表
	- 
![](http://ww1.sinaimg.cn/large/c1ff19eagy1fejqrg1it5j20zq04a74c.jpg)

* 有序列表

使用数字接着一个英文句点再加一个空格,数字可以任意值

	1. 第一项
	1. 第二项
	1. 第三项

![](http://ww1.sinaimg.cn/large/c1ff19eagy1fejqxygs4yj210w03h3yi.jpg)


#### 9. **段落和换行**

单个回车视为空格.
连续回车才能分段

两种方式  

1. 输入`<br/>` #有写编辑器不支持。。

2. 两个空格加回车键 
  
---
**一下是扩展语法**


#### 10. **任务列表**

未做任务`- + 空格 + [ ]`
已做任务`- + 空格 + [x]`

	- [ ] 任务一 给女朋友买口红
	- [x] 任务二 陪女朋友吃饭
	
![](http://ww1.sinaimg.cn/large/c1ff19eagy1fejsy1c55mj20zu01ut8t.jpg)

#### 11. **表格格式**


	第一格表头 | 第二格表头
	--------- | -------------
	内容单元格 第一列第一格 | 内容单元格第二列第一格
	内容单元格 第一列第二格 多加文字 | 内容单元格第二列第二格  

![](http://ww1.sinaimg.cn/large/c1ff19eagy1fejv0vifb5j21az03vwez.jpg)

#### 12. **删除线** 

    加删除线像这样用： ~~删除这些~~   

![](http://ww1.sinaimg.cn/large/c1ff19eagy1fejv78uo1mj20vj028jre.jpg)  

#### 13. **顺序图或流程图**

	```sequence
	张三->李四: 嘿，小四儿, 写博客了没?
	Note right of 李四: 李四愣了一下，说：
	李四-->张三: 忙得吐血，哪有时间写。
	```
	
	```flow
	st=>start: 开始
	e=>end: 结束
	op=>operation: 我的操作
	cond=>condition: 确认？
	
	st->op->cond
	cond(yes)->e
	cond(no)->op
	```

![](http://ww1.sinaimg.cn/large/c1ff19eagy1fejvy1jwgxj21f70jqmyf.jpg)

#### 14. **MathJax**
 
关于MathJax与LaTex

[可参考该文章](http://mlworks.cn/posts/introduction-to-mathjax-and-latex-expression/)
	
	块级公式：
	$$  x = \dfrac{-b \pm \sqrt{b^2 - 4ac}}{2a} $$
	
	\\[ \frac{1}{\Bigl(\sqrt{\phi \sqrt{5}}-\phi\Bigr) e^{\frac25 \pi}} =
	1+\frac{e^{-2\pi}} {1+\frac{e^{-4\pi}} {1+\frac{e^{-6\pi}}
	{1+\frac{e^{-8\pi}} {1+\ldots} } } } \\]
	
	行内公式： $\Gamma(n) = (n-1)!\quad\forall n\in\mathbb N$

![](http://ww1.sinaimg.cn/large/c1ff19eagy1fejw6yp6e1j21dd080mya.jpg)

#### 15.**脚注(Footnote)**


	第一[^1x]
	[^1x]:脚注的用发
	
	
	百度地址[^百度地址]
	[^百度地址]:www.baidu.com

![](http://ww1.sinaimg.cn/large/c1ff19eagy1fejx7dl6vij20tf05l0su.jpg)
在文档的末尾自动生成如下图所示脚注
![](http://ww1.sinaimg.cn/large/c1ff19eagy1fejxe0kh6nj20kv036a9y.jpg)

#### 16. **TOC** 

如果想点击文章中某一小标题自动滚动到标题位置，如何做呢，很简单。


```
 [toc]
```


#### 参考文章:

1. <http://www.jianshu.com/p/4Q3aay>
2. <http://www.cnblogs.com/gibbonnet/p/5373703.html>
3. <https://www.zhihu.com/question/19637157>
4. <http://zh.mweb.im/markdown-syntax-guide-suggest-version-zh.html#toc_24>
5. <http://www.williamlong.info/archives/4319.html>
6. <http://www.jianshu.com/p/1e402922ee32/>
7. Markdown 语法说明 (简体中文版)<http://wowubuntu.com/markdown/#p>
8. <http://blog.csdn.net/yhl_leo/article/details/50099843>
