---
date: "2019-05-02T02:52:20-07:00"
draft: true
title: "Web-Http_proxy"
#tags: []
#series: []
#categories: []
toc: true
---
# Web代理
Web 代理是一种存在于网络中间的实体，提供各式各样的功能。现代网络系统中，Web 代理无处不在。

HTTP 代理存在两种形式，分别简单介绍如下：
- 这种代理扮演的是「中间人」角色，对于连接到它的客户端来说，它是服务端；对于要连接的服务端来说，它是客户端。它就负责在两端之间来回传送 HTTP 报文。
- （通过 Web 代理服务器用隧道方式传输基于 TCP 的协议）描述的隧道代理。它通过 HTTP 协议正文部分（Body）完成通讯，以 HTTP 的方式实现任意基于 TCP 的应用层协议代理。这种代理使用 HTTP 的 CONNECT 方法建立连接，但 CONNECT 最开始并不是 RFC 2616 - HTTP/1.1 的一部分，直到 2014 年发布的 HTTP/1.1 修订版中，才增加了对 CONNECT 及隧道代理的描述，详见 RFC 7231 - HTTP/1.1: Semantics and Content。

本文描述的第一种代理，对应《HTTP 权威指南》一书中第六章「代理」；第二种代理，对应第八章「集成点：网关、隧道及中继」中的 8.5 小节「隧道」。

## 普通代理

HTTP 客户端向代理发送请求报文，代理服务器需要正确地处理请求和连接（例如正确处理 Connection: keep-alive），同时向服务器发送请求，并将收到的响应转发给客户端。

假如我通过代理访问 A 网站，对于 A 来说，它会把代理当做客户端，完全察觉不到真正客户端的存在，这实现了隐藏客户端 IP 的目的。当然代理也可以修改 HTTP 请求头部，通过 X-Forwarded-IP 这样的自定义头部告诉服务端真正的客户端 IP。但服务器无法验证这个自定义头部真的是由代理添加，还是客户端修改了请求头，所以从 HTTP 头部字段获取 IP 时，

给浏览器显式的指定代理，需要手动修改浏览器或操作系统相关设置，或者指定 PAC（Proxy Auto-Configuration，自动配置代理）文件自动设置，还有些浏览器支持 WPAD（Web Proxy Autodiscovery Protocol，Web 代理自动发现协议）。显式指定浏览器代理这种方式一般称之为正向代理，浏览器启用正向代理后，会对 HTTP 请求报文做一些修改，来规避老旧代理服务器的一些问题，

访问 A 网站时，实际上访问的是代理，代理收到请求报文后，再向真正提供服务的服务器发起请求，并将响应转发给浏览器。这种情况一般被称之为反向代理，它可以用来隐藏服务器 IP 及端口。一般使用反向代理后，需要通过修改 DNS 让域名解析到代理服务器 IP，这时浏览器无法察觉到真正服务器的存在，当然也就不需要修改配置了。反向代理是 Web 系统最为常见的一种部署方式，例如本博客就是使用 Nginx 的 proxy_pass 功能将浏览器请求转发到背后的 Node.js 服务。

使用我们这个代理服务后，HTTPS 网站完全无法访问，这是为什么呢？答案很简单，这个代理提供的是 HTTP 服务，根本没办法承载 HTTPS 服务。那么是否把这个代理改为 HTTPS 就可以了呢？显然也不可以，因为这种代理的本质是中间人，而 HTTPS 网站的证书认证机制是中间人劫持的克星。普通的 HTTPS 服务中，服务端不验证客户端的证书，中间人可以作为客户端与服务端成功完成 TLS 握手；但是中间人没有证书私钥，无论如何也无法伪造成服务端跟客户端建立 TLS 连接。当然如果你拥有证书私钥，代理证书对应的 HTTPS 网站当然就没问题了。

HTTP 抓包神器 Fiddler 的工作原理也是在本地开启 HTTP 代理服务，通过让浏览器流量走这个代理，从而实现显示和修改 HTTP 包的功能。如果要让 Fiddler 解密 HTTPS 包的内容，需要先将它自带的根证书导入到系统受信任的根证书列表中。一旦完成这一步，浏览器就会信任 Fiddler 后续的「伪造证书」，从而在浏览器和 Fiddler、Fiddler 和服务端之间都能成功建立 TLS 连接。而对于 Fiddler 这个节点来说，两端的 TLS 流量都是可以解密的。

