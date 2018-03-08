
---
title: shell中字符串比较
date: 2017-11-25 10:10:20
categories: shell
tags: [shell,shell数组]
---
## 判断字符串相等不相等方法



```shell
#测试各种字符串比较操作。
#shell中对变量的值添加单引号，爽引号和不添加的区别：对类型来说是无关的，即不是添加了引号就变成了字符串类型，
#单引号不对相关量进行替换，如不对$符号解释成变量引用，从而用对应变量的值替代，双引号则会进行替代
#author:tenfyguo
 
A="$1"
B="$2"
 
echo "输入的原始值：A=$A,B=$B"
 
#判断字符串是否相等
if [ "$A" = "$B" ];then
echo "[ = ]"
fi
 
#判断字符串是否相等，与上面的=等价
if [ "$A" == "$B" ];then
echo "[ == ]"
fi
 
#注意:==的功能在[[]]和[]中的行为是不同的，如下
 
#如果$a以”a”开头(模式匹配)那么将为true 
if [[ "$A" == a* ]];then
echo "[[ ==a* ]]"
fi
 
#如果$a等于a*(字符匹配),那么结果为true
if [[ "$A" == "a*" ]];then
echo "==/"a*/""
fi
 
 
#File globbing(通配) 和word splitting将会发生, 此时的a*会自动匹配到对应的当前以a开头的文件
#如在当前的目录中有个文件：add_crontab.sh,则下面会输出ok
#if [ "add_crontab.sh" == a* ];then 
#echo "ok"
#fi
if [ "$A" == a* ];then
echo "[ ==a* ]"
fi
 
#如果$a等于a*(字符匹配),那么结果为true
if [ "$A" == "a*" ];then
echo "==/"a*/""
fi
 
#字符串不相等
if [ "$A" != "$B" ];then
echo "[ != ]"
fi
 
#字符串不相等
if [[ "$A" != "$B" ]];then
echo "[[ != ]]"
fi
 
#字符串不为空，长度不为0
if [ -n "$A" ];then
echo "[ -n ]"
fi
 
#字符串为空.就是长度为0.
if [ -z "$A" ];then
echo "[ -z ]"
fi
 
#需要转义<，否则认为是一个重定向符号
if [ $A /< $B ];then
echo "[ < ]" 
fi
 
if [[ $A < $B ]];then
echo "[[ < ]]" 
fi
 
#需要转义>，否则认为是一个重定向符号
if [ $A /> $B ];then
echo "[ > ]" 
fi
 
if [[ $A > $B ]];then
echo "[[ > ]]" 
fi
```

## 字符串包含关系

方法一：利用grep查找

```
strA="long string"
strB="string"
result=$(echo $strA | grep "${strB}")
if [[ "$result" != "" ]]
then
  echo "包含"
else
  echo "不包含"
fi
```
先打印长字符串，然后在长字符串中 grep 查找要搜索的字符串，用变量result记录结果

如果结果不为空，说明strA包含strB。如果结果为空，说明不包含。

这个方法充分利用了grep 的特性，最为简洁。

 

方法二：利用字符串运算符

复制代码
```
strA="helloworld"
strB="low"
if [[ $strA =~ $strB ]]
then
    echo "包含"
else
    echo "不包含"
fi
```
利用字符串运算符 =~ 直接判断strA是否包含strB。（这不是比第一个方法还要简洁吗摔！）

 

方法三：利用通配符

```
A="helloworld"
B="low"
if [[ $A == *$B* ]]
then
    echo "包含"
else
    echo "不包含"
fi
```
这个也很easy，用通配符*号代理strA中非strB的部分，如果结果相等说明包含，反之不包含。

 

 

方法四：利用case in 语句
```
thisString="1 2 3 4 5" # 源字符串
searchString="1 2" # 搜索字符串
case $thisString in 
    *"$searchString"*) echo Enemy Spot ;;
    *) echo nope ;;
esa
```
这个就比较复杂了，case in 我还没有接触到，不过既然有比较简单的方法何必如此

 

方法五：利用替换

```
STRING_A=$1
STRING_B=$2
if [[ ${STRING_A/${STRING_B}//} == $STRING_A ]]
    then
        ## is not substring.
        echo N
        return 0
    else
        ## is substring.
        echo Y
        return 1
    fi
```

