---
date: "2019-05-02T01:24:48-07:00"
draft: true
title: "Server-Code"
#tags: []
#series: []
#categories: []
toc: true
---
## 循环服务器
UDP
```cpp
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>

int main(int argc, char *argv[]) {
	unsigned short port = 8080;	// 本地端口

	int sockfd;
	sockfd = socket(AF_INET, SOCK_DGRAM, 0); // 创建udp套接字
	if(sockfd < 0) {
		perror("socket");
		exit(-1);
	}

	// 初始化本地网络信息
	struct sockaddr_in my_addr;
	bzero(&my_addr, sizeof(my_addr));	// 清空
	my_addr.sin_family = AF_INET;		// IPv4
	my_addr.sin_port   = htons(port);	// 端口
	my_addr.sin_addr.s_addr = htonl(INADDR_ANY); // ip
	printf("Binding server to port %d\n", port);

	
	// 绑定
	int err_log;
	err_log = bind(sockfd, (struct sockaddr*)&my_addr, sizeof(my_addr));

	if(err_log != 0) {
		perror("bind");
		close(sockfd);		
		exit(-1);
	}

	printf("receive data...\n");
	while(1) {
		int recv_len;
		char recv_buf[512] = {0};
		struct sockaddr_in client_addr;
		char cli_ip[INET_ADDRSTRLEN] = "";//INET_ADDRSTRLEN=16
		socklen_t cliaddr_len = sizeof(client_addr);

		// 接收客户端数据
		recv_len = recvfrom(sockfd, recv_buf, sizeof(recv_buf), 0, (struct sockaddr*)&client_addr, &cliaddr_len);

		// 处理数据，这里只是把接收过来的数据打印
		inet_ntop(AF_INET, &client_addr.sin_addr, cli_ip, INET_ADDRSTRLEN);

		printf("\nip:%s ,port:%d\n",cli_ip, ntohs(client_addr.sin_port)); // 客户端的ip
		printf("data(%d):%s\n",recv_len,recv_buf);	// 客户端的数据

		// 反馈结果，这里把接收直接到客户端的数据回复过去
		sendto(sockfd, recv_buf, recv_len, 0, (struct sockaddr*)&client_addr, cliaddr_len);
	}

	close(sockfd);
	return 0;
}

```
TCP
```cpp
#include <stdio.h>
#include <stdlib.h>
#include <string.h>						
#include <unistd.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>

int main(int argc, char *argv[]) {
	unsigned short port = 8080;		// 本地端口	

	// 创建tcp套接字
	int sockfd = socket(AF_INET, SOCK_STREAM, 0);   
	if(sockfd < 0) {
		perror("socket");
		exit(-1);
	}

	// 配置本地网络信息
	struct sockaddr_in my_addr;
	bzero(&my_addr, sizeof(my_addr));	  // 清空   

	my_addr.sin_family = AF_INET;		  // IPv4
	my_addr.sin_port   = htons(port);	  // 端口
	my_addr.sin_addr.s_addr = htonl(INADDR_ANY); // ip

	// 绑定
	int err_log = bind(sockfd, (struct sockaddr*)&my_addr, sizeof(my_addr));
	if( err_log != 0){
		perror("binding");
		close(sockfd);		
		exit(-1);
	}

	// 监听，套接字变被动
	err_log = listen(sockfd, 10); 
	if(err_log != 0) {
		perror("listen");
		close(sockfd);		
		exit(-1);
	}	

	printf("listen client @port=%d...\n",port);

	while(1) {	
		struct sockaddr_in client_addr;		   
		char cli_ip[INET_ADDRSTRLEN] = "";	   
		socklen_t cliaddr_len = sizeof(client_addr);    

		// 取出客户端已完成的连接
		int connfd;
		connfd = accept(sockfd, (struct sockaddr*)&client_addr, &cliaddr_len);       
		if(connfd < 0) {
			perror("accept");
			continue;
		}

		// 打印客户端的ip和端口
		inet_ntop(AF_INET, &client_addr.sin_addr, cli_ip, INET_ADDRSTRLEN);
		printf("----------------------------------------------\n");
		printf("client ip=%s,port=%d\n", cli_ip,ntohs(client_addr.sin_port));

		// 接收数据
		char recv_buf[512] = {0};
		int len =  recv(connfd, recv_buf, sizeof(recv_buf), 0);

		// 处理数据，这里只是打印接收到的内容
		printf("\nrecv data:\n");
		printf("%s\n",recv_buf);

		// 反馈结果
		send(connfd, recv_buf, len, 0);

		close(connfd);     //关闭已连接套接字
		printf("client closed!\n");
	}

	close(sockfd);         //关闭监听套接字

	return 0;
}

```

