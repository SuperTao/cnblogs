

https://www.cnblogs.com/liunianshiwei/p/6045303.html

```
#include <stdio.h>
#include <string.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <fcntl.h>
#include <sys/un.h>

#define WIRELESS_SERVER_SOCKET "/var/tmp/wireless_server_socket"
#define WIRELESS_CLIENT_SOCKET "/var/tmp/wireless_client_socket"

int main(void)
{
	int ret;
	int backlog = 5;
	int server_socketfd, client_socketfd;
	int socklen;
	char buf[64];
	struct sockaddr_un server_sockaddr, client_sockaddr;
	// 创建socket
	printf("create socket.....\n");
	server_socketfd = socket(AF_UNIX, SOCK_STREAM, 0);
	if (server_socketfd < 0)
	{
		printf("Fail to create server socket\n");
		return -1;
	}
	// 创建UNIX域时会生成相应的文件，如果已经存在则bind会失败，需要先删除
	if(remove(WIRELESS_SERVER_SOCKET) < 0)
	{
		printf("Fail to remove %s", WIRELESS_SERVER_SOCKET);
	}

	// 将socket与本地文件绑定
	server_sockaddr.sun_family = AF_UNIX;
	strncpy(server_sockaddr.sun_path, WIRELESS_SERVER_SOCKET, sizeof(server_sockaddr.sun_path) - 1 );
	socklen = sizeof(server_sockaddr);
	printf("bind socket.....\n");
	ret = bind(server_socketfd, (struct sockaddr *)&server_sockaddr, socklen);
	if (ret < 0)
	{
		printf("Fail to bind wireless socket\n");
		return -2;
	}
	
	printf("listen.....\n");
	// 设置监听个数，最多可以有5个连接
	ret = listen(server_socketfd, backlog);
	if (ret < 0)
	{
		printf("Fail to listen server_sockfd\n");
		return -3;
	}

	printf("accept.....\n");
	// 等待客户端连接
	socklen = sizeof(client_sockaddr);
	ret = accept(server_socketfd, (struct sockaddr *)&client_sockaddr, &socklen);
	if (ret < 0)
	{
		printf("Fail to accept server_sockfd\n");
		return -3;
	}
	// 获取客户端的socket
	client_socketfd = ret;

	printf("client connect!\n");
	printf("client sun_path: %s\n", client_sockaddr.sun_path);

	while(1)
	{
		memset(buf, 0, sizeof(buf));
		ret = read(client_socketfd, buf, sizeof(buf));
		printf("read %d byte: %s\n", ret, buf);
		write(client_socketfd, "success", 8);
	}

	return 0;
}
```

```
#include <stdio.h>
#include <string.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <fcntl.h>
#include <sys/un.h>

#define WIRELESS_SERVER_SOCKET "/var/tmp/wireless_server_socket"
#define WIRELESS_CLIENT_SOCKET "/var/tmp/wireless_client_socket"

int main(void)
{
	int ret, len;
	int backlog = 5;
	int server_socketfd, client_socketfd;
	int socklen;
	char buf[64];
	struct sockaddr_un server_sockaddr, client_sockaddr;
	// 创建socket
	printf("create socket.....\n");
	client_socketfd = socket(AF_UNIX, SOCK_STREAM, 0);
	if (client_socketfd < 0)
	{
		printf("Fail to create socket\n");
		return -1;
	}
	// 创建UNIX域时会生成相应的文件，如果已经存在则bind会失败，需要先删除
	if(remove(WIRELESS_CLIENT_SOCKET) < 0)
	{
		printf("Fail to remove %s\n", WIRELESS_CLIENT_SOCKET);
	}

	// 将socket与本地文件绑定
	client_sockaddr.sun_family = AF_UNIX;
	strncpy(client_sockaddr.sun_path, WIRELESS_CLIENT_SOCKET, sizeof(client_sockaddr.sun_path) - 1 );
	socklen = sizeof(server_sockaddr);
	printf("bind socket.....\n");
	ret = bind(client_socketfd, (struct sockaddr *)&client_sockaddr, socklen);
	if (ret < 0)
	{
		printf("Fail to bind wireless socket\n");
		return -2;
	}

	// 连接服务器
	server_sockaddr.sun_family = AF_UNIX;
	strncpy(server_sockaddr.sun_path, WIRELESS_SERVER_SOCKET, sizeof(server_sockaddr.sun_path) - 1 );
	len = sizeof(server_sockaddr);
	ret = connect(client_socketfd, (struct sockaddr *)&server_sockaddr, len);
	if (ret < 0)
	{
		printf("connect fail\n");
		return -3;
	}

	while(1)
	{
		memset(buf, 0, sizeof(buf));
		fgets(buf, sizeof(buf), stdin);
		ret = write(client_socketfd, buf, strlen(buf));
		printf("read %d byte: %s\n", ret, buf);
		read(client_socketfd, buf, sizeof(buf));
		printf("%s\n", buf);
	}

	return 0;
}
```