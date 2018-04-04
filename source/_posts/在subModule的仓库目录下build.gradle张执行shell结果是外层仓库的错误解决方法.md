---
title: 在subModule的仓库目录下build.gradle执行shell结果是外层仓库的错误解决方法
date: 2018-04-04 17:06:20
categories: build.gradle执行shell
tags: [build.gradle,git]
---

### 在build.gradle 获取当前仓库hash值
rootProject是一个仓库工程
子目录是另外一个git仓库，是android工程的一个module
网上给出的代码很简单
```groovy

def getGitVersion() {

  return 'git rev-parse --short HEAD'.execute().text.trim()
}

```
但是工程中git 仓库嵌套 或者有submodule 的时候在内部git仓库中的build.gradle添加上面的方法，结果发现是外部git仓库的结果

原来默认git shell 执行的工作空间是rootProject的磁盘空间，执行shell的时候要传入shell执行目录
代码如下
```java
 String parent = rootProject.rootDir.absolutePath.replaceAll('\\\\','/')
  def workingDir = new File(parent,"yourmodulename")
return 'git rev-parse --short HEAD'.execute(null,workingDir).text.trim()
```


获取本地所有分支和获取所有Tag
```
def getGitBranch() {
    String parent = rootProject.rootDir.absolutePath.replaceAll('\\\\','/')
    def workingDir = new File(parent,"yourmodulename")
    return 'git rev-parse --short --symbolic --branches'.execute(null,workingDir).text.trim()
}

def getGitTag() {
    String parent = rootProject.rootDir.absolutePath.replaceAll('\\\\','/')
    def workingDir = new File(parent,"yourmodulename")
    return 'git rev-parse --short --symbolic --tags'.execute(null,workingDir).text.trim()
}
```
### android工程无脑添加module
正常添加module都是这样的

include ':app'
include ':Library'
如果你想无脑添加本地所有module
在setting.gradle添加如下代码

```groovy
def  projectDir = getSettingsDir() as File
File[] files = projectDir.listFiles()
files.each {
    println "it.name ::::::: "+it.name+":::::::"
    if(it.isDirectory() && it.name.contains("ngr_")){
        def moduleName = it.name;
        it.listFiles(new FileFilter() {
            @Override
            boolean accept(File file) {
                return file.name.equals("build.gradle")
            }
        }).each {
            println(moduleName+"setting---------")
            include(":"+moduleName)
        }

    }

}
```