## 并发服务器
TCP
```cpp
#include <stdio.h>
#include <stdlib.h>
#include <string.h>						
#include <unistd.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>	

int main(int argc, char *argv[]) {
	unsigned short port = 8080;		// 本地端口	

	// 创建tcp套接字
	int sockfd = socket(AF_INET, SOCK_STREAM, 0);   
	if(sockfd < 0) {
		perror("socket");
		exit(-1);
	}

	// 配置本地网络信息
	struct sockaddr_in my_addr;
	bzero(&my_addr, sizeof(my_addr));	  // 清空   
	my_addr.sin_family = AF_INET;		  // IPv4
	my_addr.sin_port   = htons(port);	  // 端口
	my_addr.sin_addr.s_addr = htonl(INADDR_ANY); // ip

	// 绑定
	int err_log = bind(sockfd, (struct sockaddr*)&my_addr, sizeof(my_addr));
	if( err_log != 0) {
		perror("binding");
		close(sockfd);		
		exit(-1);
	}

	// 监听，套接字变被动
	err_log = listen(sockfd, 10); 
	if(err_log != 0) {
		perror("listen");
		close(sockfd);		
		exit(-1);
	}

	while(1) {          //主进程 循环等待客户端的连接
		char cli_ip[INET_ADDRSTRLEN] = {0};
		struct sockaddr_in client_addr;
		socklen_t cliaddr_len = sizeof(client_addr);

		// 取出客户端已完成的连接
		int connfd = accept(sockfd, (struct sockaddr*)&client_addr, &cliaddr_len);
		if(connfd < 0) {
			perror("accept");
			close(sockfd);
			exit(-1);
		}

		pid_t pid = fork();
		if(pid < 0){
			perror("fork");
			_exit(-1);
		}else if(0 == pid){ //子进程 接收客户端的信息，并发还给客户端
			/*
              关闭不需要的套接字可节省系统资源，
			  同时可避免父子进程共享这些套接字
			  可能带来的不可预计的后果
			*/
			close(sockfd);   // 关闭监听套接字，这个套接字是从父进程继承过来

			char recv_buf[1024] = {0};
			int recv_len = 0;

			// 打印客户端的 ip 和端口
			memset(cli_ip, 0, sizeof(cli_ip)); // 清空
			inet_ntop(AF_INET, &client_addr.sin_addr, cli_ip, INET_ADDRSTRLEN);
			printf("----------------------------------------------\n");
			printf("client ip=%s,port=%d\n", cli_ip,ntohs(client_addr.sin_port));

			// 接收数据
			while( (recv_len = recv(connfd, recv_buf, sizeof(recv_buf), 0)) > 0 ) {
				printf("recv_buf: %s\n", recv_buf); // 打印数据
				send(connfd, recv_buf, recv_len, 0); // 给客户端回数据
			}

			printf("client closed!\n");

			close(connfd);    //关闭已连接套接字
			exit(0);

		}else if(pid > 0){	// 父进程

			close(connfd);    //关闭已连接套接字
		}

	}

	close(sockfd);
    return 0;
}


```
## 多线程服务器
```cpp
#include <stdio.h>
#include <stdlib.h>
#include <string.h>						
#include <unistd.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>					
#include <pthread.h>

pthread_mutex_t mutex;	// 定义互斥锁，全局变量
/************************************************************************
函数名称：	void *client_process(void *arg)
函数功能：	线程函数,处理客户信息
函数参数：	已连接套接字
函数返回：	无
************************************************************************/

void *client_process(void *arg) {
	int recv_len = 0;
	char recv_buf[1024] = "";	// 接收缓冲区
	int connfd = *(int *)arg; // 传过来的已连接套接字

	// 解锁，pthread_mutex_lock()唤醒，不阻塞
	pthread_mutex_unlock(&mutex); 

	// 接收数据
	while((recv_len = recv(connfd, recv_buf, sizeof(recv_buf), 0)) > 0) {
		printf("recv_buf: %s\n", recv_buf); // 打印数据
		send(connfd, recv_buf, recv_len, 0); // 给客户端回数据
	}

	printf("client closed!\n");

	close(connfd);	//关闭已连接套接字
	return 	NULL;
}

//===============================================================
// 语法格式：	void main(void)
// 实现功能：	主函数，建立一个TCP并发服务器
// 入口参数：	无
// 出口参数：	无
//===============================================================

int main(int argc, char *argv[])
{
	int sockfd = 0;				// 套接字
	int connfd = 0;
	int err_log = 0;

	struct sockaddr_in my_addr;	// 服务器地址结构体
	unsigned short port = 8080; // 监听端口
	pthread_t thread_id;

	pthread_mutex_init(&mutex, NULL); // 初始化互斥锁，互斥锁默认是打开的
	printf("TCP Server Started at port %d!\n", port);

	sockfd = socket(AF_INET, SOCK_STREAM, 0);   // 创建TCP套接字
	if(sockfd < 0) {
		perror("socket error");
		exit(-1);
	}

	bzero(&my_addr, sizeof(my_addr));	   // 初始化服务器地址
	my_addr.sin_family = AF_INET;
	my_addr.sin_port   = htons(port);
	my_addr.sin_addr.s_addr = htonl(INADDR_ANY);

	printf("Binding server to port %d\n", port);

	// 绑定
	err_log = bind(sockfd, (struct sockaddr*)&my_addr, sizeof(my_addr));
	if(err_log != 0) {
		perror("bind");
		close(sockfd);		
		exit(-1);
	}

	// 监听，套接字变被动
	err_log = listen(sockfd, 10);
	if( err_log != 0) {
		perror("listen");
		close(sockfd);		
		exit(-1);
	}

	printf("Waiting client...\n");

	while(1) {
		char cli_ip[INET_ADDRSTRLEN] = "";	   // 用于保存客户端IP地址
		struct sockaddr_in client_addr;		   // 用于保存客户端地址
		socklen_t cliaddr_len = sizeof(client_addr);   // 必须初始化!!!

		// 上锁，在没有解锁之前，pthread_mutex_lock()会阻塞
		pthread_mutex_lock(&mutex);	

		//获得一个已经建立的连接	
		connfd = accept(sockfd, (struct sockaddr*)&client_addr, &cliaddr_len);   							
		if(connfd < 0) {
			perror("accept this time");
			continue;
		}

		// 打印客户端的 ip 和端口
		inet_ntop(AF_INET, &client_addr.sin_addr, cli_ip, INET_ADDRSTRLEN);

		printf("----------------------------------------------\n");
		printf("client ip=%s,port=%d\n", cli_ip,ntohs(client_addr.sin_port));

		if(connfd > 0) {
			//给回调函数传的参数，&connfd，地址传递
			pthread_create(&thread_id, NULL, (void *)client_process, (void *)&connfd);  //创建线程
			pthread_detach(thread_id); // 线程分离，结束时自动回收资源
		}
	}

	close(sockfd);
	return 0;
}


```

