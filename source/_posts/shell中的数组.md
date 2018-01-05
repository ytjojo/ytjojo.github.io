

>ash 提供了一维数组变量。任何变量都可以作为一个数组；内建命令 declare 可以显式地定义数组。数组的大小没有上限，也没有限制在连续对成员引用和赋值时有什么要求。数组以整数为下标，从 0 开始。

### 1. 定义和初始化数组
下面的示例总结了如何定义一个数组和如何来初始化数组：

```
declare -a array #显示声明了数组array
delcare -a array[10] #数组大小没有上限，所以定义时指定的大小会被忽略
array[key]=value #array[0]=one,array[1]=two
array=(value1 value2...) #value的形式都是[subscript]=string,下标和等号可以省略，示例如下。
array=（value1 value2 value3） #array[0]=value1,array[1]=value2,array[2]=value3
array=([0]=value1 [2]=value3 [3]=value[4])
array=()#定义空数组
array="one two three" # echo ${array[0|@|*]},把array变量当作数组来处理，但数组元素只有字符串本身
```
字符串转数组，分隔符为,逗号，
```
selectModules="ngr_abc,ngr_cbd,ngr_xyz"
libNames=(`echo $selectModules|sed -r 's/,/ /g'|sed -r 's/\"//g'`)
```
字符串转数组，分隔符为\n换行符号，
```
selectModules="ngr_abc\nngr_cbd\nngr_xyz"
libNames=(`(echo $selectModules) |awk '{printf $0" "}' `)
echo ${libNames[0]}

```

```
selectModules="ngr_abc#ngr_cbd#ngr_xyz"
libNames=($(echo $selectModules | tr '#' ' ' | tr -s ' '))
echo ${libNames[1]}
```
读取文件 以换行符号\n 分割字符串组装成数组,每行当初数组的元素
```
array=(`cat hello.txt |sed ':jix;N;s/\n/ /g;b jix'`)
echo ${array[@]}

#awk方式
libNames=(`cat hello.txt |awk '{printf $0" "}'`)
echo ${libNames[0]}

```
分号分割
```
XiaoCh="xiao;j un;yu"                                                                                                                      
OldIFS=$IFS                                                                                                                              
IFS=$';'                                                                                                                                 
XiaoArr=($XiaoCh)                                                                                                                          
                                                                                                                                           
for i in ${XiaoArr[@]}                                                                                                                     
do                                                                                                                                         
    echo "$i"                                                                                                                              
done                                                                                                                                       
                                                                                                                                           
IFS=$OldIFS  
```

### 2. 数组的访问
数组的任何元素都可以用${array[subscript]}来引用，花括号是必须的，以避免和路径扩展冲突。 
如果 subscript 是@或是*，它扩展为array的所有成员。这两种下标只有在双引号中才不同。在双引号中，${name[*]}扩展为一个词，由所有数组成员的值组成，用特殊变量IFS的第一个字符分隔数组成员；${array[@]}将array的每个成员扩展为一个词。 如果数组没有成员，${name[@]} 扩展为空串。
```
#!/bin/bash

arr=("one" "two")
for i in "${arr[@]}"
do
    echo ${i}
done

${array[1]}  # ${array[key]} 数组index为1的元素

```

### 3. 数组的删除
用unset来进行数组的删除，示例如下：
```
unset array[2] #删除第三个成员
unset array #删除整个数组
```

### 4. 数组的长度
````
${#arr[@]}
${#arr[*]}
${#array[0]} #同上。 ${#array[*]} 、${#array[@]}。注意同#{array:0}的区别
# 取得数组单个元素的长度
lengthn=${#array_name[1]}#获取第二个元素的长度
${#arr} #错误的。这个获取的是数组第一个成员的长度。
```
### 5. 数组的”切片”操作
获取数组的“子串“用${arr[@]:n:m}来表示，如果没有:m那么就获取从下标n开始到最后一个元素的“字串“，示例如下：

#!/bin/bash

echo ${arr[@]:2}
echo ${arr[@]:1:3}
echo ${arr[@]:0}#打印所有
${array[@]:1} # two three four,除掉第一个元素后所有元素，那么${array[@]:0}表示所有元素

### 6. 关联数组
shell中还可以声明一个关联数组，普通数组只能使用整数作为数组的索引，而关联数组则使用字符串作为数组的索引。这个关联数组是不是有点其它语言中的字典的意思呢，o(∩∩)o…哈哈
```
#!/bin/bash

declare -A ass_arr
ass_arr["apple"]=12
ass_arr["orange"]=19

for t in "${ass_arr[@]}"
do
    echo ${t}
done

echo "ass_arr[\"orange\"]="${ass_arr["orange"]}
```

上面的示例输出如下：

```
ass_arr[“orange”]=19
```

从上面的输出来看关联数组输出的顺序跟普通数组有一些不同，关联数组是从最后一个成员开始输出。 
关联数组使用字符串作为索引，有时候我们需要获取数组的所有索引，可以用如下的方式来获取：
```
#!/bin/bash

declare -A ass_arr
ass_arr["apple"]=12
ass_arr["orange"]=19

echo ${!ass_arr[@]} #or ${!ass_arr[*]}
```
输出如下：
```
apple orange
```

### 7. 子串替换

子串替换
```
 array=( [0]=one [1]=two [2]=three [3]=four )
#第一个匹配到的，会被删除

echo ${array[@] /o/m}
mne twm three fmur
#所有匹配到的，都会被删除
echo ${array[@] //o/m}
mne twm three fmur
#没有指定替换子串，则删除匹配到的子符
echo ${array[@] //o/}
ne tw three fur
#替换字符串前端子串
echo ${array[@] /#o/k}
kne two three four
#替换字符串后端子串
echo ${array[@] /%o/k}
one twk three four

### 8. 判断数组是否包含元素

```
array=(abc sss ccc ddd)
target="ccc"
if [[ "${array[@]}" =~ $target ]];
then
 echo "包含"
else
 echo "不包含"
fi

target="aaaa"
if echo "${array[@]}" | grep -w "$target" &>/dev/null; then
 echo "包含"; 
else
 echo "不包含"
fi

```