# Python、Shell

## Shell

### 基础

命名

```
test.sh
test.py

后缀不代表任何意义，只是代表你执行的这个是用什么脚本写的
```



脚本格式

```shell
#! /bin/bash
echo "hello world"
```

运行脚本

```shell
chmod 777 ./test.sh # 添加执行权限
chmod +x ./test.sh

sh ./test.sh 以shell的方式运行
sh -x ./test.sh 调试脚本
```

脚本运行方式

```shell
#! /bin/bash
#! 告诉系统其后路径所指定的程序即是解释此脚本文件的 Shell 程序

如果要使用python执行，则#！后面添加python的路径
which python # 找出python执行命令的位置
```

**shell特性**

输出重定向

```shell
>、1 > 表示把标准正确输出重定向到指定文件
>>、1>> 表示把标准正确输出追加到指定文件

2> 表示把标准错误输出重定向到指定文件
2>> 表示把标准错误输出追加到指定文件

例子：cat a.txt > b.txt 表示读取了a.txt文件并且重定向给了b.txt
ll /home/test/a.txt  /home/test/b.txt   1>/home/test/record-1.log   2>/home/test/record-2.log 表示把标准正确输出重定向到某个指定文件中，标准错误输出重定向到另外一个指定文件中

>/dev/null 和 &>/dev/null 等价
作用是将标准正确输出1重定向到/dev/null中。 /dev/null代表linux的空设备文件，所有往这个文件里面写入的内容都会丢失，俗称“黑洞”。那么执行了>/dev/null之后，标准正确输出的内容就会不再存在，没有任何地方能够找到输出的内容。
```

输入重定向

```shell
<、1<
<< 表示输入追加

例子：
cat < 1.txt > 2.txt 表示cat读取1.txt文件把它输出给了2.txt文件。此时2.txt文件内容和1.txt的一样
cat 1.txt > 2.txt 表示cat显示的内容重定向输出给2.txt文件。虽然结果一样，但表示的含义有点差别
```

管道

```shell
| 表示前者命令的标准输出交给后者命令输入
tee 表示用于读取标准输入的数据，并将其内容输出成文件

cat 1.log | grep linux | tee 1.txt
```

命令排序

```shell
; 没有逻辑关系，无论前面命令执行成功与否，后面命令一样执行 cat 1.txt;mkdir data
&& 有逻辑关系，前面命令执行成功，后面命令才会执行 cat 1.txt && mkdir data
|| 前面执行不成功，则执行后者 cat 1.txt || mkdir data
```

程序退出

```shell
exit  0：正常运行程序并退出程序；

exit  1：非正常运行导致退出程序；

exit 0 可以告知你的程序的使用者：你的程序是正常结束的。如果 exit 非 0 值，那么你的程序的使用者通常会认为
你的程序产生了一个错误。
在 shell 中调用完你的程序之后，用 echo $? 命令就可以看到你的程序的 exit 值。在 shell 脚本中，通常会根据
上一个命令的 $? 值来进行一些流程控制。0代表程序正确的执行
```

### 变量

自定义变量

```shell
变量名=变量值（a=1）
引用变量 $变量名或${变量名}($a或${a})

unset 表示取消变量
unset a
```

系统环境变量

```shell
export 变量名=变量值
export JAVA_HOME=‘/usr/local/java/bin’

谁都可以调用${JAVA_HOME}
```

位置参数变量

```shell
脚本参数传参：
$1 表示第一个参数
$2 表示第二个参数
...
${10} 表示第十个参数
```

预先定义变量

```shell
$0 脚本名
$* 所有参数
$@ 所有参数
$# 参数个数
$$ 当前进程
$! 上一个后台进程
$? 上一个命令的返回值，0表示成功
```

变量数值运算

```shell
expr $num1 + $num2

-lt 小于 less than
-le 小于等于 less equal
-gt 大于 great than
-ge 大于等于 great equal
-eq 等于 equal

let 运算命令，变量计算中不需要加上 $ 来表示变量，可以实现变量的自增
i=1
let i++
echo $i 会输出2
```

shell条件测试

```
格式1：test 条件表达式
格式2：if [条件表达式] [这是一个命令，which [
格式3：[[条件表达式]]
```



内置变量

```shell
break 结束当前循环
continue 跳过次此循环，进入下一次循环
exit 退出程序
```



### 条件判断

if else

单分支结构

```shell
if [ 如果你有房 ];then
	我就嫁给你
fi
```

双分支结构

