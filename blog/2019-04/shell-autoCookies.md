---
date: "2019-04-25T04:26:27-07:00"
draft: false
title: "Shell-AutoCookies"
tags: [shell]
#series: []
categories: [“编码人生”]
toc: true
---

## SHELL
Shell 是操作系统的最外层，shell合并编程语言以控制进程和文件，以及启动和控制其他程序，shell通过提示用户输入可以向操作系统解释输入然后处理来自操作系统的任何结果输出。

Shell可以解释我们敲入的单条命令或者是多条命令的组合，提交给内核来处理，将处理的结果直接反馈给用户，即交互式的方式。另外一种方式是编程的方式，用户可以将要执行的命令按照自己的逻辑组织成一个脚本，然后将整个脚本提交给shell，带shell解析完成后，回馈给用户。

shell脚本是纯文本文件，可以使用任何文本编辑器编写
shell脚本通常是以.sh 作为后缀名，类似于windows下的.bat批处理文件


## 语法
Shell语法对空格敏感，要多注意规范

### 变量
变量命名大小写，以字母或者下划线开头，后面可以跟字母数字下划线，其他字符标志着变量名结束

局部变量A.sh
>A=100;echo $A

转换为全局变量B.sh
>export A
>./A.sh
设置全局变量 ： 
>export　M2＝200 
>M2=200; export M2

只读变量: 指不能被清除或重新赋值的变量
>readonly myvar

清除变量 ： unset variable

显示所有变量 ： set

显示环境变量 : env



位置参量（命令行参数）
|符号|含义|
|:-:|:-:|
|$0|当前脚本名|
|$1-$9|第1个到第9个位置变量|
|`${10}`|第10个位置变量，第11个`${11}`|
|$#|位置参量的数量|
|$*|以单字符串显示所有位置参量|
|`$@`|未加双引号与$*含义相同|
|$$|脚本运行时的当前进程号|
|$!|最后一个后台运行的进程的进程号|
|$?|上一个脚本或者命令的推出状态，0表示成功，其他值表示错误|

### 数组
数组最多支持二维?

arr=(math english chinese)

引用变量：${arr[0]} 

