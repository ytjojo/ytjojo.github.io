---
title: git利用sparsecheckout上传apk文件
date: 2018-04-8 10:18:00
categories: git
tags: [git,shell,sparsecheckout]
---

### 关键点
1. Conditional setps(multiple)插件使用
2. 根据版本号和服务器环境制定保存目录
3. sparsecheckout制定checkout文件夹提升效率。

### 缺点很明显，初始化仓库很慢


这只是一个初步的方案，写这篇文章目的只是简单记录，并不推荐使用

### 参数化构建
参数化构建选择 Boolean Parameter
Name 填 uploadApkToGitLab
![](http://ww1.sinaimg.cn/large/c1ff19eagy1fq51r0fz32j20yg07wwen.jpg)

### Conditional setps(multiple)

run？中选择Boolean condition
Token 填${uploadApkToGitLab}
表示是否勾选上传git仓库

![](http://ww1.sinaimg.cn/large/c1ff19eagy1fq51olq3jrj2158073aa5.jpg)

### add step to condition
点击add step to condition 选择 execute shell

 gitlab@gitlab.ngarihealth.com:yangtengjiao/doctorApks.git 是我的保存apk文件的git 仓库
重复上传同一个文件夹内历史记录会被清空，防止历史记录太大，pull的时候消耗时间过长。
利用sparsecheckout只checkout或者push指定的目录，这样效率会大大提升。
填入shell命令

```
# 根据服务器环境来决定放在那个文件夹里。
if [ "$NGR_ENVIR" == Test ];then
childDir=release
fi
if [ "$NGR_ENVIR" == PreRelease ];then
childDir=prerelease
fi
if [ "$NGR_ENVIR" == Release ];then
childDir=master
fi
#这只有测试版 预发布和正式版才会上传git仓库 ，其他服务器环境childDir是空值，直接退出
if [ ! $childDir ]; then
  echo "childDir IS NULL"
  exit 0
else
  echo "NOT NULL"
fi 

if [ ! -d "$WORKSPACE/doctorApks" ]; then
  mkdir  $WORKSPACE/doctorApks
fi
cd doctorApks

#根据版本号来决定父文件夹
apkDir=$APP_VERSION_NAME"_"$APP_VERSION_CODE

git init   
git config core.sparsecheckout true   #开启sparse clone 这是关键
git config --global user.email "yanttj@ngarihealth.com"#这个已经要填，不然会报错
git config --global user.name "yangtj"
echo "${apkDir}" >> .git/info/sparse-checkout   #设置需要pull的目录，*表示所有，!表示匹配相反的
git remote add -f origin gitlab@gitlab.ngarihealth.com:yangtengjiao/doctorApks.git
git pull origin master  #更新

#删除历史记录
git filter-branch --force --index-filter "git rm --cached -r --ignore-unmatch $apkDir/$childDir" --prune-empty --tag-name-filter cat -- --all


#删除 历史记录后上传
git push --force --all
rm -rf .git/refs/original/

git reflog expire --expire=now --all
# 回收git垃圾

git gc --prune=now

git gc --aggressive --prune=now

if [ ! -d "$WORKSPACE/doctorApks/$apkDir/$childDir" ]; then
  mkdir  $WORKSPACE/doctorApks/$apkDir/$childDir
fi
# 将新生成的apk文件复制到git仓库中制定目录下
cp $WORKSPACE/buildApks/*.apk $WORKSPACE/doctorApks/$apkDir/$childDir
cp $WORKSPACE/buildApks/*.txt $WORKSPACE/doctorApks/$apkDir/$childDir

git add .
git commit  -m "提交的描述信息"
# 上传apk
git push origin master
```