如果我们不导入根证书，Fiddler 的 HTTP 代理还能代理 HTTPS 流量么？实践证明，不导入根证书，Fiddler 只是无法解密 HTTPS 流量，HTTPS 网站还是可以正常访问。这是如何做到的，这些 HTTPS 流量是否安全呢

## 隧道代理
HTTP 客户端通过 CONNECT 方法请求隧道代理创建一条到达任意目的服务器和端口的 TCP 连接，并对客户端和服务器之间的后继数据进行盲转发。

假如我通过代理访问 A 网站，浏览器首先通过 CONNECT 请求，让代理创建一条到 A 网站的 TCP 连接；一旦 TCP 连接建好，代理无脑转发后续流量即可。所以这种代理，理论上适用于任意基于 TCP 的应用层协议，HTTPS 网站使用的 TLS 协议当然也可以。这也是这种代理为什么被称为隧道的原因。对于 HTTPS 来说，客户端透过代理直接跟服务端进行 TLS 握手协商密钥，所以依然是安全的

浏览器建立到服务端 TCP 连接产生的 HTTP 往返，完全是明文，这也是为什么 CONNECT 请求只需要提供域名和端口：如果发送了完整 URL、Cookie 等信息，会被中间人一览无余，降低了 HTTPS 的安全性。HTTP 代理承载的 HTTPS 流量，应用数据要等到 TLS 握手成功之后通过 Application Data 协议传输，中间节点无法得知用于流量加密的 master-secret，无法解密数据。而 CONNECT 暴露的域名和端口，对于普通的 HTTPS 请求来说，中间人一样可以拿到（IP 和端口很容易拿到，请求的域名可以通过 DNS Query 或者 TLS Client Hello 中的 Server Name Indication 拿到），所以这种方式并没有增加不安全性。

## 总结
最后，将两种代理的实现代码合二为一，就可以得到全功能的 Proxy 程序了。
需要注意的是，大部分浏览器显式配置了代理之后，只会让 HTTPS 网站走隧道代理，这是因为建立隧道需要耗费一次往返，能不用就尽量不用。但这并不代表 HTTP 请求不能走隧道代理

普通代理可以用来承载 HTTP 流量；隧道代理可以用来承载任何 TCP 流量，包括 HTTP 和 HTTPS。

# 升级HTTPS
我们知道 TLS 有三大功能：内容加密、身份认证和数据完整性。其中内容加密依赖于密钥协商机制；数据完整性依赖于 MAC（Message authentication code）校验机制；而身份认证则依赖于证书认证机制。一般操作系统或浏览器会维护一个受信任根证书列表，包含在列表之中的证书，或者由列表中的证书签发的证书都会被客户端信任。

提供 HTTPS 服务的证书可以自己生成，然后手动加入到系统根证书列表中。但是对外提供服务的 HTTPS 网站，不可能要求每个用户都手动导入你的证书，所以更常见的做法是向 CA（Certificate Authority，证书颁发机构）申请。根据证书的不同级别，CA 会进行不同级别的验证，验证通过后 CA 会用他们的证书签发网站证书，这个过程通常是收费的（有免费的证书，最近免费的 Let's Encrypt 也很火，这里不多介绍）。由于 CA 使用的证书都是由广泛内置在各系统中的根证书签发，所以从 CA 获得的网站证书会被绝大部分客户端信任。

通过 CA 申请证书很简单，本文为了方便演示，采用自己签发证书的偷懒办法。现在广泛使用的证书是 x509.v3 格式。

# Nginx 配置
## 安全篇
### 隐藏不必要的信息

大家可以看一下我的博客请求响应头，有这么一行 server: nginx，说明我用的是 Nginx 服务器，但并没有具体的版本号。由于某些 Nginx 漏洞只存在于特定的版本，隐藏版本号可以提高安全性。这只需要在配置里加上这个就可以了：
>server_tokens   off;

如果想要更彻底隐藏所用 Web Server，可以修改 Nginx 源码，把 Server Name 改掉再编译，具体步骤可以自己搜索。需要提醒的是：如果你的网站支持 SPDY，只改动网上那些文章写到的地方还不够，跟 SPDY 有关的代码也要改。更简单的做法是改用 Tengine 这个 Nginx 的增强版，并指定 server_tag 为 off 或者任何想要的值就可以了。另外，既然想要彻底隐藏 Nginx，404、500 等各种出错页也需要自定义。

