---
title: 同一台电脑为gitlab和github添加ssh key
date: 2018-04-09 10:06:00
categories: http
tags: [ssh,ssh config]
---  

生成github ssh key
```

	ssh-keygen -t rsa -f ~/.ssh/id_rsa.github -C "github邮箱"  
```
把公钥添加进github账户
生成 gitlab ssh key

```

	ssh-keygen -t rsa -f ~/.ssh/id_rsa.gitlab -C "gitlab邮箱"  
```
把公钥添加进gitlab仓库
验证是否添加成功

	ssh -T git@github.com  
	ssh -T gitlab@gitlab.com

在用户.ssh目录下如C:\Users\Administrator\.ssh新建文件名为config的文件把下面复制进去，邮箱替换自己的

```
Host gitlab.ngarihealth.com 
  
    IdentityFile ~/.ssh/id_rsa.gitlab  
    User yangtj@ngarihealth.com  
   
Host github.com  
    IdentityFile ~/.ssh/id_rsa.github  
    User ytjojo@163.com  
```