## I/O复用服务器
epoll
```cpp
#include <sys/epoll.h>
#include <stdio.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>
#include <stdlib.h>

int main(int argc, char *argv[])
{
	int ret;
	int fd;

	ret = mkfifo("test_fifo", 0666); // 创建有名管道
	if(ret != 0){
		perror("mkfifo：");

	}
    
	fd = open("test_fifo", O_RDWR); // 读写方式打开管道
	if(fd < 0){
		perror("open fifo");
		return -1;
	}

	ret = 0;
	struct epoll_event event;	// 告诉内核要监听什么事件
	struct epoll_event wait_event;

	int epfd = epoll_create(10); // 创建一个 epoll 的句柄，参数要大于 0， 没有太大意义
	if( -1 == epfd ){
		perror ("epoll_create");
		return -1;
    }

	event.data.fd = 0; 	   // 标准输入
	event.events = EPOLLIN; // 表示对应的文件描述符可以读

	// 事件注册函数，将标准输入描述符 0 加入监听事件
	ret = epoll_ctl(epfd, EPOLL_CTL_ADD, 0, &event);
	if(-1 == ret){
		perror("epoll_ctl");
		return -1;
    }

	event.data.fd = fd; 	// 有名管道
	event.events = EPOLLIN; // 表示对应的文件描述符可以读

	// 事件注册函数，将有名管道描述符 fd 加入监听事件
	ret = epoll_ctl(epfd, EPOLL_CTL_ADD, fd, &event);
	if(-1 == ret){
		perror("epoll_ctl");
		return -1;

    }
	ret = 0;

	while(1){
		// 监视并等待多个文件（标准输入，有名管道）描述符的属性变化（是否可读）
		// 没有属性变化，这个函数会阻塞，直到有变化才往下执行，这里没有设置超时
		ret = epoll_wait(epfd, &wait_event, 2, -1);
		//ret = epoll_wait(epfd, &wait_event, 2, 1000);

		if(ret == -1) { // 出错
			close(epfd);
			perror("epoll");
		}else if(ret > 0){ // 准备就绪的文件描述符
			char buf[100] = {0};
			if((0 == wait_event.data.fd)
			&& (EPOLLIN == wait_event.events & EPOLLIN)){ // 标准输入
				read(0, buf, sizeof(buf));
				printf("stdin buf = %s\n", buf);
			}else if( (fd == wait_event.data.fd ) 
			&& ( EPOLLIN == wait_event.events & EPOLLIN )){ // 有名管道
				read(fd, buf, sizeof(buf));
				printf("fifo buf = %s\n", buf);
			}
		} else if (0 == ret) { // 超时
			printf("time out\n");
		}
	}

	close(epfd);
	return 0;
}


```