同样，一些 WEB 语言或框架默认输出的 x-powered-by 也会泄露网站信息，他们一般都提供了修改或移除的方法，可以自行查看手册。如果部署上用到了 Nginx 的反向代理，也可以通过 proxy_hide_header 指令隐藏它：
>proxy_hide_header        X-Powered-By;

### 禁用非必要的方法
由于我的博客只处理了 GET、POST 两种请求方法，而 HTTP/1 协议还规定了 TRACE 这样的方法用于网络诊断，这也可能会暴露一些信息。所以我针对 GET、POST 以及 HEAD 之外的请求，直接返回了 444 状态码（444 是 Nginx 定义的响应状态码，会立即断开连接，没有响应正文）。具体配置是这样的：
```c
if ($request_method !~ ^(GET|HEAD|POST)$ ) {
    return    444;
}

```

### 合理配置响应头
Nginx 通过 proxy_pass 把请求反向代理给 Node 绑定的 IP 和端口。在最终输出时，我给响应增加了以下头部：
```nginx
add_header  Strict-Transport-Security  "max-age=31536000";
add_header  X-Frame-Options  deny;
add_header  X-Content-Type-Options  nosniff;
add_header  Content-Security-Policy  "default-src 'self'; script-src 'self' 'unsafe-inline' 'unsafe-eval' https://a.disquscdn.com; img-src 'self' data: https://www.google-analytics.com; style-src 'self' 'unsafe-inline'; frame-src https://disqus.com";
```
Strict-Transport-Security（简称为 HSTS）可以告诉浏览器，在指定的 max-age 内，始终通过 HTTPS 访问我的博客。即使用户自己输入 HTTP 的地址，或者点击了 HTTP 链接，浏览器也会在本地替换为 HTTPS 再发送请求。另外由于我的证书不支持多域名，我没有加上 includeSubDomains

X-Frame-Options 用来指定此网页是否允许被 iframe 嵌套，deny 就是不允许任何嵌套发生。

X-Content-Type-Options 用来指定浏览器对未指定或错误指定 Content-Type 资源真正类型的猜测行为，nosniff 表示不允许任何猜测。

Content-Security-Policy（简称为 CSP）用来指定页面可以加载哪些资源，主要目的是减少 XSS 的发生。我允许了来自本站、disquscdn 的外链 JS，还允许内联 JS，以及在 JS 中使用 eval；允许来自本站和 google 统计的图片，以及内联图片（Data URI 形式）；允许本站外链 CSS 以及内联 CSS；允许 iframe 加载来自 disqus 的页面。对于其他未指定的资源，都会走默认规则 self，也就是只允许加载本站的。

X-XSS-Protection 这个响应头，也可以用来防范 XSS。不过由于有了 CSP，所以我没配置它。

