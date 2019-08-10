---
title: 'IO复用select,poll'
date: 2018-06-20 14:25:07
tags:
- I/O模型
- 并发
categories:
- 网络编程
---

![I/O复用模型](http://p3euxxfa8.bkt.clouddn.com//18-6-20/7906302.jpg)

**IO多路复用是指内核一旦发现进程指定的一个或者多个IO条件准备读取，它就通知该进程**

使用场景:
1) 客户处理多个描述符时, 必须使用IO复用
2) 一个客户同时处理多个套接字
3) 一个TCP服务器既要处理监听套接字, 又要处理已连接套接字
4) 一个服务器既要处理TCP, 又要处理UDP
5) 一个服务器要处理多个协议

# 原始客户端程序

```c
#include	"unp.h"

int
main(int argc, char **argv)
{
	int					sockfd;
	struct sockaddr_in	servaddr;

	if (argc != 2)
		err_quit("usage: tcpcli <IPaddress>");

	sockfd = Socket(AF_INET, SOCK_STREAM, 0);

	bzero(&servaddr, sizeof(servaddr));
	servaddr.sin_family = AF_INET;
	servaddr.sin_port = htons(SERV_PORT);
	Inet_pton(AF_INET, argv[1], &servaddr.sin_addr); // 创建tcp套接字

	Connect(sockfd, (SA *) &servaddr, sizeof(servaddr));  // tcp连接

	str_cli(stdin, sockfd);		/* str_cli完成具体的客户端处理 */

	exit(0);
}

void
str_cli(FILE *fp, int sockfd)
{
	char	sendline[MAXLINE], recvline[MAXLINE];

	while (Fgets(sendline, MAXLINE, fp) != NULL) {  

		Writen(sockfd, sendline, strlen(sendline)); /* */

		if (Readline(sockfd, recvline, MAXLINE) == 0)
			err_quit("str_cli: server terminated prematurely");

		Fputs(recvline, stdout);
	}
}
```

这里`Fgets`如下

```c
char *
Fgets(char *ptr, int n, FILE *stream)
{
	char	*rptr;

	if ( (rptr = fgets(ptr, n, stream)) == NULL && ferror(stream))
		err_sys("fgets error");

	return (rptr);
}
```

可以看到调用的是`fgets`, 这是一个`<stdio.h>`中的函数
```c
char *fgets(char *s, int size, FILE *stream);

返回值：成功时s指向哪返回的指针就指向哪，出错或者读到文件末尾时返回NULL
```

fgets从指定的文件中读一行字符到调用者提供的缓冲区中. 参数s是缓冲区的首地址，size是缓冲区的长度，该函数从stream所指的文件中读取以'\n'结尾的一行（包括'\n'在内）存到缓冲区s中，并且在该行末尾添加一个'\0'组成完整的字符串。

具体到这个程序, 就是从标准输入读取一行文本到 sendline中, 然后使用`Writen` 将读取到的内容发送到sockfd中.

然后Readline从sockfd中读取数据, 写入recvline, 最后使用Fputs将缓冲区中的类容输出到标准输出

## 问题
可以看到, 上述代码同时处理两个输入: 标准输入和tcp套接字, 当客户阻塞于fgets调用时, 如果服务器进程被杀死, 这个时候虽然服务器给客户端发送了一个FIN, 但是因为客户端进程正阻塞于标准输入, 它将看不到这个EOF. 为了解决这个问题, 需要一种机制,可以监视多个描述符，一旦某个描述符就绪（一般是读就绪或者写就绪），能够通知程序进行相应的读写操作. 这个机制就是**IO多路复用**. 这种机制基于`select`, `poll` 这两个函数.

# select
## select函数
```c
int select(int maxfdp1, fd_set *readset, fd_set *writeset, fd_set *exceptset,struct timeval *timeout);
```

该函数返回时, 结果将指示哪些哪些描述符已就绪. 函数返回后, 调用FD_ISSET来测试fd_set中的描述符. 描述符内任何与未就绪描述符对应的位返回时均清成0


### timeout
timeout 可设置的值有三种
1) 空值, 表示一直等待下去, 直到某一个描述符就绪
2) 一定的时间
3) 根本不等待, 检查描述符后立即返回, 这称为轮询. 这种情况下该参数指向一个timeval结构, 其中的定时器值为0. 为此, 每次调用select时需要将自己关心的描述符置为1

### readset, writeset, exceptset
这三个参数指定需要内核测试读, 写, 和异常条件的描述符

select使用描述符集, 通常是一个整数数组, 每个整数的每一位对应一个描述符

### maxfdp1
待测试的描述符个数, 它的值是待测试的最大描述符加1, 描述符0, 1, ..., maxfdp1-1均将被测试

