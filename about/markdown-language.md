---
date: "2019-04-25T09:36:07-07:00"
draft: true
title: "markdown-language"
tags: []
series: []
categories: []
toc: true
---
# h1
## h2
### h3
#### h4
##### h5
###### h6
####### h7      // 错误代码
######## h8     // 错误代码
######### h9    // 错误代码
########## h10  // 错误代码

一级标题
======================
二级标题
---------------------

[TOC]

> aaaaaaaaa
>> bbbbbbbbb
>>> cccccccccc

标记之外`hello world`标记之外

注：用```生成块
```
<div>   
    <div></div>
    <div></div>
    <div></div>
</div>
```
```javascript
var num = 0;
for (var i = 0; i < 5; i++) {
    num+=i;
}
console.log(num);
```

[百度1](http://www.baidu.com/" 百度一下"){:target="_blank"}   

[百度2][2]{:target="_blank"}
[2]: http://www.baidu.com/   "百度二下"

[图片上传失败...(image-90880b-1542510791300)]

![name][01]
[01]: ./01.png '描述'

[[图片上传失败...(image-f83b77-1542510791300)]](http://www.baidu.com){:target="_blank"}       // 内链式

[[图片上传失败...(image-4dc956-1542510791300)]][5]{:target="_blank"}                      // 引用式
[5]: http://www.baidu.com

1. one
2. two
3. three

* one
* two
* three

这是文字……

- [x] 选项一
- [ ] 选项二  
- [ ]  [选项3]

|    a    |       b       |      c     |
|:-------:|:------------- | ----------:|
|   居中  |     左对齐    |   右对齐   |
|=========|===============|============|

~~删除线~~

*斜体*


**加粗**

Z<sup>a</sup>
 
Z<sub>a</sub>

<kbd>Ctrl</kbd>

$$ x \href{why-equal.html}{=} y^2 + 1 $$
$ x = {-b \pm \sqrt{b^2-4ac} \over 2a}. $

***
---
* * *

Markdown[^1]
[^1]: Markdown是一种纯文本标记语言        // 在文章最后面显示脚注

[公式标题锚点](#1)

### [需要跳转的目录] {#1}    // 方括号后保持空格

Markdown 
:   轻量级文本标记语言，可以转换成html，pdf等格式  //  开头一个`:` + `Tab` 或 四个空格

代码块定义
:   代码块定义……

        var a = 10;         // 保持空一行与 递进缩进


<xxx@outlook.com>

```flow                     // 流程
st=>start: 开始|past:> http://www.baidu.com // 开始
e=>end: 结束              // 结束
c1=>condition: 条件1:>http://www.baidu.com[_parent]   // 判断条件
c2=>condition: 条件2      // 判断条件
c3=>condition: 条件3      // 判断条件
io=>inputoutput: 输出     // 输出
//----------------以上为定义参数-------------------------

//----------------以下为连接参数-------------------------
// 开始->判断条件1为no->判断条件2为no->判断条件3为no->输出->结束
st->c1(yes,right)->c2(yes,right)->c3(yes,right)->io->e
c1(no)->e                   // 条件1不满足->结束
c2(no)->e                   // 条件2不满足->结束
c3(no)->e                   // 条件3不满足->结束
```

tag=>type: content:>url         // 形参格式 
st=>start: 开始:>http://www.baidu.com[blank]  //实参格式

注：** st=>start: 开始 的：后面保持空格**



形参
实参
含义




tag
st
标签 (可以自定义)


=>
=>
赋值


type
start
类型  (6种类型)


content
开始
描述内容 (可以自定义)


:>url
http://www.baidu.com[blank]
链接与跳转方式 兼容性很差







6种类型
含义




start
启动


end
结束


operation
程序


subroutine
子程序


condition
条件


inputoutput
输出


->
->
连接


condition
c1
条件


(布尔值,方向)
(yes,right)
如果满足向右连接，4种方向：right ，left，up ，down 默认为：down


```flow                             // 流程
st=>start: 启动|past:>http://www.baidu.com[blank] // 开始
e=>end: 结束                      // 结束
op1=>operation: 方案一             // 运算1
sub2=>subroutine: 方案二|approved:>http://www.baidu.com[_parent]  // 运算2
sub3=>subroutine: 重新制定方案        // 运算2
cond1=>condition: 行不行？|request  // 判断条件1
cond2=>condition: 行不行？          // 判断条件2
io=>inputoutput: 结果满意           // 输出

// 开始->方案1->判断条件->
st->op1->cond1
// 判断条件1为no->方案2->判断条件2为no->重新制定方案->方案1
cond1(no,right)->sub2->cond2(no,right)->sub3(right)->op1
cond1(yes)->io->e       // 判断条件满足->输出->结束
cond2(yes)->io->e       // 判断条件满足->输出->结束
```

```sequence
A->>B: 你好
Note left of A: 我在左边     // 注释方向，只有左右，没有上下
Note right of B: 我在右边
B-->A: 很高兴认识你
```

```sequence
起床->吃饭: 稀饭油条
吃饭->上班: 不要迟到了
上班->午餐: 吃撑了
上班->下班:
Note right of 下班: 下班了
下班->回家:
Note right of 回家: 到家了
回家-->>起床:
Note left of 起床: 新的一天
```



注：operation (程序); subroutine (子程序) ;condition (条件)，都可以在括号里加入连接方向。