### HTTPS 安全配置
```
ssl_certificate      /home/jerry/ssl/server.crt;
ssl_certificate_key  /home/jerry/ssl/server.key;
ssl_dhparam          /home/jerry/ssl/dhparams.pem;

ssl_ciphers          ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA:ECDHE-RSA-AES256-SHA:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA256:AES128-GCM-SHA256:AES256-GCM-SHA384:DES-CBC3-SHA;

ssl_prefer_server_ciphers  on;

ssl_protocols        TLSv1 TLSv1.1 TLSv1.2;
```
如何配置 [ssl_ciphers](https://cipherli.st/)需要注意的是，这个网站默认提供的加密方式安全性较高，一些低版本客户端并不支持，例如 IE9-、Android2.2- 和 Java6-。如果需要支持这些老旧的客户端，需要点一下网站上的「Yes, give me a ciphersuite that works with legacy / old software」链接。

我在 ssl_ciphers 最开始加上了 CHACHA20，这是因为我的 Nginx 支持了 CHACHA20_POLY1305 加密算法，这是由 Google 开发的新一代加密方式，它有两方面优势：更好的安全性和更好的性能（尤其是在移动和可穿戴设备上）

启用 CHACHA20_POLY1305 最简单的方法是在编译 Nginx 时，使用 LibreSSL 代替 OpenSSL。

ssl_dhparam 的配置，可以参考这篇文章：[Guide to Deploying Diffie-Hellman for TLS](https://weakdh.org/sysadmin.html)。

SSLv3 已被证实不安全，所以在 ssl_protocols 指令中，我并没有包含它。
将 ssl_prefer_server_ciphers 配置为 on，可以确保在 TLSv1 握手时，使用服务端的配置项，以增强安全性。

## 性能篇

### TCP优化
Nginx 关于 TCP 的优化基本都是修改系统内核提供的配置项:
```
http {
    sendfile           on;
    tcp_nopush         on;
    tcp_nodelay        on;

    keepalive_timeout  60;
    ... ...
}
```
sendfile 配置可以提高 Nginx 静态资源托管效率。sendfile 是一个系统调用，直接在内核空间完成文件发送，不需要先 read 再 write，没有上下文切换开销。

TCP_NOPUSH 是 FreeBSD 的一个 socket 选项，对应 Linux 的 TCP_CORK，Nginx 里统一用 tcp_nopush 来控制它，并且只有在启用了 sendfile 之后才生效。启用它之后，数据包会累计到一定大小之后才会发送，减小了额外开销，提高网络效率。

TCP_NODELAY 也是一个 socket 选项，启用后会禁用 Nagle 算法，尽快发送数据，某些情况下可以节约 200ms（Nagle 算法原理是：在发出去的数据还未被确认之前，新生成的小数据先存起来，凑满一个 MSS 或者等到收到确认后再发送）。Nginx 只会针对处于 keep-alive 状态的 TCP 连接才会启用 tcp_nodelay。

可以看到 TCP_NOPUSH 是要等数据包累积到一定大小才发送，TCP_NODELAY 是要尽快发送，二者相互矛盾。实际上，它们确实可以一起用，最终的效果是先填满包，再尽快发送。

配置最后一行用来指定服务端为每个 TCP 连接最多可以保持多长时间。Nginx 的默认值是 75 秒，有些浏览器最多只保持 60 秒，所以我统一设置为 60。

还有一个 TCP 优化策略叫 TCP Fast Open（TFO），这里先介绍下，配置在后面贴出。TFO 的作用是用来优化 TCP 握手过程。客户端第一次建立连接还是要走三次握手，所不同的是客户端在第一个 SYN 会设置一个 Fast Open 标识，服务端会生成 Fast Open Cookie 并放在 SYN-ACK 里，然后客户端就可以把这个 Cookie 存起来供之后的 SYN 用。

TCP 的 SO_REUSEPORT 选项了。这里也先简单介绍下，具体配置方法后面统一介绍。启用这个功能后，Nginx 会在指定的端口上监听多个 socket，每个 Worker 都能分到一个。

### 开启 Gzip
代码（JS、CSS 和 HTML）会做压缩，图片也会做压缩（PNGOUT、Pngcrush、JpegOptim、Gifsicle 等）。对于文本文件，在服务端发送响应之前进行 GZip 压缩也很重要，通常压缩后的文本大小会减小到原来的 1/4 - 1/3。

```
http {
    gzip               on;
    gzip_vary          on;

    gzip_comp_level    6;
    gzip_buffers       16 8k;

    gzip_min_length    1000;
    gzip_proxied       any;
    gzip_disable       "msie6";

    gzip_http_version  1.0;

    gzip_types         text/plain text/css application/json application/x-javascript text/xml application/xml application/xml+rss text/javascript application/javascript;
    ... ...
}

```
gzip_vary 用来输出 Vary 响应头，用来解决某些缓存服务的一个问题，

gzip_disable 指令接受一个正则表达式，当请求头中的 UserAgent 字段满足这个正则时，响应不会启用 GZip，这是为了解决在某些浏览器启用 GZip 带来的问题。特别地，指令值 msie6 等价于 MSIE [4-6]\.，但性能更好一些。

默认 Nginx 只会针对 HTTP/1.1 及以上的请求才会启用 GZip，因为部分早期的 HTTP/1.0 客户端在处理 GZip 时有 Bug。现在基本上可以忽略这种情况，于是可以指定 gzip_http_version 1.0 来针对 HTTP/1.0 及以上的请求开启 GZip。

### 开启缓存
优化代码逻辑的极限是移除所有逻辑；优化请求的极限是不发送任何请求。这两点通过缓存都可以实现。

### 服务端
我的博客更新并不频繁，评论部分也早就换成了 Disqus，所以完全可以将页面静态化，这样就省掉了所有代码逻辑和数据库开销。实现静态化有很多种方案，我直接用的是 Nginx 的 proxy_cache（注：本博客为了做更精细的静态化，已经将缓存逻辑挪到 Web 应用里实现

```
proxy_cache_path  /home/jerry/cache/nginx/proxy_cache_path levels=1:2 keys_zone=pnc:300m inactive=7d max_size=10g;
proxy_temp_path   /home/jerry/cache/nginx/proxy_temp_path;
proxy_cache_key   $host$uri$is_args$args;

server {
    location / {
        resolver                  127.0.0.1;  
        proxy_cache               pnc;
        proxy_cache_valid         200 304 2h;
        proxy_cache_lock          on;
        proxy_cache_lock_timeout  5s;
        proxy_cache_use_stale     updating error timeout invalid_header http_500 http_502;

        proxy_http_version        1.1;

        proxy_ignore_headers      Set-Cookie;
        ... ...
    }
    ... ...
}

```
首先，在配置最外层定义一个缓存目录，并指定名称（keys_zone）和其他属性，这样在配置 proxy_pass 时，就可以使用这个缓存了。这里我对状态值等于 200 和 304 的响应缓存了 2 小时。
默认情况下，如果响应头里有 Set-Cookie 字段，Nginx 并不会缓存这次响应，因为它认为这次响应的内容是因人而异的。我的博客中，这个 Set-Cookie 对于用户来说没有用，也不会影响输出内容，所以我通过配置 proxy_ignore_header 移除了它。

### 客户端
服务端在输出响应时，可以通过响应头输出一些与缓存有关的信息，从而达到少发或不发请求的目的。HTTP/1.1 的缓存机制稍微有点复杂，这里简单介绍下：
首先，服务端可以通过响应头里的 Last-Modified（最后修改时间） 或者 ETag（内容特征） 标记实体。浏览器会存下这些标记，并在下次请求时带上 If-Modified-Since: 上次 Last-Modified 的内容 或 If-None-Match: 上次 ETag 的内容，询问服务端资源是否过期。如果服务端发现并没有过期，直接返回一个状态码为 304、正文为空的响应，告知浏览器使用本地缓存；如果资源有更新，服务端返回状态码 200、新的 Last-Modified、Etag 和正文。这个过程被称之为 HTTP 的协商缓存，通常也叫做弱缓存。
可以看到协商缓存并不会节省连接数，但是在缓存生效时，会大幅减小传输内容（304 响应没有正文，一般只有几百字节）。另外为什么有两个响应头都可以用来实现协商缓存呢？这是因为一开始用的 Last-Modified 有两个问题：1）只能精确到秒，1 秒内的多次变化反映不出来；2）时间采用绝对值，如果服务端 / 客户端时间不对都可能导致缓存失效在轮询的负载均衡算法中，如果各机器读到的文件修改时间不一致，有缓存无故失效和缓存不更新的风险。HTTP/1.1 并没有规定 ETag 的生成规则，而一般实现者都是对资源内容做摘要，能解决前面两个问题。
另外一种缓存机制是服务端通过响应头告诉浏览器，在什么时间之前（Expires）或在多长时间之内（Cache-Control: Max-age=xxx），不要再请求服务器了。这个机制我们通常称之为 HTTP 的强缓存。
一旦资源命中强缓存规则后，再次访问完全没有 HTTP 请求（Chrome 开发者工具的 Network 面板依然会显示请求，但是会注明 from cache；Firefox 的 firebug 也类似，会注明 BFCache），这会大幅提升性能。所以我们一般会对 CSS、JS、图片等资源使用强缓存，而入口文件（HTML）一般使用协商缓存或不缓存，这样可以通过修改入口文件中对强缓存资源的引入 URL 来达到即时更新的目的。
这里也解释下为什么有了 Expires，还要有 Cache-Control。也有两个原因：1）Cache-Control 功能更强大，对缓存的控制能力更强；2）Cache-Control 采用的 max-age 是相对时间，不受服务端 / 客户端时间不对的影响。
另外关于浏览器的刷新（F5 / cmd + r）和强刷（Ctrl + F5 / shift + cmd +r）：普通刷新会使用协商缓存，忽略强缓存；强刷会忽略浏览器所有缓存（并且请求头会携带 Cache-Control:no-cache 和 Pragma:no-cache，用来通知所有中间节点忽略缓存）。只有从地址栏或收藏夹输入网址、点击链接等情况下，浏览器才会使用强缓存。
默认情况下，Nginx 对于静态资源都会输出 Last-Modified，而 ETag、Expires 和 Cache-Control 则需要自己配置：
```
location ~ ^/static/ {
    root      /home/jerry/www/blog/www;
    etag      on;
    expires   max;
}
```
expires 指令可以指定具体的 max-age，例如 10y 代表 10 年，如果指定为 max，最终输出的 Expires 会是 2037 年最后一天，Cache-Control 的 max-age 会是 10 年（准确说是 3650 天，315360000 秒）。


### 使用 SPDY（HTTP/2）
 Nginx 只支持 SPDY/3.1，这样配置就可以启用了（编译 Nginx 时需要加上 --with-http_spdy_module 和 --with-http_ssl_module）
```
server {
    listen             443 ssl spdy fastopen=3 reuseport;
    spdy_headers_comp  6;
    ... ...
}
```
那个 fastopen=3 用来开启前面介绍过的 TCP Fast Open 功能。3 代表最多只能有 3 个未经三次握手的 TCP 链接在排队。超过这个限制，服务端会退化到采用普通的 TCP 握手流程。这是为了减少资源耗尽攻击：TFO 可以在第一次 SYN 的时候发送 HTTP 请求，而服务端会校验 Fast Open Cookie（FOC），如果通过就开始处理请求。如果不加限制，恶意客户端可以利用合法的 FOC 发送大量请求耗光服务端资源。
reuseport 就是用来启用前面介绍过的 TCP SO_REUSEPORT 选项的配置。

### HTTPS 优化
```
server {
    ssl_session_cache        shared:SSL:10m;
    ssl_session_timeout      60m;

    ssl_session_tickets      on;

    ssl_stapling             on;
    ssl_stapling_verify      on;
    ssl_trusted_certificate  /xxx/full_chain.crt;

    resolver                 8.8.4.4 8.8.8.8  valid=300s;
    resolver_timeout         10s;
    ... ...
}

```
我的这部分配置就两部分内容：TLS 会话恢复和 OCSP stapling。

TLS 会话恢复的目的是为了简化 TLS 握手，有两种方案：Session Cache 和 Session Ticket。他们都是将之前握手的 Session 存起来供后续连接使用，所不同是 Cache 存在服务端，占用服务端资源；Ticket 存在客户端，不占用服务端资源。另外目前主流浏览器都支持 Session Cache，而 Session Ticket 的支持度一般。

ssl_stapling 开始的几行用来配置 OCSP stapling 策略。浏览器可能会在建立 TLS 连接时在线验证证书有效性，从而阻塞 TLS 握手，拖慢整体速度。OCSP stapling 是一种优化措施，服务端通过它可以在证书链中封装证书颁发机构的 OCSP（Online Certificate Status Protocol）响应，从而让浏览器跳过在线查询。服务端获取 OCSP 一方面更快（因为服务端一般有更好的网络环境），另一方面可以更好地缓存。

在给 Nginx 指定证书时，需要选择合适的证书链。因为浏览器在验证证书信任链时，会从站点证书开始，递归验证父证书，直至信任的根证书。这里涉及到两个问题：1）服务器证书是在握手期间发送的，由于 TCP 初始拥塞窗口的存在，如果证书太长很可能会产生额外的往返开销；2）如果服务端证书没包含中间证书，大部分浏览器可以正常工作，但会暂停验证并根据子证书指定的父证书 URL 自己获取中间证书。这个过程会产生额外的 DNS 解析、建立 TCP 连接等开销。配置服务端证书链的最佳实践是包含站点证书和中间证书两部分。有的证书提供商签出来的证书级别比较多，这会导致证书链变长，选择的时候需要特别注意。

这些策略设置好之后，可以通过 Qualys SSL Server Test 这个工具来验证是否生效



## 版本控制

| Version | Action                   | Time       |
| ------- | ------------------------ | ---------- |
| 1.0     | Init                     | 2019-05-02T02:52:20-07:00|