## HTTP 服务器
```cpp
/*
     （1） 服务器启动，在指定端口或随机选取端口绑定 httpd 服务。

     （2）收到一个 HTTP 请求时（其实就是 listen 的端口 accpet 的时候），派生一个线程运行 accept_request 函数。

     （3）取出 HTTP 请求中的 method (GET 或 POST) 和 url,。对于 GET 方法，如果有携带参数，则 query_string 指针指向 url 中 ？ 后面的 GET 参数。

     （4） 格式化 url 到 path 数组，表示浏览器请求的服务器文件路径，在 tinyhttpd 中服务器文件是在 htdocs 文件夹下。当 url 以 / 结尾，或 url 是个目录，则默认在 path 中加上 index.html，表示访问主页。

     （5）如果文件路径合法，对于无参数的 GET 请求，直接输出服务器文件到浏览器，即用 HTTP 格式写到套接字上，跳到（10）。其他情况（带参数 GET，POST 方式，url 为可执行文件），则调用 excute_cgi 函数执行 cgi 脚本。

    （6）读取整个 HTTP 请求并丢弃，如果是 POST 则找出 Content-Length. 把 HTTP 200  状态码写到套接字。

    （7） 建立两个管道，cgi_input 和 cgi_output, 并 fork 一个进程。

    （8） 在子进程中，把 STDOUT 重定向到 cgi_outputt 的写入端，把 STDIN 重定向到 cgi_input 的读取端，关闭 cgi_input 的写入端 和 cgi_output 的读取端，设置 request_method 的环境变量，GET 的话设置 query_string 的环境变量，POST 的话设置 content_length 的环境变量，这些环境变量都是为了给 cgi 脚本调用，接着用 execl 运行 cgi 程序。

    （9） 在父进程中，关闭 cgi_input 的读取端 和 cgi_output 的写入端，如果 POST 的话，把 POST 数据写入 cgi_input，已被重定向到 STDIN，读取 cgi_output 的管道输出到客户端，该管道输入是 STDOUT。接着关闭所有管道，等待子进程结束。这一部分比较乱，见下图说明
    （10） 关闭与浏览器的连接，完成了一次 HTTP 请求与回应，因为 HTTP 是无连接的。
*/
#include <stdio.h>
#include <sys/socket.h>
#include <sys/types.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <unistd.h>
#include <ctype.h>
#include <strings.h>
#include <string.h>
#include <sys/stat.h>
#include <pthread.h>
#include <sys/wait.h>
#include <stdlib.h>

#define ISspace(x) isspace((int)(x))
#define SERVER_STRING "Server: jdbhttpd/0.1.0\r\n"

void accept_request(int);
void bad_request(int);
void cat(int, FILE *);
void cannot_execute(int);
void error_die(const char *);
void execute_cgi(int, const char *, const char *, const char *);
int  get_line(int, char *, int);
void headers(int, const char *);
void not_found(int);
void serve_file(int, const char *);
int  startup(u_short *);
void unimplemented(int);

/**********************************************************************/
/* A request has caused a call to accept() on the server port to
 * return.  Process the request appropriately.
 * Parameters: the socket connected to the client */
/**********************************************************************/

void accept_request(int client)
{
    char buf[1024];
    int numchars;
    char method[255];
    char url[255];
    char path[512];
    size_t i, j;
    struct stat st;

    int cgi = 0;      /* becomes true if server decides this is a CGI program */
    char *query_string = NULL;

    /*得到请求的第一行*/
    numchars = get_line(client, buf, sizeof(buf));
    i = 0; j = 0;

    /*把客户端的请求方法存到 method 数组*/
    while (!ISspace(buf[j]) && (i < sizeof(method) - 1)) {
        method[i] = buf[j];
        i++; j++;
    }

    method[i] = '\0';

    /*如果既不是 GET 又不是 POST 则无法处理 */
    if (strcasecmp(method, "GET") && strcasecmp(method, "POST")) {
        unimplemented(client);
        return;
    }

    /* POST 的时候开启 cgi */
    if (strcasecmp(method, "POST") == 0)
        cgi = 1;

    /*读取 url 地址*/
    i = 0;

    while (ISspace(buf[j]) && (j < sizeof(buf)))
        j++;

    while (!ISspace(buf[j]) && (i < sizeof(url) - 1) && (j < sizeof(buf))) {

        /*存下 url */
        url[i] = buf[j];
        i++; j++;
    }
    url[i] = '\0';

    /*处理 GET 方法*/
    if (strcasecmp(method, "GET") == 0) {
        /* 待处理请求为 url */
        query_string = url;
        while ((*query_string != '?') && (*query_string != '\0'))
            query_string++;

        /* GET 方法特点，? 后面为参数*/
        if (*query_string == '?') {
            /*开启 cgi */
            cgi = 1;
            *query_string = '\0';
            query_string++;
        }
    }

    /*格式化 url 到 path 数组，html 文件都在 htdocs 中*/
    sprintf(path, "htdocs%s", url);

    /*默认情况为 index.html */
    if (path[strlen(path) - 1] == '/')
        strcat(path, "index.html");

    /*根据路径找到对应文件 */
    if (stat(path, &st) == -1) {
        /*把所有 headers 的信息都丢弃*/

        while ((numchars > 0) && strcmp("\n", buf))  /* read & discard headers */
            numchars = get_line(client, buf, sizeof(buf));

        /*回应客户端找不到*/
        not_found(client);
    } else {
        /*如果是个目录，则默认使用该目录下 index.html 文件*/
        if ((st.st_mode & S_IFMT) == S_IFDIR)
            strcat(path, "/index.html");
        if ((st.st_mode & S_IXUSR) || (st.st_mode & S_IXGRP) || (st.st_mode & S_IXOTH)    )
            cgi = 1;

        /*不是 cgi,直接把服务器文件返回，否则执行 cgi */
        if (!cgi)
            serve_file(client, path);
        else
            execute_cgi(client, path, method, query_string);
    }

    /*断开与客户端的连接（HTTP 特点：无连接）*/
    close(client);
}


/**********************************************************************/
/* Inform the client that a request it has made has a problem.
 * Parameters: client socket */
/**********************************************************************/
void bad_request(int client)
{
    char buf[1024];

    /*回应客户端错误的 HTTP 请求 */
    sprintf(buf, "HTTP/1.0 400 BAD REQUEST\r\n");
    send(client, buf, sizeof(buf), 0);

    sprintf(buf, "Content-type: text/html\r\n");
    send(client, buf, sizeof(buf), 0);

    sprintf(buf, "\r\n");
    send(client, buf, sizeof(buf), 0);

    sprintf(buf, "<P>Your browser sent a bad request, ");
    send(client, buf, sizeof(buf), 0);

    sprintf(buf, "such as a POST without a Content-Length.\r\n");
    send(client, buf, sizeof(buf), 0);
}

/**********************************************************************/
/* Put the entire contents of a file out on a socket.  This function
 * is named after the UNIX "cat" command, because it might have been
 * easier just to do something like pipe, fork, and exec("cat").
 * Parameters: the client socket descriptor
 *             FILE pointer for the file to cat */
/**********************************************************************/
void cat(int client, FILE *resource)
{
    char buf[1024];

    /*读取文件中的所有数据写到 socket */
    fgets(buf, sizeof(buf), resource);
    while (!feof(resource)) {
        send(client, buf, strlen(buf), 0);
        fgets(buf, sizeof(buf), resource);
    }
}



/**********************************************************************/
/* Inform the client that a CGI script could not be executed.
 * Parameter: the client socket descriptor. */
/**********************************************************************/
void cannot_execute(int client)
{
    char buf[1024];

    /* 回应客户端 cgi 无法执行*/
    sprintf(buf, "HTTP/1.0 500 Internal Server Error\r\n");
    send(client, buf, strlen(buf), 0);

    sprintf(buf, "Content-type: text/html\r\n");
    send(client, buf, strlen(buf), 0);

    sprintf(buf, "\r\n");
    send(client, buf, strlen(buf), 0);

    sprintf(buf, "<P>Error prohibited CGI execution.\r\n");
    send(client, buf, strlen(buf), 0);
}

/**********************************************************************/
/* Print out an error message with perror() (for system errors; based
 * on value of errno, which indicates system call errors) and exit the
 * program indicating an error. */
/**********************************************************************/
void error_die(const char *sc)
{
    /*出错信息处理 */
    perror(sc);
    exit(1);
}



/**********************************************************************/
/* Execute a CGI script.  Will need to set environment variables as
 * appropriate.
 * Parameters: client socket descriptor
 *             path to the CGI script */
/**********************************************************************/
void execute_cgi(int client, const char *path, const char *method, const char *query_string)
{
    char buf[1024];
    int cgi_output[2];
    int cgi_input[2];
    pid_t pid;
    int status;
    int i;
    char c;
    int numchars = 1;
    int content_length = -1;

    buf[0] = 'A'; buf[1] = '\0';
    if (strcasecmp(method, "GET") == 0)
        /*把所有的 HTTP header 读取并丢弃*/
        while ((numchars > 0) && strcmp("\n", buf))  /* read & discard headers */
            numchars = get_line(client, buf, sizeof(buf));
    else {   /* POST */
        /* 对 POST 的 HTTP 请求中找出 content_length */
        numchars = get_line(client, buf, sizeof(buf));
        while ((numchars > 0) && strcmp("\n", buf)) {

            /*利用 \0 进行分隔 */
            buf[15] = '\0';

            /* HTTP 请求的特点*/
            if (strcasecmp(buf, "Content-Length:") == 0)
                content_length = atoi(&(buf[16]));
            numchars = get_line(client, buf, sizeof(buf));
        }

        /*没有找到 content_length */
        if (content_length == -1) {
            /*错误请求*/
            bad_request(client);
            return;
        }
    }

    /* 正确，HTTP 状态码 200 */
    sprintf(buf, "HTTP/1.0 200 OK\r\n");
    send(client, buf, strlen(buf), 0);

    /* 建立管道*/
    if (pipe(cgi_output) < 0) {
        /*错误处理*/
        cannot_execute(client);
        return;
    }

    /*建立管道*/
    if (pipe(cgi_input) < 0) {
        /*错误处理*/
        cannot_execute(client);
        return;
    }

    if ((pid = fork()) < 0 ) {
        /*错误处理*/
        cannot_execute(client);
        return;
    }

    if (pid == 0) {/* child: CGI script */
        char meth_env[255];
        char query_env[255];
        char length_env[255];

        /* 把 STDOUT 重定向到 cgi_output 的写入端 */
        dup2(cgi_output[1], 1);

        /* 把 STDIN 重定向到 cgi_input 的读取端 */
        dup2(cgi_input[0], 0);

        /* 关闭 cgi_input 的写入端 和 cgi_output 的读取端 */
        close(cgi_output[0]);
        close(cgi_input[1]);

        /*设置 request_method 的环境变量*/
        sprintf(meth_env, "REQUEST_METHOD=%s", method);
        putenv(meth_env);

        if (strcasecmp(method, "GET") == 0) {
            /*设置 query_string 的环境变量*/
            sprintf(query_env, "QUERY_STRING=%s", query_string);
            putenv(query_env);
        } else {   /* POST */
            /*设置 content_length 的环境变量*/
            sprintf(length_env, "CONTENT_LENGTH=%d", content_length);
            putenv(length_env);
        }

        /*用 execl 运行 cgi 程序*/
        execl(path, path, NULL);

        exit(0);
    } else {    /* parent */

        /* 关闭 cgi_input 的读取端 和 cgi_output 的写入端 */
        close(cgi_output[1]);
        close(cgi_input[0]);

        if (strcasecmp(method, "POST") == 0)
            /*接收 POST 过来的数据*/
            for (i = 0; i < content_length; i++) {
                recv(client, &c, 1, 0);

                /*把 POST 数据写入 cgi_input，现在重定向到 STDIN */
                write(cgi_input[1], &c, 1);
            }

        /*读取 cgi_output 的管道输出到客户端，该管道输入是 STDOUT */
        while (read(cgi_output[0], &c, 1) > 0)
            send(client, &c, 1, 0);

        /*关闭管道*/
        close(cgi_output[0]);
        close(cgi_input[1]);

        /*等待子进程*/
        waitpid(pid, &status, 0);
    }
}

/**********************************************************************/
/* Get a line from a socket, whether the line ends in a newline,
 * carriage return, or a CRLF combination.  Terminates the string read
 * with a null character.  If no newline indicator is found before the
 * end of the buffer, the string is terminated with a null.  If any of
 * the above three line terminators is read, the last character of the
 * string will be a linefeed and the string will be terminated with a
 * null character.
 * Parameters: the socket descriptor
 *             the buffer to save the data in
 *             the size of the buffer
 * Returns: the number of bytes stored (excluding null) */
/**********************************************************************/

int get_line(int sock, char *buf, int size)
{
    int i = 0;
    char c = '\0';
    int n;

    /*把终止条件统一为 \n 换行符，标准化 buf 数组*/
    while ((i < size - 1) && (c != '\n')) {
        /*一次仅接收一个字节*/
        n = recv(sock, &c, 1, 0);

        /* DEBUG printf("%02X\n", c); */
        if (n > 0) {

            /*收到 \r 则继续接收下个字节，因为换行符可能是 \r\n */
            if (c == '\r') {

                /*使用 MSG_PEEK 标志使下一次读取依然可以得到这次读取的内容，可认为接收窗口不滑动*/
                n = recv(sock, &c, 1, MSG_PEEK);

                /* DEBUG printf("%02X\n", c); */
                /*但如果是换行符则把它吸收掉*/
                if ((n > 0) && (c == '\n'))
                    recv(sock, &c, 1, 0);
                else
                    c = '\n';
            }

            /*存到缓冲区*/
            buf[i] = c;
            i++;
        } else
            c = '\n';
    }
    buf[i] = '\0';

    /*返回 buf 数组大小*/
    return(i);
}

/**********************************************************************/
/* Return the informational HTTP headers about a file. */
/* Parameters: the socket to print the headers on
 *             the name of the file */
/**********************************************************************/
void headers(int client, const char *filename)
{
    char buf[1024];
    (void)filename;  /* could use filename to determine file type */

    /*正常的 HTTP header */
    strcpy(buf, "HTTP/1.0 200 OK\r\n");
    send(client, buf, strlen(buf), 0);

    /*服务器信息*/
    strcpy(buf, SERVER_STRING);
    send(client, buf, strlen(buf), 0);

    sprintf(buf, "Content-Type: text/html\r\n");
    send(client, buf, strlen(buf), 0);

    strcpy(buf, "\r\n");
    send(client, buf, strlen(buf), 0);
}

/**********************************************************************/
/* Give a client a 404 not found status message. */
/**********************************************************************/
void not_found(int client)
{
    char buf[1024];

    /* 404 页面 */
    sprintf(buf, "HTTP/1.0 404 NOT FOUND\r\n");
    send(client, buf, strlen(buf), 0);

    /*服务器信息*/
    sprintf(buf, SERVER_STRING);
    send(client, buf, strlen(buf), 0);

    sprintf(buf, "Content-Type: text/html\r\n");
    send(client, buf, strlen(buf), 0);

    sprintf(buf, "\r\n");
    send(client, buf, strlen(buf), 0);

    sprintf(buf, "<HTML><TITLE>Not Found</TITLE>\r\n");
    send(client, buf, strlen(buf), 0);

    sprintf(buf, "<BODY><P>The server could not fulfill\r\n");
    send(client, buf, strlen(buf), 0);

    sprintf(buf, "your request because the resource specified\r\n");
    send(client, buf, strlen(buf), 0);

    sprintf(buf, "is unavailable or nonexistent.\r\n");
    send(client, buf, strlen(buf), 0);

    sprintf(buf, "</BODY></HTML>\r\n");
    send(client, buf, strlen(buf), 0);
}

/**********************************************************************/
/* Send a regular file to the client.  Use headers, and report
 * errors to client if they occur.
 * Parameters: a pointer to a file structure produced from the socket
 *              file descriptor
 *             the name of the file to serve */
/**********************************************************************/
void serve_file(int client, const char *filename)
{
    FILE *resource = NULL;
    int numchars = 1;
    char buf[1024];

    /*读取并丢弃 header */
    buf[0] = 'A'; buf[1] = '\0';
    while ((numchars > 0) && strcmp("\n", buf))  /* read & discard headers */
        numchars = get_line(client, buf, sizeof(buf));

    /*打开 sever 的文件*/
    resource = fopen(filename, "r");
    if (resource == NULL)
        not_found(client);
    else {
        /*写 HTTP header */
        headers(client, filename);

        /*复制文件*/
        cat(client, resource);
    }

    fclose(resource);
}

/**********************************************************************/
/* This function starts the process of listening for web connections
 * on a specified port.  If the port is 0, then dynamically allocate a
 * port and modify the original port variable to reflect the actual
 * port.
 * Parameters: pointer to variable containing the port to connect on
 * Returns: the socket */
/**********************************************************************/
int startup(u_short *port)
{
    int httpd = 0;
    struct sockaddr_in name;

    /*建立 socket */
    httpd = socket(PF_INET, SOCK_STREAM, 0);
    if (httpd == -1)
        error_die("socket");
    memset(&name, 0, sizeof(name));
    name.sin_family = AF_INET;
    name.sin_port = htons(*port);
    name.sin_addr.s_addr = htonl(INADDR_ANY);

    if (bind(httpd, (struct sockaddr *)&name, sizeof(name)) < 0)
        error_die("bind");

    /*如果当前指定端口是 0，则动态随机分配一个端口*/
    if (*port == 0) {/* if dynamically allocating a port */
        int namelen = sizeof(name);
        if (getsockname(httpd, (struct sockaddr *)&name, &namelen) == -1)
            error_die("getsockname");
        *port = ntohs(name.sin_port);
    }

    /*开始监听*/
    if (listen(httpd, 5) < 0)
        error_die("listen");

    /*返回 socket id */
    return(httpd);
}

/**********************************************************************/
/* Inform the client that the requested web method has not been
 * implemented.
 * Parameter: the client socket */
/**********************************************************************/
void unimplemented(int client)
{
    char buf[1024];

    /* HTTP method 不被支持*/
    sprintf(buf, "HTTP/1.0 501 Method Not Implemented\r\n");
    send(client, buf, strlen(buf), 0);

    /*服务器信息*/
    sprintf(buf, SERVER_STRING);
    send(client, buf, strlen(buf), 0);

    sprintf(buf, "Content-Type: text/html\r\n");
    send(client, buf, strlen(buf), 0);

    sprintf(buf, "\r\n");
    send(client, buf, strlen(buf), 0);

    sprintf(buf, "<HTML><HEAD><TITLE>Method Not Implemented\r\n");
    send(client, buf, strlen(buf), 0);

    sprintf(buf, "</TITLE></HEAD>\r\n");
    send(client, buf, strlen(buf), 0);

    sprintf(buf, "<BODY><P>HTTP request method not supported.\r\n");
    send(client, buf, strlen(buf), 0);

    sprintf(buf, "</BODY></HTML>\r\n");
    send(client, buf, strlen(buf), 0);
}

/**********************************************************************/

int main(void)
{
    int server_sock = -1;
    u_short port = 0;
    int client_sock = -1;
    struct sockaddr_in client_name;
    int client_name_len = sizeof(client_name);
    pthread_t newthread;

    /*在对应端口建立 httpd 服务*/
    server_sock = startup(&port);
    printf("httpd running on port %d\n", port);

    while (1) {
        /*套接字收到客户端连接请求*/
        client_sock = accept(server_sock,(struct sockaddr *)&client_name,&client_name_len);

        if (client_sock == -1)
            error_die("accept");

        /*派生新线程用 accept_request 函数处理新请求*/
        /* accept_request(client_sock); */

        if (pthread_create(&newthread , NULL, accept_request, client_sock) != 0)
            perror("pthread_create");
    }

    close(server_sock);
    return(0);
}

```

