Activity栈与任务管理探究1——栈与任务的概述
### 内容概览
1. 前言

2. Activity中的Stack

3. Activity中的Task

4. Activity栈与任务管理基本原则

5. 参考文献

### 前言
      Activity是Android的四大组件之一，是Android开发中非常重要的一环。无论是Android开发新手，还是老司机，在日常的开发工作中，都会经常与Activity/Intent打交道。在开发之初，对Activity的很多知识都是似是而非，一知半解，尤其是Activity栈（Stack）与任务（Task）的相关管理。后来接触到SingTask，SingTop等LaunchMode、 Intent flag等组合功能，以及做了一些模拟Activity栈管理开发后，发现其中很多知识对日常开发的实用价值非常高。

      随着开发的继续深入，逐步发现，Activity的Stack和Task管理逻辑中有很多妙不可言的东西。

      基于此，接下来会用几篇博文整理下自己的若干心得。疑义相与析，欢迎拍砖和指正。

### Activity中的Stack
      栈是一种数据结构，栈中的数据的存储和访问采取的是 “先入后出，后入先出”的模式。对于Activity而言，Activity在不断地跳转（onCreate）和回退（onDestory）过程中涉及到的Activity的创建和销毁，即涉及到Activity栈的压栈和出栈。

### Activity中的Task
      官方上对Activity中的Task是这样定义的：

>A task is a collection of activities that users interact with when performing a certain job. The activities are arranged in a stack (the back stack), in the order in which each activity is opened.

简单来说，Activity中的Task就是一组以栈为模式聚集在一起的Activity组件集合。Activity Task有点类似于一个Activity Stack的容器，如下图所示：
当App启动时如果不存在当前App的任务栈就会自动创建一个，默认情况下一个App中的所有Activity都是放在一个Task中的。但是如果指定了特殊的启动模式（例如SingInstance启动模式），那么就会出现同一个App的Activity出现在不同的任务栈中的情况，也会有任务栈中包含来自于不同App的Activity。

### Activity栈与任务管理基本原则
(1) 一般情况下：

App中采用的是单任务模式，也即全部地Activity都在同一个容器中压栈和出栈；
App中Activity的压栈和出栈都是采用“先入后出，后入先出”的模式，并且每次Activity跳转和回退都只会导致一个Activity的压栈和出栈；
      (2) 在特殊的应用场景中，Activity的栈和任务管理会有很大的变动，直接影响Activity的栈和任务管理的因素有：

LaunchMode
Intent Flag
TaskAffinity
      在后续的博客中会陆续对相关内容进行总结。

      (3) App process被杀死并不意味着Activity的Task和Stack会被自动清空。

      App在Activity进入后台的情况下，都会通过onSaveInstanceState方法保存当前Activity栈中的信息，一旦App被意外杀死，而Activity栈没有清空的情况下，下次点击进入App，或者App自动被重新拉起的时候，会自动拉起到之前栈中的内容，并保有之前的跳转和回退逻辑。

      杀死App 进程有以下几类典型的场景：

代码中杀死当前进程：System.exit(0)（杀死process，但是没有清空Activity栈）；

程序运行中遇到了崩溃问题（杀死process，但是没有清空Activity栈）；

Terminal中杀死进程，adb kill pid（杀死process，且清空Activity栈）；

在手机进程中kill掉正在运行的进程（杀死process，且清空Activity栈）；

其他异常操作，引起系统自动杀死当前进程，如：

后台切换语言（杀死process，但是没有清空Activity栈）；

后台切换字体（杀死process，但是没有清空Activity栈）；

后台关闭当前程序的权限（杀死process，但是没有清空Activity栈）；

但是在回退的过程中，每个页面都是重新创建（onCreate）的，也即页面view是重新绘制的，页面数据时重新获取的。在实际的开发过程中，很有可能会涉及到Stack中不同Activity之间的数据交互或页面跳转，开发者一般默认认为下层栈或底层栈中的Activity内容是存在的，这样遇到App被意外杀死，并重新启动的情况下，就很有可能会造成空指针问题。在实际的开发过程中，要格外留意。