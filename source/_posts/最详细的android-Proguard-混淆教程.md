---
title: 最详细的android Proguard 混淆教程
date: 2017-04-21 17:43:00
categories: Android Proguard
tags: [android混淆,Proguard,android Proguard,恢复堆栈,Unknown Source]
---

>ProGuard是一个压缩、优化和混淆Java字节码文件的免费的工具，它可以删除无用的类、字段、方法和属性。可以删除没用的注释，最大限度地优化字节码文件。它还可以使用简短的无意义的名称来重命名已经存在的类、字段、方法和属性。常常用于Android开发用于混淆最终的项目，增加项目被反编译的难度。  
  
## ProGuard工作原理

ProGuar由shrink、optimize、obfuscate和preveirfy四个步骤组成，每个步骤都是可选的，我们可以通过配置脚本来决定执行其中的哪几个步骤。
![](http://ww1.sinaimg.cn/large/c1ff19eagy1fextrggeb9j20p8068mx3.jpg)

1. 压缩(Shrink)：检测并移除代码中无用的类、字段、方法和特性（Attribute）。
2. 优化(Optimize)：对字节码进行优化，移除无用的指令。
3. 混淆(Obfuscate)：使用a，b，c，d这样简短而无意义的名称，对类、字段和方法进行重命名。
4. 预检(Preveirfy)：在Java平台上对处理后的代码进行预检，确保加载的class文件是可执行的。

那么ProGuard是如何工作的呢？在这里我们引入一个概念EntryPoint。EntryPoint可以理解为一种标志，它是在ProGuard过程中不会被处理的类或者方法，在压缩的过程,ProGuard会从上述的EntryPoint中开始遍搜索出哪些类和类的成员在使用（被标记为EntryPoint的类和方法有些是在使用的而且这些是我们在混淆文件中配置不希望被混淆的类和方法，有些是没有使用的）。对于那些没有被使用的类和成员，就会在压缩阶段被丢弃。然后在接下来的优化步骤中，那些非EntryPoint的类，方法都会被设置为private，static或者final，而且不使用的参数都会被移除。接着在混淆的步骤中，ProGuard会对非EntryPoint的类和方法进行重命名。最后就会对代码进行预检测，以便保证稳定性。

### Entry points官方介绍

>###Entry points

>In order to determine which code has to be preserved and which code can be discarded or obfuscated, you have to specify one or more entry points to your code. These entry points are typically classes with main methods, applets, midlets, activities, etc.

>In the shrinking step, ProGuard starts from these seeds and recursively determines which classes and class members are used. All other classes and class members are discarded.
In the optimization step, ProGuard further optimizes the code. Among other optimizations, classes and methods that are not entry points can be made private, static, or final, unused parameters can be removed, and some methods may be inlined.
In the obfuscation step, ProGuard renames classes and class members that are not entry points. In this entire process, keeping the entry points ensures that they can still be accessed by their original names.
The preverification step is the only step that doesn't have to know the entry points. 

使用-keep可以使指定的类和类成员成为Entry Point, 其实就是扫描开始的地方, 即使没有其他人直接引用这个类或者类成员, 也保留它.


## Android 混淆配置原则:

* 反射用到的类不混淆
* JNI方法不混淆
* AndroidMainfest中的类不混淆，四大组件和Application的子类和Framework层下所有的类默认不会进行混淆
* Parcelable的子类和Creator静态成员变量不混淆，否则会产生android.os.BadParcelableException异常
* 继承了Serializable接口的类不混淆
* 使用GSON、fastjson等框架时，所写的JSON对象类不混淆，否则无法将JSON解析成对应的对象
* 使用第三方开源库或者引用其他第三方的SDK包时，需要在混淆文件中加入对应的混淆规则
* 有用到WEBView的JS调用也需要保证写的接口方法不混淆
* 在AndroidManifest中配置的类，比如四大组件
* Layout文件引用到的自定义View
* 引入的第三方库（一般）。这里推荐两个开源项目，里面收集了一些第三方库的混淆规则
	* [android-proguard-snippets](https://github.com/krschultz/android-proguard-snippets)
	* [android-proguard-cn](https://github.com/msdx/android-proguard-cn)
	
* 删除Syst.out 和Log.d打印输出代码

## 语法
详细说明参见[官方文档](https://www.guardsquare.com/en/proguard/manual/usage#classspecification)

### 保留

* -keep {Modifier} {class_specification} 保护指定的类文件和类的成员
* -keepclassmembers {modifier} {class_specification} 保护指定类的成员，如果此类受到保护他们会保护的更好
* -keepclasseswithmembers {class_specification} 保护指定的类和类的成员，但条件是所有指定的类和类成员是要存在。
* -keepnames {class_specification} 保护指定的类和类的成员的名称（如果他们不会压缩步骤中删除）
* -keepclassmembernames {class_specification} 保护指定的类的成员的名称（如果他们不会压缩步骤中删除）
* -keepclasseswithmembernames {class_specification} 保护指定的类和类的成员的名称，如果所有指定的类成员出席（在压缩步骤之后）
* -printseeds {filename} 列出类和类的成员-keep选项的清单，标准输出到给定的文件
### 压缩

* -dontshrink 不压缩输入的类文件
* -printusage {filename}
* -whyareyoukeeping {class_specification}
### 优化

* -dontoptimize 不优化输入的类文件
* -assumenosideeffects {class_specification} 优化时假设指定的方法，没有任何副作用
* -allowaccessmodification 优化时允许访问并修改有修饰符的类和类的成员
### 混淆

* -dontobfuscate 不混淆输入的类文件
* -obfuscationdictionary {filename} 使用给定文件中的关键字作为要混淆方法的名称
* -overloadaggressively 混淆时应用侵入式重载
* -useuniqueclassmembernames 确定统一的混淆类的成员名称来增加混淆
* -flattenpackagehierarchy {package_name} 重新包装所有重命名的包并放在给定的单一包中
* -repackageclass {package_name} 重新包装所有重命名的类文件中放在给定的单一包中
* -dontusemixedcaseclassnames 混淆时不会产生形形色色的类名
* -keepattributes {attribute_name,…} 保护给定的可选属性，例如LineNumberTable, LocalVariableTable, SourceFile, * * * Deprecated, Synthetic, Signature, and InnerClasses.
* -renamesourcefileattribute {string} 设置源文件中给定的字符串常量	

### 通配符


|通配符|规则|
|:-----|----------:|
|？|	匹配单个字符|
|*|匹配类名中的任何部分，但不包含额外的包名|
|**|匹配类名中的任何部分，并且可以包含额外的包名|
|%|匹配任何基础类型的类型名|
|*|匹配任意类型名 ,包含基础类型/非基础类型|
|...|匹配任意数量、任意类型的参数|
|<init>|匹配任何构造器|
|<ifield>|匹配任何字段名|
|<imethod>|匹配任何方法|
|*(当用在类内部时)|匹配任何字段和方法|
|$|指内部类|



## 公共配置选项
这是配置项，基本不需要动，全部加上

	
	# 忽略警告，避免打包时某些警告出现
	# 
	#-ignorewarnings  #不要忽略警告，风险太大                  
	# 指定代码的压缩级别
	-optimizationpasses 5
	# 不使用大小写混合，混淆后的类名为小写
	-dontusemixedcaseclassnames
	#指定不去忽略非公共的库的类
	-dontskipnonpubliclibraryclasses
	#指定不去忽略非公共的库的类的成
	-dontskipnonpubliclibraryclassmembers
	# 不做预校验，preverify是proguard的4个步骤之一
	# Android不需要preverify，去掉这一步可加快混淆速度
	-dontpreverify
	# 有了verbose这句话，混淆后就会生成映射文件
	# 包含有类名->混淆后类名的映射关系
	# 然后使用printmapping指定映射文件的名称
	-verbose
	-printmapping proguardMapping.txt

	# 指定混淆时采用的算法，后面的参数是一个过滤器
	# 这个过滤器是谷歌推荐的算法，一般不改变
	#-optimizations !code/simplification/cast,!field/*,!class/merging/*

	#避免混淆注解和内部类，对json解析重要
	-keepattributes *Annotation*,InnerClasses
	# 避免混淆泛型，这在JSON实体映射时非常重要，比如fastJson
	-keepattributes Signature
	#报错后方便恢复堆栈信息
	-keepattributes SourceFile,LineNumberTable 
	-keepattributes EnclosingMethod 
	# 指定混淆时采用的算法，后面的参数是一个过滤器
	# 这个过滤器是谷歌推荐的算法，一般不改变
	-optimizations !code/simplification/arithmetic,!field/*,!class/merging/*

## 需要保留的东西
这个不需要动的，要全部添加上
	
	# 保留所有的本地native方法不被混淆
	-keepclasseswithmembernames class * {
	    native <methods>;
	}
	 
	# 保留了继承自Activity、Application这些类的子类
	# 因为这些子类，都有可能被外部调用
	# 比如说，第一行就保证了所有Activity的子类不要被混淆
	-keep class android.os.**{*;} 
	-keep public class * extends android.app.Activity
	-keep public class * extends android.support.v4.**
	-keep public class * extends android.support.v7.**    
	-keep public class * extends android.app.Fragment 
	-keep public class * extends android.app.Application
	-keep public class * extends android.app.Service
	-keep public class * extends android.content.BroadcastReceiver
	-keep public class * extends android.content.ContentProvider
	-keep public class * extends android.app.backup.BackupAgentHelper
	-keep public class * extends android.preference.Preference
	-keep public class * extends android.view.View
	-keep public class com.android.vending.licensing.ILicensingService
	-keep class android.app.Notification { *;}
	 
	# 如果有引用android-support-v4.jar包，可以添加下面这行
	#-keep public class com.xxxx.app.ui.fragment.** {*;}
	 
	# 保留在Activity中的方法参数是view的方法，
	# 从而我们在layout里面编写onClick就不会被影响
	-keepclassmembers class * extends android.app.Activity {
	    public void *(android.view.View);
	}
	 
	# 枚举类不能被混淆
	-keepclassmembers enum * {
	public static **[] values();
	public static ** valueOf(java.lang.String);
	}
	 
	# 保留自定义控件（继承自View）不被混淆
	-keep public class * extends android.view.View {
	    *** get*();
	    void set*(***);
	    public <init>(android.content.Context);
	    public <init>(android.content.Context, android.util.AttributeSet);
	    public <init>(android.content.Context, android.util.AttributeSet, int);
	}
	#Design包下的Beheviar
	-keepclasseswithmembers class * {
    public <init>(android.content.Context, android.util.AttributeSet);
    public <init>(android.content.Context, android.util.AttributeSet, int);
	}
	 
	# 保留Parcelable序列化的类不被混淆
	-keep class * implements android.os.Parcelable {
	    public static final android.os.Parcelable$Creator *;
	}
	#需要序列化和反序列化的类不能被混淆（注：Java反射用到的类也不能被混淆）  
	-keepnames class * implements java.io.Serializable  
	 
	# 保留Serializable序列化的类不被混淆
	-keepclassmembers class * implements java.io.Serializable {
	    static final long serialVersionUID;
	    private static final java.io.ObjectStreamField[] serialPersistentFields;
	    private void writeObject(java.io.ObjectOutputStream);
	    private void readObject(java.io.ObjectInputStream);
	    java.lang.Object writeReplace();
	    java.lang.Object readResolve();
	}
	 
	# 对于R（资源）下的所有类及其方法，都不能被混淆
	-keep class **.R$* {
	    *;
	}
	 
	# 对于带有回调函数onXXEvent的，不能被混淆
	-keepclassmembers class * {
	    void *(**On*Event);
	}
	#webView
	-keepattributes *JavascriptInterface*
	-keepclassmembers class * extends android.webkit.WebViewClient {
	    public void *(android.webkit.WebView, java.lang.String, android.graphics.Bitmap);
	    public boolean *(android.webkit.WebView, java.lang.String);
	}
	-keepclassmembers class * extends android.webkit.WebViewClient {
	    public void *(android.webkit.WebView, java.lang.String);
	}
	-keep public class android.net.http.SslError
	#v4
	-dontwarn android.support.v4.**
	-dontwarn android.app.Notification
	-dontwarn android.net.SSLCertificateSocketFactory
	-dontwarn android.util.FloatMath
	-dontwarn android.test.**
	-dontwarn org.junit.**
	-dontwarn android.utils.imagecache.*
	-dontwarn android.utils.*
	-dontwarn android.view.*
	#-----------------------------------
	-assumenosideeffects class android.util.Log {
	    public static boolean isLoggable(java.lang.String, int);
	        public static int v(...);
	        public static int i(...);
	        public static int w(...);
	        public static int d(...);
	        public static int e(...);
	    }


## 保留model中的实体类和成员不被混淆
	
将下面的包名（com.android4package.model）替换你自己的

	
	-keep public class com.android4package.model.**{  
		*; 
	}
## 反射的处理

在程序中使用SomeClass.class.method这样的静态方法，在ProGuard中是在压缩过程中被保留的，那么对于Class.forName("SomeClass")呢，SomeClass不会被压缩过程中移除，它会检查程序中使用的Class.forName方法，对参数SomeClass法外开恩，不会被移除。但是在混淆过程中，无论是Class.forName("SomeClass")，还是SomeClass.class，都不能蒙混过关，SomeClass这个类名称会被混淆，因此，我们要在ProGuard文件中保留这个类名称。

* Class.forName("SomeClass")
* SomeClass.class
* SomeClass.class.getField("someField")
* SomeClass.class.getDeclaredField("someField")
* SomeClass.class.getMethod("someMethod", new Class[] {})
* SomeClass.class.getMethod("someMethod", new Class[] { A.class })
* SomeClass.class.getMethod("someMethod", new Class[] { A.class, B.class })
* SomeClass.class.getDeclaredMethod("someMethod", new Class[] {})
* SomeClass.class.getDeclaredMethod("someMethod", new Class[] { A.class })
* SomeClass.class.getDeclaredMethod("someMethod", new Class[] { A.class, B.class })
* AtomicIntegerFieldUpdater.newUpdater(SomeClass.class, "someField")
* AtomicLongFieldUpdater.newUpdater(SomeClass.class, "someField")
* AtomicReferenceFieldUpdater.newUpdater(SomeClass.class, SomeType.class, "someField")
在混淆的时候，要在项目中搜索一下上述方法，将相应的类或者方法的名称进行保留而不被混淆。

## 第三方包的处理

第三方包可以去库的官网查看混淆规则,没什么难点，复制粘贴就ok。
这里推荐两个开源项目，里面收集了一些第三方库的混淆规则

* [android-proguard-snippets](https://github.com/krschultz/android-proguard-snippets)
* [android-proguard-cn](https://github.com/msdx/android-proguard-cn)

>	


	#---------------------------------2.第三方包-------------------------------
	#eventBus
	-keepattributes *Annotation*
	-keepclassmembers class ** {
	    @org.greenrobot.eventbus.Subscribe <methods>;
	}
	-keep enum org.greenrobot.eventbus.ThreadMode { *; }
	-keepclassmembers class * extends org.greenrobot.eventbus.util.ThrowableFailureEvent {
	    <init>(java.lang.Throwable);
	}
	
	#glide
	-keep public class * implements com.bumptech.glide.module.GlideModule
	-keep public enum com.bumptech.glide.load.resource.bitmap.ImageHeaderParser$** {
	  **[] $VALUES;
	  public *;
	}
	
	#-------------------------------------------------------------------------


## Android Studio中使用方法

按照上面的语法规则编写proguard-rules.pro后，需要在build.gradle中配置，需要混淆的时候，设置minifyEnabled为true即可

	buildTypes {
	    debug {
			minifyEnabled true
            //proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
			proguardFiles 'proguard-rules.pro'

	    }
	    release {
	        signingConfig signingConfigs.release
	        minifyEnabled true
	        proguardFiles 'proguard-rules.pro'
	    }
	}
## android混淆之后文件

优化结果文件会输出到<module-name>/build/outputs/mapping/release/目录.

* dump.txt : 描述APK中所有类文件的内部结构(internal structure)
* mapping.txt : 提供混淆前后类(class)名, 方法(method)名和成员变量(field)名的对应关系。表示混淆前后代码的对照表，这个文件非常重要。如果你的代码混淆后会产生bug的话，log提示中是混淆后的代码，希望定位到源代码的话就可以根据mapping.txt反推。
* seeds.txt : 列出没有被混淆的类和成员(classes and members)
* usage.txt : 列出从APK中移除的代码(code)



## 如何恢复混淆之后堆栈信息?
使用方法如下：




### GUI

1. 打开/tools/proguard/bin/proguardgui.bat
2. 选择左边栏的ReTrace选项
3. 添加你的mapping文件和混淆过的堆栈信息
4. 点击ReTrace!

![](http://ww1.sinaimg.cn/large/c1ff19eagy1feyppq0yusj20o106baab.jpg)

### 命令行

1.需要你的ProGuard的mapping文件和你想要还原的堆栈信息（如stacktrace.txt）
2.最简单的方法就是将这些文件拷贝到/tools/proguard/bin/目录
3.运行以下命令
//Windows
retrace.bat -verbose mapping.txt stacktrace.txt > out.txt

//Mac/Linux
retrace.sh -verbose mapping.txt stacktrace.txt > out.txt

所以我们每次打包版本都需要保存最新的mapping.txt文件。如果要使用到第三方的crash统计平台，比如bugly，还需要我们上传APP版本对应的mapping.txt.每次都要保存最新的mapping文件，那不就很麻烦？放心，gradle会帮到你，只需要在bulid.gradle加入下面的一句。每次我们编译的时候，都会自动帮你保存mapping文件到本地的。


	android {
	applicationVariants.all { variant ->
	        variant.outputs.each { output ->
	            if (variant.getBuildType().isMinifyEnabled()) {
	                variant.assemble.doLast{
	                        copy {
	                            from variant.mappingFile
	                            into "${projectDir}/mappings"
	                            rename { String fileName ->
	                                "mapping-${variant.name}.txt"
	                            }
	                        }
	                }
	            }
	        }
	        ......
	    }
	}

## 混淆成中文
原始的混淆功能是把类名方法名混淆成a、b、c、d、e,改造后也可以混淆成中文，参考[这篇文章](http://blog.csdn.net/jiangwei0910410003/article/details/61618945)
和github上的这个库
<https://github.com/heruoxin/proguard-elder-dictionary>


## 其他

* 建议开发和测试阶段都要开启混淆，如果嫌开发阶段恢复堆栈信息麻烦，必须保证测试阶段是开启的，可以尽早发现问题。
* 打包生成的mapping文件一定要保存好，bug收集网站如腾讯bugly和友盟bug统计还原堆栈的时候都需要对应版本的mapping文件。



## 参考

1. 官方文档<https://www.guardsquare.com/en/proguard/manual/usage>
2. 官方examples<https://www.guardsquare.com/en/proguard/manual/examples#androidapplication>
3. <http://www.jianshu.com/p/f3455ecaa56e>
4. <https://segmentfault.com/a/1190000004461614>
5. <http://www.cnblogs.com/cr330326/p/5534915.html>
6. 混淆成中文<http://blog.csdn.net/jiangwei0910410003/article/details/61618945>
7. Proguard Elder Dictionary,使用中文作为字典可以略微提升二次打包的难度<https://github.com/heruoxin/proguard-elder-dictionary>