## select版本的str_cli函数

```c
void
str_cli(FILE *fp, int sockfd)
{
	int			maxfdp1;
	fd_set		rset;
	char		sendline[MAXLINE], recvline[MAXLINE];

	FD_ZERO(&rset); // FD_ZERO宏将一个 fd_set类型变量的所有位都设为 0
	for ( ; ; ) {
		FD_SET(fileno(fp), &rset); // FD_SET将变量的某个位置位1. 这里关心的是fileno 和sockfd
		FD_SET(sockfd, &rset);
		maxfdp1 = max(fileno(fp), sockfd) + 1; //获得maxfdp1
		Select(maxfdp1, &rset, NULL, NULL, NULL);


        /*如果sockfd准备就绪, 将sockfd中的类容读入recvline, 最终输出到标准输出 */
		if (FD_ISSET(sockfd, &rset)) {	/* socket is readable */
			if (Readline(sockfd, recvline, MAXLINE) == 0)
				err_quit("str_cli: server terminated prematurely");
			Fputs(recvline, stdout);
		}

        /*在这里, 如果从标准输入读到数据, 就将输入写到sendline, 然后写入sockfd*/
		if (FD_ISSET(fileno(fp), &rset)) {  /* input is readable */
			if (Fgets(sendline, MAXLINE, fp) == NULL)
				return;		/* all done */
			Writen(sockfd, sendline, strlen(sendline));
		}
	}
}
```

上面这个方法已经足够应对单对单的请求, 但是如果是批量请求, 标准输入的EOF并不意味着可以关闭通道, 因为这时候可能套接字中依然还存在类容. 这种情况下可以通过shutdown来告诉服务器已经完成了数据的发送, 但是仍然保持套接字描述符打开以便于读取.

unp中解决这个问题的代码如下:

```c
void
str_cli(FILE *fp, int sockfd)
{
	int			maxfdp1, stdineof;
	fd_set		rset;
	char		buf[MAXLINE];
	int		n;

	stdineof = 0;
	FD_ZERO(&rset);
	for ( ; ; ) {
		if (stdineof == 0)
			FD_SET(fileno(fp), &rset);
		FD_SET(sockfd, &rset);
		maxfdp1 = max(fileno(fp), sockfd) + 1;
		Select(maxfdp1, &rset, NULL, NULL, NULL);


        /*当在套接字上读到EOF, 如果已经在标准输入中读到EOF, 则为正常终止, 否则为服务器过早关闭 */
		if (FD_ISSET(sockfd, &rset)) {	/* socket is readable */
			if ( (n = Read(sockfd, buf, MAXLINE)) == 0) {  /*Read返回值：成功返回读取的字节数，出错返回-1并设置errno，如果在调read之前已到达文件末尾，则这次read返回0*/
				if (stdineof == 1) 
					return;		/* normal termination */
				else
					err_quit("str_cli: server terminated prematurely");
			}

			Write(fileno(stdout), buf, n);
		}

		if (FD_ISSET(fileno(fp), &rset)) {  /* input is readable */
			if ( (n = Read(fileno(fp), buf, MAXLINE)) == 0) { 
				stdineof = 1; // 标准输入EOF, 标志位置1,发送FIN
				Shutdown(sockfd, SHUT_WR);	/* send FIN */
				FD_CLR(fileno(fp), &rset);
				continue;
			}

			Writen(sockfd, buf, n);
		}
	}
}
```

## select版本的服务器程序
```c
int
main(int argc, char **argv)
{   
    ... // 省略创建套接字并连接的部分
    maxfd = listenfd;			/* initialize */
    maxi = -1;					/* index into client[] array */
    for (i = 0; i < FD_SETSIZE; i++)
        client[i] = -1;			/* -1 indicates available entry */
    FD_ZERO(&allset);
    FD_SET(listenfd, &allset);
    for ( ; ; ) {
		rset = allset;		/* structure assignment */
		nready = Select(maxfd+1, &rset, NULL, NULL, NULL); /* select等待某个事件的发生*/

		if (FD_ISSET(listenfd, &rset)) {	/* new client connection. 监听套接字可读, 说明建立了一个新连接 */ 
			clilen = sizeof(cliaddr);
			connfd = Accept(listenfd, (SA *) &cliaddr, &clilen); //调用Accept
#ifdef	NOTDEF
			printf("new client: %s, port %d\n",
					Inet_ntop(AF_INET, &cliaddr.sin_addr, 4, NULL),
					ntohs(cliaddr.sin_port));
#endif

            // 更新数据结构
			for (i = 0; i < FD_SETSIZE; i++)  //  client[FD_SETSIZE]
				if (client[i] < 0) {
					client[i] = connfd;	/* save descriptor, 使用第一个未用项记录这个已连接描述符, 然后跳出循环.  */ 
					break;
				}
			if (i == FD_SETSIZE)
				err_quit("too many clients");

			FD_SET(connfd, &allset);	/* add new descriptor to set */
			if (connfd > maxfd)
				maxfd = connfd;			/* for select */
			if (i > maxi)
				maxi = i;				/* max index in client[] array */

			if (--nready <= 0)
				continue;				/* no more readable descriptors */
		}

		for (i = 0; i <= maxi; i++) {	/* check all clients for data */
			if ( (sockfd = client[i]) < 0)
				continue;
			if (FD_ISSET(sockfd, &rset)) {
				if ( (n = Read(sockfd, buf, MAXLINE)) == 0) {
						/*4connection closed by client */
					Close(sockfd);
					FD_CLR(sockfd, &allset);
					client[i] = -1;
				} else
					Writen(sockfd, buf, n);

				if (--nready <= 0)
					break;				/* no more readable descriptors */
			}
		}
	}
}
```

