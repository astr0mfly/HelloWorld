---
date: "2019-04-25T09:19:17-07:00"
draft: true
title: "Distri-Paxos"
tags: []
series: []
categories: []
toc: true
---

[分布式一致性算法Paxos介绍](https://segmentfault.com/a/1190000005717258)

## Introduction
PAXOS可以用来解决分布式环境下，选举（或设置）某一个值的问题（比如更新数据库中某个user的age是多少）。分布式系统中有多个节点就会存在节点间通信的问题，存在着两种节点通讯模型：共享内存（Shared memory）、消息传递（Messages passing），Paxos是基于消息传递的通讯模型的。它的假设前提是，在分布式系统中进程之间的通信会出现丢失、延迟、重复等现象，但不会出现传错的现象。Paxos算法就是为了保证在这样的系统中进程间基于消息传递就某个值达成一致。
注意，在Paxos算法中是不考虑拜占庭将军问题的

## 版本控制

| Version | Action                   | Time       |
| ------- | ------------------------ | ---------- |
| 1.0     | Init                     | 2019-04-25T09:19:17-07:00|
