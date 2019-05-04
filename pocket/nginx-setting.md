---
date: "2019-04-25T10:13:34-07:00"
draft: true
title: "Nginx-Setting"
tags: [nginx]
series: []
categories: []
toc: true
---
# 手动安装
```shell
# 运行之前需要切换到 root 用户
serviceDir=/data/home/username

# 安装配置依赖，这里直接用 apt-get
echo "[INFO] Installing make g++"
cd $serviceDir
apt-get install make
apt-get install g++
echo "[INFO] Done(1/7)"

# 安装 openssl，其中 openssl-1.0.2 是长期支持版本，所以我采用这个版本
# 更多信息请访问 https://www.openssl.org/source/
echo "[INFO] Installing openssl" 
cd $serviceDir
wget https://www.openssl.org/source/openssl-1.0.2h.tar.gz
tar -xzvf openssl-1.0.2h.tar.gz
cd $serviceDir/openssl-1.0.2h
./config
make
make install
ldconfig
echo "[INFO] Done(2/7)"

# 安装 Pcre，为了保证兼容我们这里使用较老的版本
# 源用的是 stanford 的（因为 pcre.org 我这里打不开）
# 源：http://ftp.cs.stanford.edu/pub/exim/pcre/
echo "[INFO] Installing pcre"
cd $serviceDir
wget http://ftp.cs.stanford.edu/pub/exim/pcre/pcre-8.37.tar.gz
tar -xzvf pcre-8.37.tar.gz
cd $serviceDir/pcre-8.37
./configure
make
make install
ldconfig
echo "[INFO] Done(3/7)"

# 安装 zlib，用的就是最新的 1.2.8
# 源 http://zlib.net/zlib-1.2.8.tar.gz
echo "[INFO] Installing zlib"
cd $serviceDir
wget http://zlib.net/zlib-1.2.8.tar.gz
tar -xzvf zlib-1.2.8.tar.gz
cd $serviceDir/zlib-1.2.8
./configure
make
make install
ldconfig
echo "[INFO] Done(4/7)"

# 安装 Nginx
echo "[INFO] Installing Nginx"
cd $serviceDir
wget https://nginx.org/download/nginx-1.10.1.tar.gz
tar -xzvf nginx-1.10.1.tar.gz
cd $serviceDir/nginx-1.10.1
./configure --prefix=$serviceDir/nginx-server --with-openssl=$serviceDir/openssl-1.0.2h --with-http_ssl_module --with-http_stub_status_module --with-stream
make
make install
cd $serviceDir/nginx-server/conf
rm -rf nginx.conf
# 这里用的是已经配置好的配置文件
wget http://xssz.oss-cn-shenzhen.aliyuncs.com/server_software/nginx.conf
ln -s $serviceDir/nginx-server/sbin/nginx /usr/local/bin/nginx
mkdir $serviceDir/nginx-server/run
mkdir $serviceDir/nginx-config
ln -s $serviceDir/nginx-server/sbin/nginx /usr/local/bin/nginx
nginx -c $serviceDir/nginx-server/conf/nginx.conf
echo "[INFO] Done(5/7)"

# 安装守护
echo "[INFO] Config daemon"
echo $serviceDir'/nginx-server/sbin/nginx -c '$serviceDir'/nginx-server/conf/nginx.conf' > /etc/rc.local
echo 'exit 0' >> /etc/rc.local
echo "[INFO] Done(6/7)"

# 清理工作，把所有的安装包保存到 software 文件夹中
echo "[INFO] Clean up all the mess"
cd $serviceDir
mkdir $serviceDir/software
mv *.gz $serviceDir/software
echo "[INFO] Done(7/7)"

echo "All Done. You can now continue your work."


```

# 配置
因为是手动安装的 nginx 的，默认的 nginx 配置在 `~/nginx-server/conf/nginx.conf` 中

