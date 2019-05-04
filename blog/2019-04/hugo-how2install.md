---
date: "2019-04-22T04:09:23-07:00"
draft: false
title: "使用hugo制作静态个人界面"
tags: ["hugo", "go", "web", "blog"]
series: ["个人博客搭建之路"]
categories: ["编码人生"]
toc: true
---

### 安装[hugo](https://www.gohugo.org/)
建议到如下链接下载对应文件，使用go get挺慢的。
https://github.com/gohugoio/hugo/releases

1. 安装hugo
2. 生成site并且 `clone` 对应皮肤的Git到`theme`文件夹下
3. 调试 ：`hugo server --theme=hyde --buildDrafts`
4. 打开 : `http://localhost:1313` 实时调试
5. 生成部署： `hugo --theme=hyde`
6. 上传到git ： `git push -u origin master`


### 实例皮肤：redlougue
hugo链接：
https://github.com/tmaiaroto/hugo-redlounge

### 路径解释
 - config.toml
站点全局的参数配置文件

 - archetypes
Hugo的markdown文件中前置数据Front Matter定义的结构，默认使用的是default.md文件，可以自定义，然后在config.toml中指定自定义的结构文件，打开default.md文件。
至于喜欢哪种格式，可以在config.toml中进行配置，默认使用的是yaml格式。通过执行hugo new 命令生成的markdown文件，头部默认会有这段渲染之后的Front Matter，一般我们会把draft属性去掉，draft草稿的意思，有这个属性的md文件不会渲染输出， 当然通过hugo –buildDrafts可以强制输出。


 - content
存放网页内容的目录，即我们编写的markdown文件都存放在此目录，此目录是Hugo的默认源目录，在E:/website/second-blog下执行命令 hugo new post/hugo_introduce.md之后， 则会在content目录下生成子目录post，post中有一个hugo_introduce.md文件。

 - data
data目录用来存放数据文件，一般是json文件，Hugo提供了相关命令可以从data目录下读取相关的文件数据，然后渲染到HTML页面中，将业务数据与模板分离。

  - layouts
存放自定义的模板文件，Hugo优先使用layouts目录下的模板，未发现再去themes目录下查找。

 - static
存放静态文件，比如css、js、img等文件目录，Hugo在渲染时，会直接将static目录下的文件直接复制到public目录下，不会做任何渲染。

 - themes
存放网站主题，可以下载多个主题，themes目录下的每个子目录代表了一个主题，可以通过在config.toml中通过参数theme指定主题，即theme目录下的子目录名字， 也可以在执行hugo命令渲染时通过增加flag参数–theme=xx指定。


### 添加代码高亮
ZIP工具包路径：https://highlightjs.org/
工具包目录
~~~ make
▸ styles/  
  CHANGES.MD  
  highlight.pack.js  
  LICENSE  
  README.md
  README.ru.md
~~~

styles目录下有很多不同类型的高亮方式，都是css文件。
接下来把`highlight.js`文件放到`site/themes/hyde/static/js/`目录下，没有目录自己建；
把需要的哪种风格的css文件放到`site/themes/hyde/static/css`目录下。
接着在`site/themes/hyde/layouts/partials/head.html`文件中插入如下内容：
~~~ javascript

<!-- Highlight.js and css -->
<script src="{{ .Site.BaseURL }}js/highlight.pack.js">
</script><link rel="stylesheet" href="{{ .Site.BaseURL }}css/monokai.css">
<script>hljs.initHighlightingOnLoad();</script>
~~~

因为个人比较喜欢monokai风格，所以用了monokai.css，相应的要把monokai.css复制到刚刚说的css目录下，你也可以根据自己的喜好选择其他的风格。
当你再次调试站点查看时如果内容中有代码可以发现代码已经被着色，更加清楚明了。


### 修改字体
进入 hugo-redlounge/static/css 文件夹，用编辑器打开 theme.css 文件

