---
date: "2019-04-25T04:24:56-07:00"
draft: true
title: "K8s-Introduce"
tags: []
series: []
categories: []
toc: true
---
## k8s
kubernetes，简称K8s，是用8代替8个字符“ubernete”而成的缩写。是一个开源的，用于管理云平台中多个主机上的容器化的应用，Kubernetes的目标是让部署容器化的应用简单并且高效(powerful),Kubernetes提供了应用部署，规划，更新，维护的一种机制。

>可移植: 支持公有云，私有云，混合云，多重云(multi-cloud)
>可扩展: 模块化, 插件化, 可挂载, 可组合
>自动化: 自动部署，自动重启，自动复制，自动伸缩/扩展

**kubernetes 主要功能:**

- 数据卷
- 应用程序健康检查
- 复制应用程序实例
- 弹性伸缩
- 服务发现
- 负载均衡
- 滚动更新
- 服务编排
- 资源监控
- 提供认证和授权

## k8s组件有哪些，其功能是什么
master: kube-apiserver、manage-controller、scheduler,以及etcd
node: kubelet、kube-proxy,以及docker

## 版本控制

| Version | Action                   | Time       |
| ------- | ------------------------ | ---------- |
| 1.0     | Init                     | 2019-04-25T04:24:56-07:00|