```nignx
user www-data;
worker_processes 1;
worker_rlimit_nofile 262140;
worker_cpu_affinity 1;
error_log logs/error.log;
pid run/nginx.pid;

events
{
    use epoll;
    worker_connections 65535;
}

http
{
    include mime.types;
    default_type application/octet-stream;
    
    sendfile on;
    #aio on;
    directio 512;
    output_buffers 1 128k;
    log_not_found off;
    keepalive_timeout 65;
    server_tokens off;
    
    gzip on;
    gzip_comp_level 6;
    gzip_min_length 1k;
    gzip_buffers 4 8k;
    gzip_disable "MSIE [1-6]\.(?!.*SV1)";
    gzip_types text/plain application/x-javascript text/css application/xml text/javascript application/javascript application/json;
    
    log_format main '$remote_addr - $remote_user [$time_local] "$request" $status $body_bytes_sent "$http_referer" "$http_user_agent" "$request_time" "$upstream_response_time"';
    access_log logs/${server_name}.access.log main;
    fastcgi_intercept_errors on;
    error_page 500 502 503 504 /50x.html;
    
    server_names_hash_max_size 4096;
    
    server
    {
        listen 80 default;
        server_name _;
        access_log off;
        
        location /
        {
            return 403;
        }
    }
    include /data/home/user/nginx-config/*.http;
}

stream{
    include /data/home/user/nginx-config/*.tcp;
}
```
前面的都不重要，重要的是最后两句 include 语句，这里我们就把需要自己配置的部分抽取了出来，放到了文件夹 nginx-config 中。

Nginx配置文件由以下几个部分构成：
全局块：配置影响nginx全局的指令。一般有运行nginx服务器的用户组，nginx进程pid存放路径，日志存放路径，配置文件引入，允许生成worker process数等。
events块：配置影响nginx服务器或与用户的网络连接。有每个进程的最大连接数，选取哪种事件驱动模型处理连接请求，是否允许同时接受多个网路连接，开启多个网络连接序列化等。
http块：可以嵌套多个server，配置代理，缓存，日志定义等绝大多数功能和第三方模块的配置。如文件引入，mime-type定义，日志自定义，是否使用sendfile传输文件，连接超时时间，单连接请求数等。
server块：配置虚拟主机的相关参数，一个http中可以有多个server。
location块：配置请求的路由，以及各种页面的处理情况。


接着，只要我们在 nginx-config 文件夹中，针对不同的域名和应用进行配置即可，比方说我的 Go 应用跑在本机的 12345 端口上，那么可以这么配置：

```
upstream http_pool{
    server 127.0.0.1:12345 weight=1 max_fails=3 fail_timeout=30s;
}

server{
    server_name test.xxx.com;
    listen 80;
    ssl off;
    
    gzip on;
    gzip_min_length 1k;
    gzip_buffers 16 64k;
    gzip_comp_level 9;
    gzip_types text/plain text/css application/json application/x-javascript application/xml application/xml+rss text/javascript application/atom+xml;
    gzip_vary on;
    
    location /
    {
        proxy_next_upstream http_404 http_502 http_504 http_500 error timeout invalid_header;
        proxy_pass http://http_pool;
        proxy_set_header X-Forwarded-For $remote_addr;
        proxy_set_header Host $http_host;
        client_max_body_size 5000k;
    }
}

```




# 使用

Nginx 的使用我比较常用的其实在 `nginx -h`中已经有介绍，一般来说我就用一个命令 `nginx -s [stop|quit|reopen|reload]`，其实也就是 `sudo nginx -s reload`

另外有一个需要注意的地方就是，在 nginx 中配置的 80(http) 和 8080(tcp) 端口不能被占用，不然会一直冲突。另外需要注意配置路径的时候不要弄错了。


# 总结

# 参考链接

## 版本控制

| Version | Action                   | Time       |
| ------- | ------------------------ | ---------- |
| 1.0     | Init                     | 2019-04-25T10:13:34-07:00|