编辑如下代码中的 font-family 值，将需要的字体替换
~~~
body 
{
  background-color: #fff;
  font-family: "Hannotate SC", Meiryo, sans-serif;
  /*font-family: "Hiragino Kaku Gothic ProN", Meiryo, sans-serif;*/
  font-size: 14px;
  color: #666;
  line-height: 1.6em;
  letter-spacing: 0.5px;
}
~~~
>字体对照表->[中英文字体名称对照](https://www.zhangxinxu.com/study/201703/font-family-chinese-english.html)


### 添加国内社交账号
进入 angels-ladder/layouts/partials/文件夹，用编辑器打开header.html文件

编辑如下代码中的 ul 内的内容;

另外 Site.Params.xx 需要在 hugo 网站根目录中的 config.toml 文件中定义


~~~html
  <header role="banner">
    <div class="row gutters">
      <div id="site-title" class="col span_6">
        <h1><a href="{{ .Site.BaseURL }}">{{ .Site.Title }}</a></h1>
        {{ with .Site.Params.subtitle }}<h2>{{ . }}</h2>{{ end }}
      </div>
      <div id="social" class="col span_6">
        <ul>
          {{ with .Site.Params.weibo }}<li><a href="{{ . }}" target="_blank">Weibo</a></li>{{ end }}
          {{ with .Site.Params.wechat }}<li><a href="{{ . }}" target="_blank">Wechat</a></li>{{ end }}
          {{ with .Site.Params.jianshu }}<li><a href="{{ . }}" target="_blank">JianShu</a></li>{{ end }}
          {{ with .Site.Params.github }}<li><a href="{{ . }}" target="_blank">GitHub</a></li>{{ end }}
          {{ with .Site.Params.mail }}<li><a href="{{ . }}" target="_blank">Mail Me</a></li>{{ end }}
          {{ if .Site.Params.showsRSS }}<li><a href="{{ .Site.BaseURL }}index.xml" type="application/rss+xml" target="_blank">RSS</a></li>{{ end }}
        </ul>
      </div>
    </div>
  </header>
~~~

贴上我的 config.toml文件中 Params 定义
~~~ toml
	[params]
	subtitle = "信心是黑暗中的灯塔，任何时候都不能丢"
	weibo = "http://www.baidu.com"
	wechat = "htttp://www.baidu.com"
	jianshu = "http://www.jianshu.com/users/f5820a96231b/timeline"
	github = "http://www.baidu.com"
	mail = "pengwanring@live.com"
	right = "Written by Valencia，转载请注明出处，谢谢！"
~~~
	
	
### 添加文字分类
首先看了下发文章时默认分类的存储地方，查文档发现在：
/hugo-redlounge/archetypes/default.md
~~~
+++
Description = ""
Tags = ["规范", "分析", "技巧", "研究","原型"]
Categories = ["产品规范", "产品分析", "技巧分析", "技术研究","产品原型"]
+++
~~~


根据自己的文章需求，修改默认需要的 Tags 和 Categories，貌似在 执行hugo命令时，会根据此文件中的设置，在 public 下 tags 和 categories 文件夹下建立相关的文件夹；
进入 /hugo-redlounge/layouts/partials/ 文件夹，用编辑器打开 footer.html 文件

在文件中插入如下代码
~~~html
   <div>
    <br>
    </div>
    <div style="text-align: center;" id="social">
          {{ with .Site.Params.about }}<li><a href="{{ . }}" target="_blank">关于Ethan</a></li>{{ end }}
          {{ with .Site.Params.demo }}<li><a href="{{ . }}" target="_blank">产品原型</a></li>{{ end }}
          {{ with .Site.Params.guifan }}<li><a href="{{ . }}" target="_blank">产品规范</a></li>{{ end }}
          {{ with .Site.Params.fenxi }}<li><a href="{{ . }}" target="_blank">产品分析</a></li>{{ end }}
          {{ with .Site.Params.jiqiao }}<li><a href="{{ . }}" target="_blank">技巧分析</a></li>{{ end }}
          {{ with .Site.Params.yanjiu }}<li><a href="{{ . }}" target="_blank">技术研究</a></li>{{ end }}
    </div>
~~~
### 添加文章摘要
分别找到

/hugo-redlounge/layouts/_default/ 中的 list.html
/hugo-redlounge/layouts/ 中的 index.html
添加 {{.Summary}} 部分的代码

~~~html
<main id="index" role="main">
  <div class="article-header light-gray"><h2>最新文章</h2></div>
  {{ $paginator := .Paginate (where .Data.Pages "Type" "post") }}
  {{ range $paginator.Pages }}
  <div class="summary">
    <h2><a href="{{ .Permalink }}">{{ .Title }} {{ if .Draft }}:: DRAFT{{end}}</a></h2>
    <div class="meta">
      {{ .Date.Format "Jan 2, 2006 " }}  
      {{ range .Params.tags }}
        #<a href="/tags/{{ . | urlize }}">{{ . }}</a> 
        {{ end }}
    </div>
      {{ .Summary }}  
      {{ if .Truncated }}
        <div class="read-more-link">
        <a href="{{ .RelPermalink }}">Read More…</a>
        </div>
      {{ end }}
  </div>
  {{ end }}
</main>
~~~

### 添加符号链接
1. 需求：
   在welcome首页添加自定义联系链接，并且可以通过小图标点击
2. 寻找代码路径：
   - 在config.toml找到github, email等属性配置
   - 在homepage-header.html下找到相关代码
   - 在[图标网址]([图标网址](https://www.thinkcmf.com/font/font_awesome/icons.html))选择公网图标
3. 加入代码段
4. html出现在:footer.html page-header.html homepage-header.html(注意图标后缀不一致)
[新增本地ICON添加方式]:
1. 准备VM虚拟机和Windows共享文件夹。
   1. 关闭虚拟机 
   2. 右键VM的“虚拟机(M)”进入“设置”->“选项”->“共享文件夹”->点击“总是启用”->添加共享文件夹并命名
   3. 开启虚拟机，输入`mount |grep /dev/cdrom`是否有`/dev/cdrom on /mnt/cdrom type iso9660 (ro,nosuid,nodev)`信息
   4. 如果不存在，需要`mkdir /mnt/cdrom`创建装载点目录，`mount /dev/cdrom /mnt/cdrom`装载CD-ROM 驱动器
   5. 在VM15的“虚拟机(M)”选项中选择安装VMwareTolls，然后在指定文件夹把安装文件复制(否则安装时报错：只读文件夹)并解压。`cp VMwareTools-x.x.x-yyyy.tar.gz /tmp` `tar zxpf VMwareTools-x.x.x-yyyy.tar.gz`
   6. `cd vmware-tools-distrib` `sudo ./vmware-install.pl`
   7. 如果可以就卸载镜像CD-ROM `umount /dev/cdrom `
   8. 此时进入`la /mnt/hgfs` 就能看到被共享的文件夹。
2. 进入[iconfont](https://www.iconfont.cn/)登陆并且搜索对应的图标
3. 把图标加入购物车并且创建工程，下载安装包到windows共享给虚拟机
4. 解压安装包到`static/css`下
5. 在`head.html`引入css
   ```javascript
   <!-- 引入iconfont的css -->
   <link rel="stylesheet" href="assets/font/iconfont.css">
   ```
6. 使用时
   ```javascript
   <i class="iconfont  icon-zhihu"></i>
   ```


### 文章添加图片
在markDown添加图片时，文字和图片统一管理，
此时它们都在统一个文件夹之下，文件夹名称和title要统一，只支持小写。
~~~markdown
图片路径不需要是绝对路径
直接用文件名就好： ！[picture](picture-name.jpg)
~~~
  
### 手机分享图片
- 打开网页wx.qq.com登陆网页版微信
- 手机扫一扫登陆之后直接发送图片即可接收

## 版本控制

| Version | Action                   | Time       |
| ------- | ------------------------ | ---------- |
| 1.0     | Init                     | 2019-04-22T04:09:23-07:00|
| 1.01     | 添加符号链接                    | 2019-04-22T04:09:23-07:00|