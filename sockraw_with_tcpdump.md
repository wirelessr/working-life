# SOCK_RAW with tcpdump

Raw socket是linux network programming一個很進階的技巧，大致上說起來就是，跳過socket所在的transport layer，直接往下撈"封包"，也不能說封包(packet)，因為raw socket撈到的是frame；其實就是從linux kernel的ip_recv直接殺一條路出來，將尚未解析的frame往user space丟，有註冊的application就可以收起來看，當然就包括L3 header甚至L2 header。
 
這篇要講得不是raw socket怎麼寫和怎麼解析封包，那個隨便孤狗都有，不過還是給個簡單的例子吧
```
gRawSock = socket(PF_PACKET, SOCK_RAW, htons(ETH_P_ALL));

memset(&addr, 0, sizeof(addr));
addr.sll_family = AF_PACKET;
addr.sll_protocol = htons(ETH_P_ALL);
addr.sll_pkttype = PACKET_HOST;
addr.sll_ifindex = if_nametoindex(gInterfaceName);

bind(gRawSock, (struct sockaddr *)&addr, sizeof(addr));

strncpy(ethreq.ifr_name, gInterfaceName, IFNAMSIZ);
ioctl(gRawSock, SIOCGIFFLAGS, &ethreq);

ethreq.ifr_flags |= IFF_PROMISC;
ioctl(gRawSock, SIOCSIFFLAGS, &ethreq);
```
例子中的raw socket綁在gInterfaceName上，我也不想亂亂抓。另外提醒一個觀念，set if flag記得用|=運算，不然會把自家系統搞死。
 
接著就是recvfrom後冗長的解析header和擷取資料了，若是今天想抓的frame實在很單純不過，例如我只想知道192.168.1.155這個host上的icmp包，直接把interface上所有的封包收下來再去脫衣服脫帽子實在勞民傷財，尤其牽扯到thoughput的工作在user space自己搞，那個效能會爛到很可怕。

這時候就要用今天的主角2了，tcpdump，相信沒甚麼人有去看tcpdump的man page，裡面有一行其實點出重點了
 
> -dd    Dump packet-matching code as a C program fragment
 
其實tcpdump可以直接產生C code用的封包遮罩，就是透過`-dd`，回到上面『我只想知道192.168.1.155這個host上的icmp包』其實只要
 
> tcpdump -dd host 192.168.1.155 and icmp
 
就可以產生封包遮罩了，以下是使用方法：
```
struct sock_fprog filter;
struct sock_filter bpf_code[] = {
	{ 0x28, 0, 0, 0x0000000c },
	{ 0x15, 0, 7, 0x00000800 },
	{ 0x20, 0, 0, 0x0000001a },
	{ 0x15, 2, 0, 0xc0a8019b },
	{ 0x20, 0, 0, 0x0000001e },
	{ 0x15, 0, 3, 0xc0a8019b },
	{ 0x30, 0, 0, 0x00000017 },
	{ 0x15, 0, 1, 0x00000001 },
	{ 0x6, 0, 0, 0x0000ffff },
	{ 0x6, 0, 0, 0x00000000 },
};
filter.len = sizeof(bpf_code)/sizeof(bpf_code[0]);
filter.filter = bpf_code;

setsockopt(gRawSock, SOL_SOCKET, SO_ATTACH_FILTER, &filter, sizeof(filter));
```
bpf_code內的東西就是由tcpdump產生的，不要懷疑，如此一來，gRawSock就只會抓到符合條件的frame，不需要再一個一個frame去查header內容了。