```shell
if [ 如果你有房 ];then
		我就嫁给你
	else
		再见
fi
```

多分支结构

```shell
if [ 如果你有房 ];then
		我就嫁给你
elif [ 如果你有车 ];then
		我就嫁给你
elif [ 如果你有钱 ];then
		我就嫁给你
	else 
		再见
fi
```



### 循环语句

for、while

```shell
for 变量名 in [ 取值列表 ]
do
	循环体
done

for i in {1..10}
do
	echo "123"
done
```



### 流程控制

case

```shell
case 变量 in
模式 1）
	命令序列;;
模式 2）
	命令序列;;
模式 3）
	命令序列;;
*）
	无匹配后命令序列;;
esac

case $1 in
	RED HAT)
		echo "centos"
			;;
	centos)
		echo "RED HAT"
			;;
	*)
		echo "其他版本"
		exit 1
esac
```



### 函数

定义函数

```shell
function 函数名(){
	命令1
	命令2
	...
}

function关键字可省略
```

调用函数

```shell
直接用函数名
function sayHello(){
	echo "sayHello"
}
# 函数调用
sayHello
```

函数返回值

```
使用return关键字
获取上一个命令返回值的方式$?
```

函数传参

```shell
内置命令set给脚本指定位置参数的值（又叫重置）
一旦使用了set设置了传入参数的值，脚本将忽略运行时传入的位置参数（实际上是被set命令重置了位置参数的值）

函数名 $1 传入脚本后第一个参数
函数名 $* 接收所有参数的传递
```



## Python

### os模块



### python调用shell命令和shell脚本

#### os.system(cmd)

结果：执行完cmd命令，以及以及执行完毕后的退出状态，使用该命令无法将执行结果保存起来。

```shell
>>> import os
>>> os.system('ls')
0
>>> os.system('ls /opt')
module  softwa re
0
```

#### os.popen(cmd,mode,bufsize)

os.popen（）可以实现一个“管道”，从这个命令获取的值可以继续被使用。因为它返回一个文件对象，可以对这个文件对象进行相关的操作。

参数解析：

cmd:使用命令

mode:模式权限可以是 'r'(默认) 或 'w'。如果mode为’r'，可以使用此函数的返回值调用read()或readlines()来获取command命令的执行结果。

bufsize:指明了文件需要的缓冲大小：0意味着无缓冲；1意味着行缓冲；其它正值表示使用参数大小的缓冲

```shell
>>> import os
>>> p = os.popen('ls /opt')
>>> p.read()
'module\nsoftware\n'
>>> p.close()
```

#### commands模块

getstatusoutput(cmd):可以获得返回值和输出

getoutput(cmd)：只获得输出

getstatus(cmd)：只获得返回值

```shell
>>> import commands
>>> (status,output) = commands.getstatusoutput('ls /opt')
>>> print(status)
0
>>> print(output)
module
software
```

#### subporcess 这个模块

(前面三个够用了，等用到了再学)

#### python调用shell脚本并传递参数

##### python调用shell脚本，没有传参

```shell
vim hello.sh

#!/bin/bash
echo "hello world!"
exit 0

chmod +x hello.sh

>>> import os
>>> os.system('./hello.sh')
hello world!
0
```

##### shell脚本使用python脚本参数

```shell
vim hello.sh

#!/bin/bash
echo "hello world! ${1} ${2}"
exit 0

chmod +x hello.sh

[chenli@hadoop1 python]$ which python
/usr/bin/python

vim hello.py
#!/usr/bin/env python
# -*- coding: utf-8 -*-
import os
import sys

if(len(sys.argv)<3):
        print("please input two arguments!")
        sys.exit(1)
arg0=sys.argv[1]#argv[0] is .py file
arg1=sys.argv[2]
print(sys.argv[0])
os.system('./hello.sh'+' '+arg0+' '+arg1)


chmod +x hello.py

[chenli@hadoop1 python]$ python ./hello.py 
please input two arguments!
[chenli@hadoop1 python]$ python ./hello.py chenli chocolate
./hello.py
hello world! chenli chocolate
```

##### shell脚本传递参数

```shell
vim hello.sh

#!/bin/bash
echo "hello world! ${1}"
exit 0

chmod +x hello.sh

vim date.py
#!/usr/bin/env python
# -*- coding: utf-8 -*-
import os
import time

time=time.strftime("%Y-%m-%d",time.localtime())
os.system('./hello.sh'+' '+time)

chomd +x date.py

[chenli@hadoop1 python]$ python ./date.py 
hello world! 2021-12-05 

```