```cpp
#include <stdio.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <unistd.h>

int main(int argc, char *argv[])
{
    int sockfd;
    int len;
    struct sockaddr_in address;
    int result;
    char ch = 'A';

    sockfd = socket(AF_INET, SOCK_STREAM, 0);

    address.sin_family = AF_INET;
    address.sin_addr.s_addr = inet_addr("127.0.0.1");
    address.sin_port = htons(9734);
    len = sizeof(address);
    result = connect(sockfd, (struct sockaddr *)&address, len);

    if (result == -1) {
        perror("oops: client1");
        exit(1);
    }

    write(sockfd, &ch, 1);
    read(sockfd, &ch, 1);
    printf("char from server = %c\n", ch);

    close(sockfd);
    exit(0);
}

```
cgi
```cpp
/*
CGI(Common Gateway Interface)是HTTP服务器与你的或其它客户机 
上的程序进行联系的一种工具，其程序须运行在网络服务器上。是公共网关接口
如果只是查看代码，那记事本就可以。
如果运行，那要看CGI用什么语言来写的，如用PEAL写或C/C++写，如果是前者那么得需要个解释器，如：安装ActivePerl；如果是后者的，则直接运行。当然，也可以在IIS中影射CGI。 
*/
//check.cgi
#!/usr/local/bin/perl -Tw

use strict;
use CGI;

my($cgi) = new CGI;

print $cgi->header('text/html');
print $cgi->start_html(-title => "Example CGI script",
                       -BGCOLOR => 'red');

print $cgi->h1("CGI Example");
print $cgi->p, "This is an example of CGI\n";
print $cgi->p, "Parameters given to this script:\n";
print "<UL>\n";

foreach my $param ($cgi->param)
{
    print "<LI>", "$param ", $cgi->param($param), "\n";
}

print "</UL>";
print $cgi->end_html, "\n";

//color.cgi
#!/usr/local/bin/perl -Tw

use strict;
use CGI;

my($cgi) = new CGI;
print $cgi->header;

my($color) = "blue";
$color = $cgi->param('color') if defined $cgi->param('color');

print $cgi->start_html(-title => uc($color),
                       -BGCOLOR => $color);

print $cgi->h1("This is $color");
print $cgi->end_html;

//index.html
<HTML>
<TITLE>Index</TITLE>
    <BODY>
        <P>webserver.
        <H1>CGI demo

        <FORM ACTION="color.cgi" METHOD="POST">
        Enter a color: <INPUT TYPE="text" NAME="color">
        <INPUT TYPE="submit">
        </FORM>
    </BODY>
</HTML>
```


## 版本控制

| Version | Action                   | Time       |
| ------- | ------------------------ | ---------- |
| 1.0     | Init                     | 2019-05-02T01:24:48-07:00|