数组个数：${#arr[*]}

所有元素：${arr[*]}

数组赋值：arr[0]=chemical


### 输入输出
```shell
read name  #输入名字

echo "What is your name ?"  
# What is your name ?

printf "Hello world" 
#Hello world

printf "%-5s %-10s %-4s\n" 
#No Name Mark

echo -e "\e[1;35m This is red text \e[0m" 
#\e[1;35 将颜色设置为紫色，\e[0m将颜色置回

echo "What is your name ?" >name.txt  
cat name.txt
#输出：What is your name ?

echo "Hello,limit !" >>name.txt
cat name.txt
#输出：
#What is your name ?
#Hello,limit!
```

### 条件测试
```shell
if [ condition ];then

    command
fi

if condition; then

    command
elif condition2

    command2

else confition3

    command3
fi

case 变量 in
值1）
命令（命令组）
;;
值2）
命令（命令组）
;;
*)
命令（命令组）
;;
esac
```

### 控制语句

```shell
while (( $#>0 ))
do
echo $1
shift
done

for index in arr
do

done
```
### 函数
```shell
function func(){

    echo "hello"
    return 0
}
func
if [ $? -eq 0];then

    echo "successful"
fi
```

### 内部关键字
#### 操作符

- 整数操作符
  - -eq #等于
  - -ne #不等于
  - - -gt #大于
  - -lt #小于
  - -ge #大于等于
  - -le #小于等于

- 字符串操作符
  - = #检测两个字符串是否相等，相等返回- true
  - != #检测两个字符串是否相等，不想等返回true
  - -z #检测字符串长度是否为0，为0返回true
  - -n #检测字符串长度是否为0，不为0返回true
  - str #检测字符串是否为空，不为空返回true
- 文件测试运算符
    - -d file #检测文件是否是目录，如果是则返回true
    - -f file #检测文件是否是普通文件，如果是则返回true
    - -r file #检测文件是否是可读，如果是则返回true
    - -w file #检测文件是否可写，如果是则返回true
    - -x file #检测文件是否可执行，如果是则返回true
    - -s file #检测文件是否为空（文件大小是否大于0），如果不是则返回true
    - -e file #检测文件是否存在（包括目录），如果不是则返回true

- 布尔运算符
    - ！ #非运算，表达式为true,则返回false
    - -o #或运算，有一个表达式为true,则返回true
    - -a #与运算，两个表达式为true,才返回true
      - >$ if true;then echo "YES"; else echo "NO"; fi
        >YES
        

- 逻辑运算符
    - && #与运算，逻辑的and
    - || #或运算，逻辑的or


`[[ ]]`运算,如果是条件运算，还需要加一层[]对条件表达式的返回值做bool转换。

#### 算术运算
```shell
#!/bin/sh
num=121213232
echo "obase=2;$num" | bc #转换为二进制

num1=1110001010
echo "obase=10;ibase=2;$num1" | bc #转换为十进制

echo "sqrt(100)" | bc  #平方根，结果为10
echo "10^2" | bc #2次方，结果为100

$ echo "scale=3; 1/13"  | bc
#.076

$ echo "1 13" | awk '{printf("%0.3f\n",$1/$2)}'
#0.077

$ echo $RANDOM
#81

##srand() 在无参数时，采用当前时间作为 rand() 随机数产生器的一个 seed 。
$ echo "" | awk '{srand(); printf("%f", rand());}'
#0.237788

```

#### seq
它可以产生一系列数，你可以指定数的递增间隔，也可以指定相邻两个数之间的分割符

```shell
# wget一系列书籍
$ for i in `seq -f"http://thns.tsinghua.edu.cn/thnsebooks/ebook73/%02g.pdf" 1 21`;do wget -c $i; done

$ for i in `seq -w 1 21`;do wget -c "http://thns.tsinghua.edu.cn/thnsebooks/ebook73/$i"; done

#BASH3.0 以上版本支持
$ for i in {1..5}; do echo -n "$i "; done
#1 2 3 4 5

```
#### grep
```
datafile.txt：
northwest   NW     Charles Main          3.0          .98          3          34
western     WE     Sharon Gray           5.3          .97          5          23
southwest   SW     Lewis Dalsass         2.7          .8           2          18
southern    SO     Suan Chin             5.1          .95          4          15
eastern     EA     TB Savage             4.4          .84          5          20
northeast   NE     AM Main Jr            5.1          .94          3          13
north       NO     Margot Weber          4.5          .89          5          9
central     CT     Ann Stephens          5.7          .94          5          13

```
---
```shell
#打印所有名字以J开头的行
grep '^J' datebook

#打印所有以700结尾的行
grep '700$' datebook

#打印所有不包含834的行
grep -v "834" datebook

#打印所以包含一个大写字母，后跟4个小写字母，一个逗号，一个空格和一个大写字母的行
grep '[A-Z][a-z]\{4\}\,[[:space:]][A-Z]' datebook

#打印包含有Lincoln或lincoln（注意：grep不区分大小写）的行
grep '[Ll]incoln' datebook

# 去掉空行
grep -v ^$
```
#### uniq
uniq -c：统计相同行的个数，即每个单词的个数

#### sort
sort -n -k 1 -r：按照第一列 -k 1 的数字 

-n 逆序 -r 排序

#### head/tail
head -10：取出前十行

tail -10：取出尾十行
#### sed
选项与参数：

-n ：使用安静(silent)模式。在一般 sed 的用法中，所有来自 STDIN 的数据一般都会被列出到终端上。但如果加上 -n 参数后，则只有经过sed 特殊处理的那一行(或者动作)才会被列出来。

-e ：直接在命令列模式上进行 sed 的动作编辑；

-f ：直接将 sed 的动作写在一个文件内， -f filename 则可以运行 filename 内的 sed 动作；

-r ：sed 的动作支持的是延伸型正规表示法的语法。(默认是基础正规表示法语法)

-i ：直接修改读取的文件内容，而不是输出到终端。

sed命令功能：

p 打印

d 删除行

g 取出暂存缓冲区的内容，将其复制到模式空间，覆盖该处的原有内容

s 用一个字符串替换另一个

替换标志：

g在行内进行全局替换

p打印行

w将行写入文件

x交换暂存缓冲区与模式空间的内容

y将字符转换为另一字符（不能对正则表达式使用y命令）

```
1.把Jon的名字改为Jonathan
sed -n 's/Jon/Jonathan/p' datebook

2.把头3行删除
sed '1,3d' datebook

3.打印第5~10行
sed -n  '5,10p' datebook

4.删除含有Lane的所有行
sed '/Lane/d' datebook

9.删除所有空行
sed '/^ *$/d' datebook    # 如果空行中包含了有space和Tab键，则此命令也成立
```

#### awk
```shell
awk '{print $1 $2}'

$ var="get the length of me"
$ echo $var | awk '{printf("%d\n", match($0,"the"));}'
5

$ awk '/consists of/{ printf("%s:%d:%s\n",FILENAME, FNR, $0)}' text  #看到没？和grep的结果一样
$ sed -n -e '/consists of/=;/consists of/p' text #同样可以打印行号
```

#### expr  
expr的每个参数之间必须用空格分隔，特殊字符无需转义
举例：expr "$number" + 0


#### & 
放在启动参数后面表示设置此进程为后台进程

#### select
```
select var in worldlist
do
命令（或命令组）
done
```
举例：
```shell
#!/bin/bash
#Scriptname:runit
ps3="Select a program to execute:"
select program in `ls -F` pwd date
do
$program
done
```

#### shif

将参量左移指定的次数，未加参数时表示左移一次，移动后，左端的参数被删除
举例：
```shell
set joe mary tom sam
shift
echo $*
#结果显示：mary tom sam
```
## 规范

> #!/bin/bash 就可以直接./bash_script执行

把bash手册输出为pdf文件
>man -t bash | ps2pdf - bash.pdf

## 范例
### 用awk过滤docker容器ip

~~~shell
#!/bin/bash
#docker ps |awk '{if($2=="hyperledger/fabric-kafka") print $1}'
#docker ps |awk '{if($2=="hyperledger/fabric-kafka"){ print $1;cmd="docker exec  "$1" ifconfig";system(cmd);}}'|grep addr:|awk '{print $2}'|awk -F ":" '{print $2}'
#docker ps |awk '{if($2=="hyperledger/fabric-kafka"){ print $1;cmd="docker exec  "$1" ifconfig";system(cmd);}}'|grep addr:|awk '{print $2}'|awk -F '[ :]' '{print $2}'
docker ps |awk '{if($2=="hyperledger/fabric-kafka"){ print $1;cmd="docker exec  "$1" ifconfig";system(cmd);}}'|awk '{if($2 ~/addr:*/ && $2!~/addr:127.0.0.1/){print $2}}'|awk -F '[ :]' '{print $2}'

~~~
---
out
~~~shell
root@liu:~/work/go/src/github.com/hyperledger/fabric/examples/e2e_cli# ./catip.sh 
172.18.0.8
172.18.0.7
172.18.0.6
172.18.0.5
root@liu:~/work/go/src/github.com/hyperledger/fabric/examples/e2e_cli# 
~~~

### 处理URL地址

```shell
$ url="ftp://anonymous:ftp@mirror.lzu.edu.cn/software/scim-1.4.7.tar.gz"

#匹配URL地址，判断URL地址的有效性
$ echo $url | grep "ftp://[a-z]*:[a-z]*@[a-z\./-]*"

#截取服务器类型
$ echo ${url%%:*}
ftp
$ echo $url | cut -d":" -f 1
ftp

#截取域名
$ tmp=${url##*@} ; echo ${tmp%%/*}
mirror.lzu.edu.cn

#截取路径
$ tmp=${url##*@} ; echo ${tmp%/*}
mirror.lzu.edu.cn/software

#截取文件名
$ basename $url
scim-1.4.7.tar.gz
$ echo ${url##*/}
scim-1.4.7.tar.gz

#截取文件类型（扩展名）
$ echo $url | sed -e 's/.*[0-9].\(.*\)/\1/g'
tar.gz

```

### 查看进程
```shell
#查看指定程序名的进程ID
$ sleep 100 &
$ pidof sleep
#查看进程的内存映像
$ cat /proc/9298/maps
#打印 cpu 使用率最高的前 4 个程序
$ ps -e -o "%C %c" | sort -u -k1 -r | head -5
 7.5 firefox-bin
 1.1 Xorg
 0.8 scim-panel-gtk
 0.2 scim-bridge

#获取使用虚拟内存最大的 5 个进程：
$ ps -e -o "%z %c" | sort -n -k1 -r | head -5
349588 firefox-bin
 96612 xfce4-terminal
 88840 xfdesktop
 76332 gedit
 58920 scim-panel-gtk

#系统所有进程之间都有“亲缘”关系，可以通过 pstree 查看这种关系
$ pstree

#用top动态查看进程信息
$ top

#确保特定程序只有一个副本在运行
pidfile=/tmp/$0".pid"
if [ -f $pidfile ]; then
       OLDPID=$(cat $pidfile)
    ps -e -o "%p" | tr -d " " | grep -q "^$OLDPID$"
    [ $? -eq 0 ] && exit
fi

echo $$ > $pidfile

#... 代码主体

#设置信号0的动作，当程序退出时触发该信号从而删除掉临时文件
trap "rm $pidfile"      0

#获取进程优先级
$ ps -e -o "%p %c %n" | grep xfs
 5089 xfs               0

#结束进程
$ sleep 50 &   #启动一个进程
[1] 11347
$ kill 11347   #结束进程

#暂停某个进程
$ sleep 50 &
[1] 11441
$ jobs
[1]+  Running                 sleep 50 &
$ kill -s SIGSTOP 11441   #这个等同于我们对一个前台进程执行CTRL+Z操作
$ jobs
[1]+  Stopped                 sleep 50
$ kill -s SIGCONT 11441   #这个等同于之前我们使用bg %1操作让一个后台进程运行起来
$ jobs
[1]+  Running                 sleep 50 &
$ kill %1                  #在当前会话(session)下，也可以通过作业号控制进程
$ jobs
[1]+  Terminated              sleep 50

#无名管道（pipe）
$ ps -ef | grep init

#有名管道（named pipe）
$ mkfifo fifo_test    #通过mkfifo命令创建一个有名管道
$ echo "fewfefe" > fifo_test
#试图往fifo_test文件中写入内容，但是被阻塞，要另开一个终端继续下面的操作
$ cat fifo_test        #另开一个终端，记得，另开一个。试图读出fifo_test的内容
fewfefe

#使用 Shell 内置命令 fg 把作业 1 调到前台运行，然后按下 CTRL+Z 让该进程暂停
$ fg %1
sleep 50
^Z
[1]+  Stopped                 sleep 50

#启动停止的进程并运行在后台
$ bg %1
[2]+ sleep 50 &



```
### 消费者模式
一种应用架构非常适合本地的多程序任务设计，如果结合 web cgi，那么也将适合远程控制的要求。引入 web cgi 的唯一改变是，要把控制程序 ./control.sh 放到 web 的 cgi 目录下，并对它作一些修改，以使它符合 CGI 的规范，这些规范包括文档输出格式的表示（在文件开头需要输出 content-tpye: text/html 以及一个空白行）和输入参数的获取 (web 输入参数都存放在 QUERY_STRING 环境变量里头）

```shell
$ cat app.sh
#!/bin/bash

FIFO=fifo_test
while :;
do
    CI=`cat $FIFO`  #CI --> Control Info
    case $CI in
        0) echo "The CONTROL number is ZERO, do something ..."
            ;;
        1) echo "The CONTROL number is ONE, do something ..."
            ;;
        *) echo "The CONTROL number not recognized, do something else..."
            ;;
    esac
done

$ cat control.sh
#!/bin/bash

FIFO=fifo_test
CI=$1

[ -z "$CI" ] && echo "the control info should not be empty" && exit

echo $CI > $FIFO

#一个程序通过管道控制另外一个程序的工作
$ chmod +x app.sh control.sh    #修改这两个程序的可执行权限，以便用户可以执行它们
$ ./app.sh  #在一个终端启动这个应用程序，在通过./control.sh发送控制码以后查看输出
The CONTROL number is ONE, do something ...    #发送1以后
The CONTROL number is ZERO, do something ...    #发送0以后
The CONTROL number not recognized, do something else...  #发送一个未知的控制码以后
$ ./control.sh 1            #在另外一个终端，发送控制信息，控制应用程序的工作
$ ./control.sh 0
$ ./control.sh 4343


#CGI 控制程序
#!/bin/bash
FIFO=./fifo_test
CI=$QUERY_STRING

[ -z "$CI" ] && echo "the control info should not be empty" && exit

echo -e "content-type: text/html\n\n"
echo $CI > $FIFO

#在实际使用时，请确保 control.sh 能够访问到 fifo_test 管道，并且有写权限，以便通过浏览器控制 app.sh
>> "http://ipaddress\_or\_dns/cgi-bin/control.sh?0"
#问号 ? 后面的内容即 QUERY_STRING，类似之前的 $1 

```
### 网络
```shell
#通过dhclient获取IP地址
$ dhclient ethN

#静态配置IP地址
$ ifconfig eth0 ip_address on
$ route add deafult gw gw_ip_address

```

### 调试菜单的模板
```shell
function CDAN(){  
#EOF是“end of file”，表示文本结束符。
#<<EOF
#（内容）
#EOF

#首先必须要说明的是EOF在这里没有特殊的含义，你可以使用FOE或OOO等（当然也不限制在三个字符或大写字符）。
#可以把EOF替换成其他东西，意思是把内容当作标准输入传给程
#结合这两个标识，即可避免使用多行echo命令的方式，并实现多行输出的结果。

cat << someone    
+------------------------------------------------+  
|                                                |  
|        _o0o_          1. 安装Nginx             |  
|        08880          2. 安装Apache            |  
|       88"."88         3. 安装MySQL             |  
|       (|-_-|)         4. 安装PHP               |  
|        0\=/0          5. 部署LNMP环境          |  
|      __/   \__        6. 安装zabbix监控        |  
|     ‘\   ///‘       7. 退出此管理程序         |  
|    / Linux一键 \      8. 关闭计算机            |  
|  ||    Server   ||    ======================   |    
|  \        ////         一键安装服务           |  
|   |||  i i i    |||               by someone   |  
|   ___        ___      ======================   |  
|___‘.  /--.--\ .‘___                            |  
+------------------------------------------------+  
someone  
}  
CDAN  
  
LOG_DIR=/usr/local/src  
read -p "请您输入1-8任意数值:" NUM  
  
if [ ${#NUM} -ne 1 ]  
  then  
        echo "请您输入1|2|3|4|5|6|7|8"  
        exit 1  
fi  
  
expr $NUM + 1 &>/dev/null  
if [ "$?" -ne 0 ]  
   then  
        echo "请您输入数值！"  
        exit 1  
fi  
  
if [ "$NUM" -gt 8 ];then  
        echo "请您输入比8小的数值"  
        exit 1  
elif [ "$NUM" -eq 0 ];then  
        echo "请您输入比0大的数值"  
        exit 1  
fi  
######################  
function Nginx_DIR() {  
        yum install -y gcc gcc-c++ pcre-devel zlib-devel openssl-devel &>/dev/null  
        if [ $? -eq 0 ]  
           then  
                cd $LOG_DIR  && wget http://nginx.org/download/nginx-1.12.2.tar.gz &>/dev/null && useradd -M -s /sbin/nologin nginx && \  
        tar zxf nginx-1.12.2.tar.gz && cd nginx-1.12.2/ && \  
                ./configure --prefix=/usr/local/nginx --with-http_dav_module --with-http_stub_status_module --with-http_addition_module --with-http_sub_module  --with-http_flv_module --with-http_mp4_module --with-pcre --with-http_ssl_module --with-http_gzip_static_module --user=nginx &>/dev/null && make &>/dev/null && make install &>/dev/null   
        fi  
  
        if [ -e /usr/local/nginx/sbin/nginx ];then  
                ln -s /usr/local/nginx/sbin/nginx /usr/local/sbin/ && nginx && echo "Nginx安装并启动成功!!!"  
        fi  
}  
  
if [ $NUM -eq 1 ]  
  then  
     echo "开始安装Nginx请稍等..." && Nginx_DIR  
fi


```
### 调试shell

```shell

sh -n script.sh #查看语法错误

sh -x script.sh #查看调用栈

#正确使用 source 和 .
仅使用 source 和 . 来执行你的环境配置等功能，建议不要用于其它用途。 在Shell中使用脚本时，使用 bash your_script.sh 而不是 source your_script.sh 或 . your_script.sh。

当使用 bash 的时候，当前的Shell会创建一个新的子进程执行你的脚本；当使用 source 和 . 时，当前的Shell会直接解释执行 your_script.sh 中的代码。如果 your_script.sh 中包含了类似 exit 0 这样的代码，使用source 和 . 执行会导致当前Shell意外地退出。

#多行注释
:<<EOF
注释内容...
注释内容...
注释内容...
EOF
```

## 版本控制

| Version | Action                   | Time       |
| ------- | ------------------------ | ---------- |
| 1.0     | Init                     | 2019-04-25T04:26:27-07:00|