# poll
## poll函数

```c
int poll(struct pollfd *fdarray, unsigned long nfds, int timeout);  
``` 

- fdarray. 指向一个poofd结构的第一个元素的指针, 用于指定某个给定描述符的测试条件. 

```c
struct pollfd {
    int fd;
    short events;
    short revents;
}
```

- 每一个 pollfd 结构体指定了一个被监视的文件描述符
- events：指定监测fd的事件（输入、输出、错误），每一个事件有多个取值

![events取值](http://p3euxxfa8.bkt.clouddn.com/2018-06-22-10-54-06.png)

- revents 域是文件描述符的操作结果事件  

- nfds:用来指定第一个参数数组元素个数
- timeout: 指定等待的毫秒数，无论 I/O 是否准备好，poll() 都会返回.

timeout值|说明|
---|---|
INFTIM | 永远等待
0 | 立即返回 ,不阻塞进程
>0 | 等待指定数目的毫秒数

- poll函数的返回值: 成功时，poll() 返回结构体中 revents 域不为 0 的文件描述符个数；如果在超时前没有任何事件发生，poll()返回 0；失败时，poll() 返回 -1，并设置 errno 

## 基于poll的服务端程序

```c
int
main(int argc, char **argv)
{   
    ... // 省略初始化相关代码, listen函数将一个主动套接字转化为监听套接字
    client[0].fd = listenfd;
	client[0].events = POLLRDNORM; //client[0] 固定用于监听套接字
	for (i = 1; i < OPEN_MAX; i++)
		client[i].fd = -1;		/* -1 indicates available entry */
	maxi = 0;					/* max index into client[] array */
    for ( ; ; ) {
		nready = Poll(client, maxi+1, INFTIM);

		if (client[0].revents & POLLRDNORM) {	/* new client connection */
			clilen = sizeof(cliaddr);
			connfd = Accept(listenfd, (SA *) &cliaddr, &clilen);
#ifdef	NOTDEF
			printf("new client: %s\n", Sock_ntop((SA *) &cliaddr, clilen));
#endif

			for (i = 1; i < OPEN_MAX; i++)
				if (client[i].fd < 0) {
					client[i].fd = connfd;	/* save descriptor */
					break;
				}
			if (i == OPEN_MAX)
				err_quit("too many clients");

			client[i].events = POLLRDNORM;
			if (i > maxi)
				maxi = i;				/* max index in client[] array */

			if (--nready <= 0)
				continue;				/* no more readable descriptors */
		}

		for (i = 1; i <= maxi; i++) {	/* check all clients for data */
			if ( (sockfd = client[i].fd) < 0)
				continue;
			if (client[i].revents & (POLLRDNORM | POLLERR)) {
				if ( (n = read(sockfd, buf, MAXLINE)) < 0) {
					if (errno == ECONNRESET) {
							/*4connection reset by client */
#ifdef	NOTDEF
						printf("client[%d] aborted connection\n", i);
#endif
						Close(sockfd);
						client[i].fd = -1;
					} else
						err_sys("read error");
				} else if (n == 0) {
						/*4connection closed by client */
#ifdef	NOTDEF
					printf("client[%d] closed connection\n", i);
#endif
					Close(sockfd);
					client[i].fd = -1;
				} else
					Writen(sockfd, buf, n);

				if (--nready <= 0)
					break;				/* no more readable descriptors */
			}
		}
	}
}
```

pollfd结构包含了要监视的event和发生的event，不再使用select“参数-值”传递的方式。同时，pollfd并没有最大数量限制（但是数量过大后性能也是会下降）。 和select函数一样，poll返回后，需要轮询pollfd来获取就绪的描述符。