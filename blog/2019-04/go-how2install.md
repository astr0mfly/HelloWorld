---
date: "2019-04-22T04:12:48-07:00"
draft: false
title: "Golang安装环境搭建"
tags: [“go”]
series: ["golang"]
categories: [“编码人生”]
toc: true
---
### Go安装包下载
由于Go的官方网址国内不可达，推荐一个国内的镜像源：
[go语言中文网.Go安装包下载](https://studygolang.com/dl)

### 编译安装
下载Linux版本：go1.12.4.linux-amd64.tar.gz
~~~ shell
#在下载文件夹下的Shell敲一下如下命令
sudo tar -C /usr/local -xzvf go1.12.4.linux-amd64.tar.gz
#/etc/profile（全系统安装）或 $HOME/.profile 添加环境变量
export PATH=$PATH:/usr/local/go/bin
~~~

也有另一种方法添加环境变量：

```shell
# vim /etc/profile.d/go.sh
	export PATH=$PATH:/usr/local/go/bin
	export GOPATH=$HOME/dev/go
	export GOBIN=$GOPATH/bin

   :wq!
# source /etc/profile.d/go.sh

```

### 验证结果
创建一个`hello.go`的文件
~~~ go
package main

import "fmt"

func main() {
    fmt.Printf("hello, world!\n")
}
~~~

然后用刚才安装好的go去运行
~~~ shell
go version
go version go1.12.4 linux/amd64

$ go run hello.go
hello, world!
~~~

### 工程目录

```
bin/
 hello                 # 可执行命令
pkg/
 linux_amd64/          # 这里会反映出你的操作系统和架构
  github.com/user/
   stringutil.a  # 包对象
src/
 github.com/user/
  hello/
   hello.go      # 命令源码
  stringutil/
   reverse.go       # 包源码

```

### 卸载Go
从你的系统中移除既有的Go安装，需删除 go 目录/usr/local/go;
从你的 PATH 环境变量中移除 Go 的 bin 目录, 编辑 /etc/profile 或 $HOME/.profile。


### 日常问题
解决sudo go command not found 报错
在这个 /etc/sudoers 文件里面，有一个secure_path配置，大家一看就知道了，它的意思当你使用 sudo+command 这种形式执行命令的时候会从其配置的路径里面寻找命令，肯定是没有你自定义的PATH的，这个主要是安全考虑。
解决方法有几种：

直接把自定义PATH路径配置在secure_path里面，简单粗暴，就是有点麻烦
将 Defaults env_reset 改成 Defaults !env_reset 取消掉对PATH变量的重置，然后在.bashrc中最后添加alias sudo='sudo env PATH=$PATH'，这个感觉更麻烦
直接把这3行注释掉，经测试完全没有任何问题

解决go install github.com/user/valens_blog: open /home/valens/dev/go/bin/valens_blog: permission denied
~~~ shell
sudo  chmod -R 777 ~/dev/go
~~~

设置GOPATH
~~~ shell
#设置当前目录为$GOPATH
alias gopath='export GOPATH=`pwd` && echo $GOPATH'
#尤其适合多个不同目录下的go项目,在每个项目下运行一次gopath,就设置好了当前的gopath,十分方便

~~~



## 版本控制

| Version | Action                   | Time       |
| ------- | ------------------------ | ---------- |
| 1.0     | Init                     | 2019-04-22T04:12:48-07:00|