## 判断字符串是否以cn开头
```
if [[ ${var:0:2} -eq "cn" ]]
then
echo sub is  chinese ${var:0:2}
else
echo sub is english ${var:0:2}
fi
```

## 逻辑运算符、逻辑表达式详解

shell的逻辑运算符 涉及有以下几种类型，因此只要适当选择，可以解决我们很多复杂的判断，达到事半功倍效果。

**一、逻辑运算符**

| **逻辑卷标**     | **表示意思**                                 |
| ------------ | :--------------------------------------- |
| 1.           | **关于档案与目录的侦测逻辑卷标！**                      |
| -f           | 常用！侦测『档案』是否存在 eg: if [ -f filename ]     |
| -d           | 常用！侦测『目录』是否存在                            |
| -b           | 侦测是否为一个『 block 档案』                       |
| -c           | 侦测是否为一个『 character 档案』                   |
| -S           | 侦测是否为一个『 socket 标签档案』                    |
| -L           | 侦测是否为一个『 symbolic link 的档案』              |
| -e           | 侦测『某个东西』是否存在！                            |

| 2.           | **关于程序的逻辑卷标！**                           |
| ------------ | :--------------------------------------- |
| -G           | 侦测是否由 GID 所执行的程序所拥有                      |
| -O           | 侦测是否由 UID 所执行的程序所拥有                      |
| -p           | 侦测是否为程序间传送信息的 name pipe 或是 FIFO （老实说，这个不太懂！） |

| 3.           | **关于档案的属性侦测！**                           |
| ------------ | :--------------------------------------- |
| -r           | 侦测是否为可读的属性                               |
| -w           | 侦测是否为可以写入的属性                             |
| -x           | 侦测是否为可执行的属性                              |
| -s           | 侦测是否为『非空白档案』                             |
| -u           | 侦测是否具有『 SUID 』的属性                        |
| -g           | 侦测是否具有『 SGID 』的属性                        |
| -k           | 侦测是否具有『 sticky bit 』的属性                  |

| 4.           | **两个档案之间的判断与比较** ；例如[ test file1 -nt file2 ] |
| ------------ | :--------------------------------------- |
| -nt          | 第一个档案比第二个档案新                             |
| -ot          | 第一个档案比第二个档案旧                             |
| -ef          | 第一个档案与第二个档案为同一个档案（ link 之类的档案）           |
| 5.           | 逻辑的『和(and)』『或(or)』                       |
| &&           | 逻辑的 AND 的意思                              |
| &#124;&#124; | 逻辑的 OR 的意思                               |


| 运算符号 | 代表意义                            |
| ---- | :------------------------------ |
| =    | 等于 应用于：整型或字符串比较 如果在[] 中，只能是字符串  |
| !=   | 不等于 应用于：整型或字符串比较 如果在[] 中，只能是字符串 |
| <    | 小于 应用于：整型比较 在[] 中，不能使用 表示字符串    |
| >    | 大于 应用于：整型比较 在[] 中，不能使用 表示字符串    |
| -eq  | 等于 应用于：整型比较                     |
| -ne  | 不等于 应用于：整型比较                    |
| -lt  | 小于 应用于：整型比较                     |
| -gt  | 大于 应用于：整型比较                     |
| -le  | 小于或等于 应用于：整型比较                  |
| -ge  | 大于或等于 应用于：整型比较                  |
| -a   | 双方都成立（and） 逻辑表达式 –a 逻辑表达式       |
| -o   | 单方成立（or） 逻辑表达式 –o 逻辑表达式         |
| -z   | 空字符串                            |
| -n   | 非空字符串                           |

**二、逻辑表达式**

*   **test 命令**

> 
>
> **使用方法：**test EXPRESSION
>
> 如：
>
> [root@localhost ~]# test 1 = 1 && echo 'ok'
> ok
>
> [root@localhost ~]# test -d /etc/ && echo 'ok'
> ok
>
> [root@localhost ~]# test 1 -eq 1 && echo 'ok'
> ok
>
> [root@localhost ~]# if test 1 = 1 ; then echo 'ok'; fi
> ok
>
> 

