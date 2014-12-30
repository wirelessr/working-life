# Multicast IPC

在linux programming中有五種常用的IPC (inter-processes communication)，白話的講就是行程間的溝通機制，有pipe、shared memory、signal、message queue和domain socket。

這些IPC中除了signal外其他都是unicast的，一個發送端一次只能通知一個process，而signal雖然可以一次通知給許多processes，但是無法夾帶資訊，只能用signal ID去identify。

若是熟悉linux kernel programming的人一定知道，kernel space和user space的溝通方法有許多種，其中一種：netlink，就可以支援multicast，也就是kernel有許多event會送給user space中有去listen的process。

這機制很重要，所謂的射後不理就是這樣，還可以一次射很多，這篇要來介紹一種機制，使得IPC也可以做到multicast，誠如上述，五種常見機制都辦不到，因此要別開蹊徑，這邊我採用IP layer的multicast。

透過IGMP和join group的方式做到，但因為是IPC，所以不希望射到外部網路上，僅僅在loopback上打。

下面是server端和client端的source code，請自行參考
```
sock = socket(AF_INET, SOCK_DGRAM, 0);

addr.sin_family = AF_INET;
addr.sin_port = htons(HELLO_PORT);

inet_pton(AF_INET, HELLO_GROUP, &addr.sin_addr.s_addr);
inet_pton(AF_INET, "127.0.0.1", &ipaddr.s_addr);

if(setsockopt(sock, IPPROTO_IP, IP_MULTICAST_IF, (char*)&ipaddr, sizeof(ipaddr)) != 0)
{
	perror("setsockopt");
	exit(EXIT_FAILURE);
}

while(1)
{
	n = sendto(sock, msg, strlen(msg), 0, (struct sockaddr *)&addr, sizeof(addr));
	
	if(n < 1)
	{
		perror("sendto");
	}
	sleep(1);
}
```
`HELLO_PORT`和`HELLO_GROUP`要自行定義，且server端和client端要一致。

```
sock = socket(AF_INET, SOCK_DGRAM, 0);

if(setsockopt(sock, SOL_SOCKET, SO_REUSEADDR, &yes, sizeof(yes)) != 0)
{
	perror("setsockopt");
	exit(EXIT_FAILURE);
}

addr.sin_family = AF_INET;
addr.sin_port = htons(HELLO_PORT);
addr.sin_addr.s_addr = htonl(INADDR_ANY);

inet_pton(AF_INET, HELLO_GROUP, &addr.sin_addr.s_addr);
inet_pton(AF_INET, "127.0.0.1", &ipaddr.s_addr);

if(bind(sock, (struct sockaddr *)&addr, sizeof(addr)) < 0)
{
	perror("bind");
	exit(EXIT_FAILURE);
}

mreq.imr_multiaddr.s_addr = inet_addr(HELLO_GROUP);
inet_pton(AF_INET, "127.0.0.1", &mreq.imr_multiaddr.s_addr);

if(setsockopt(sock, IPPROTO_IP, IP_ADD_MEMBERSHIP, &mreq, sizeof(mreq)) != 0)
{
	perror("setsockopt");
	exit(EXIT_FAILURE);
}

while(1)
{
	n = recvfrom(sock, msg, MAX_BUF_LEN, 0, (struct sockaddr *)&addr, &addrlen));
	
	if(n > 0)
	{
		printf("%s\n", msg);
	}
	sleep(1);
}
```

