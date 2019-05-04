---
date: "2019-04-24T06:45:07-07:00"
draft: false
title: "Go-国内下载官方工具"
tags: [“go”]
series: ["golang"]
categories: [“编码人生”]
toc: true
---

## 国内下载golang.org中的工具包

国内有某种不可描述的原因，导致无法访问 `golang.org`,但是使用golang时官方工具包是必不可少的一部分.

**golang.org/x/**这个目录，其实镜像托管在 **github.com/golang/**

### GIT迁移
将Github的包下载到本地后，移动到相应的目录中.

>如:
>net -->> GOPATH/src/golang.org/x/net
>crypto -->> GOPATH/src/golang.org/x/crypto
>build -->> GOPATH/src/golang.org/x/build
>...

### GIT下载

>在 $GOPATH/src/golang.org/x/ 下通过git 进行 clone
>如:
> ~/go/src/golang.org/x$ git clone http://github.com/golang/build

然后用git pull直接维护更新本地代码

## 版本控制

| Version | Action                   | Time       |
| ------- | ------------------------ | ---------- |
| 1.0     | Init                     | 2019-04-24T06:45:07-07:00|