> 
>
> **注意：所有字符 与逻辑运算符直接用“空格”分开，不能连到一起。**
>
> 

*   **精简表达式**

> 
>
> *   **[] 表达式**
>
> [root@localhost ~]# [ 1 -eq 1 ] && echo 'ok'          
> ok
>
> [root@localhost ~]# [ 2 < 1 ] && echo 'ok'                 
> -bash: 2: No such file or directory
>
> [root@localhost ~]# [ 2 \< 1 ] && echo 'ok'
>
> [root@localhost ~]# [ 2 -gt 1 -a 3 -lt 4 ] && echo 'ok'
>
> ok    
>
> [root@localhost ~]# [ 2 -gt 1 && 3 -lt 4 ] && echo 'ok'  
> -bash: [: missing `]' 注意：在[] 表达式中，常见的>,<需要加转义字符，表示字符串大小比较，以acill码 位置作为比较。 不直接支持<>运算符，还有逻辑运算符|| && 它需要用-a[and] –o[or]表示
>
> 

> 
>
> *   **[[]] 表达式**
>
> [root@localhost ~]# [ 1 -eq 1 ] && echo 'ok'          
> ok
>
> [root@localhost ~]$ [[ 2 < 3 ]] && echo 'ok'
> ok
>
> [root@localhost ~]$ [[ 2 < 3 && 4 > 5 ]] && echo 'ok'
> ok

> 
>
> 注意：[[]] 运算符只是[]运算符的扩充。能够支持<,>符号运算不需要转义符，它还是以字符串比较大小。里面支持逻辑运算符：|| &&
>


```
echo "========= 逻辑表达式 test ========="
#注意：所有字符与逻辑运算符直接用“空格”分开，不能连到一起。
if test 3 -eq 3 -a 3 == 3 ;then echo "true" ;fi

#当3 大于 2 或 4 大于 3 并且 bxp 不等于 bixiaopeng  或 变量website不为空时,为真
if test 3 > 2 -a 4 -gt 2 -a "bxp" != "bixiaopeng" -o -n "$website" ;then echo "true"; else echo "false"; fi

#判断文件是否存在
if test -f "/Users/bixiaopeng/justtest.txt" ;then echo "true"; else echo "false"; fi
#判断目录是否存在
if test -d "/Users/bixiaopeng" ;then echo "true"; else echo "false"; fi

echo "========= 逻辑表达式 [] ========="

#在[] 表达式中，常见的>,<需要加转义字符，表示字符串大小比较，以acill码位置作为比较。
#不直接支持<>运算符，还有逻辑运算符 || 和 && 它需要用-a[and] –o[or]表示。
if [ 3 -eq 3 -a 3 == 3 ];then echo "true" ;fi

#当3 大于 2 或 4 大于 3 并且 bxp 不等于 bixiaopeng  或 变量website不为空时,为真
if [ 3 \> 2 -a 4 -gt 2 -a "bxp" != "bixiaopeng" -o -n "$website" ] ;then echo "true"; else echo "false"; fi

#判断文件是否存在
if [ -f "/Users/bixiaopeng/justtest.txt" ] ;then echo "true"; else echo "false"; fi
#判断目录是否存在
if [ -d "/Users/bixiaopeng" ] ;then echo "true"; else echo "false"; fi

echo "========= 逻辑表达式 [[]] ========="

#[[]] 运算符只是[]运算符的扩充。能够支持<,>符号运算不需要转义符，它还是以字符串比较大小。里面支持逻辑运算符 || 和 &&
if [[ 3 -eq 3 && 3 == 3 ]];then echo "true" ;fi

#当3 大于 2 或 4 大于 3 并且 bxp 不等于 bixiaopeng  或 变量website不为空时,为真
if [[ 3 > 2 && 4 -gt 2 && "bxp" != "bixiaopeng" || -n "$website" ]] ;then echo "true"; else echo "false"; fi

#判断文件是否存在
if [[ -f "/Users/bixiaopeng/justtest.txt" ]] ;then echo "true"; else echo "false"; fi
#判断目录是否存在
if [[ -d "/Users/bixiaopeng" ]] ;then echo "true"; else echo "false"; fi

#[[]] 中可以使用通配符,不需要引号
[[ $myname = b*peng ]] && echo "true"
```

**三、性能比较**

bash的条件表达式中有三个几乎等效的符号和命令：test，[]和[[]]。通常，大家习惯用if [];then这样的形式。而[[]]的出现，根据ABS所说，是为了兼容><之类的运算符。以下是比较它们性能，发现[[]]是最快的。

$ time (for m in {1..100000}; do test -d .;done;)
real    0m0.658s
user    0m0.558s
sys     0m0.100s

$ time (for m in {1..100000}; do [ -d . ];done;)
real    0m0.609s
user    0m0.524s
sys     0m0.085s

$ time (for m in {1..100000}; do [[ -d . ]];done;)
real    0m0.311s
user    0m0.275s
sys     0m0.036s

不考虑对低版本bash和对sh的兼容的情况下，用[[]]是兼容性强，而且性能比较快，在做条件运算时候，可以使用该运算符。

## Shell脚本中判断输入变量或者参数是否为空的方法
1. 判断变量
  ```
  read -p "input a word :" word
  if  [ ! -n "$word" ] ;then
      echo "you have not input a word!"
  else
      echo "the word you input is $word"
  fi
  ```
2. 判断输入参数
  ```
  #!/bin/bash
  if [ ! -n "$1" ] ;then
      echo "you have not input a word!"
  else
      echo "the word you input is $1"
  fi
  ```
3. 直接通过变量判断

如下所示:得到的结果为: IS NULL
	​```
	#!/bin/sh
	para1=
	if [ ! $para1 ]; then
	  echo "IS NULL"
	else
	  echo "NOT NULL"
	fi 
	​```
4. 使用test判断

得到的结果就是: dmin is not set! 
	​```
	#!/bin/sh
	dmin=
	if test -z "$dmin"
	then
	  echo "dmin is not set!"
	else  
	  echo "dmin is set !"
	fi
	​```
5. 使用""判断
  ```
  #!/bin/sh 
  dmin=
  if [ "$dmin" = "" ]
  then
    echo "dmin is not set!"
  else  
    echo "dmin is set !"
  fi
  ```

zhezhelin
linux shell 中判断字符串为空的正确方法
help命令可以查看帮助

help test

 

正确做法：
 
```
#!/bin/sh

STRING=

if [ -z "$STRING" ]; then 
    echo "STRING is empty" 
fi

if [ -n "$STRING" ]; then 
    echo "STRING is not empty" 
fi

 

root@james-desktop:~# ./zerostring.sh 
STRING is empty

````
-------------------------------------------------------------------------

错误做法：
```
#!/bin/sh

STRING=

if [ -z $STRING ]; then 
    echo "STRING is empty" 
fi

if [ -n $STRING ]; then 
    echo "STRING is not empty" 
fi 
```
## Linux命令之exit - 退出当前shell【返回值状态】

使用示例
示例一 退出当前shell
```
[root@new55 ~]# [root@new55 ~]# exit logout
```
 

示例二 在脚本中，进入脚本所在目录，否则退出
```
cd $(dirname $0) || exit 1 
cd $(dirname $0) || exit 1
``` 

示例三 在脚本中，判断参数数量，不匹配就打印使用方式，退出
```
if [ "$#" -ne "2" ]; then 
    echo "usage: $0 <area> <hours>" 
    exit 2 
fi 
if [ "$#" -ne "2" ]; then
    echo "usage: $0 <area> <hours>"
    exit 2
fi
``` 

示例四 在脚本中，退出时删除临时文件
```
trap "rm -f tmpfile; echo Bye." EXIT 
trap "rm -f tmpfile; echo Bye." EXIT
``` 

示例五 检查上一命令的退出码
```
./mycommand.sh 
EXCODE=$? 
if [ "$EXCODE" == "0" ]; then 
    echo "O.K" 
fi 
./mycommand.sh
EXCODE=$?
if [ "$EXCODE" == "0" ]; then
    echo "O.K"
fi
```

### 使用trap和kill退出整个脚本

cat >test.sh<<EOF''
#!/bin/bash

export TOP_PID=$$
trap 'exit 1' TERM

exit_script(){
    kill -s TERM $TOP_PID
}

echo "before exit"
:|exit_script
echo "after exit"
EOF

chmod a+x test.sh
./test.sh
echo $?