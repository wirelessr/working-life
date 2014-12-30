# recvfrom vs. recvmsg

目標：收到一個UDP的封包，並且找出incoming interface
 
看起來很簡單的目標但著實不簡單，首先要告訴你，recvfrom無法做到這種高尚的工作，若是原來的程式是用recvfrom寫成的，第一步就必須是要將recvfrom改寫成recvmsg。

以下將介紹如何改寫recvfrom，但不會去講解這兩者的差異和原理。我們的目標是改寫原來的
> len = recvfrom(fd, msgbuf, UDP_MAX_PAYLOAD_LEN, 0, (struct sockaddr \*)dest, (socklen_t*)&destlen);    

將其改為recvmsg並且設定一些sockopt去取得interface的相關資訊

```
setsockopt(fd, SOL_IP, IP_PKTINFO, &on, sizeof(on));
iov.iov_base = (void*)msgbuf;
iov.iov_len = (size_t) UDP_MAX_PAYLOAD_LEN;
msg.msg_name = (struct sockaddr*)dest;
msg.msg_namelen = (socklen_t)destlen;
msg.msg_iov = &iov;
msg.msg_iovlen = 1;
msg.msg_control = cbuf;
msg.msg_controllen = sizeof(cbuf);

for(cmsg = CMSG_FIRSTHDR(&msg); cmsg != NULL; cmsg = CMSG_NEXTHDR(&msg, cmsg))
{
	if(cmsg->cmsg_level == SOL_IP && cmsg->cmsg_type == IP_PKTINFO)
	{
		struct in_pktinfo *i = (struct in_pktinfo *)CMSG_DATA(cmsg);
		if(i->ipi_ifindex)
		{
			if_indextoname(i->ipi_ifindex, ifname);
			printf("incoming interface(if: %d, name: %s)\n", i->ipi_ifindex, fname);
		}
		break;
	}
}
```
有兩個重點iov和msg_control。
 
iov我晚點會敘述，msg_control則是要達到setsockopt額外功能的話必備的，如果只是要將recvfrom改成recvmsg或是想使用iov的功能則可以清空不填。
 
重頭戲是如何取得IP_PKTINFO這個option的data呢？
 
man 7 ip和man recvmsg還有man getsockopt裡面幾乎都是屁話，一般來說，欲取得sockopt都是用getsockopt，所以，嘿嘿大錯特錯，這邊請特別注意用法，這是我特地去查linux kernel code才知道的，請用setsockopt並且帶上`IP_PKTINFO`，level也必須給對，是`SOL_IP`，而不是IPPROTO_IP。
 
用處是給msghdr帶上flag，當recvmsg看到這個flag，就會將東西放進msg_control中，而msg_control也不能直接取得東西，因為可能會上很多flag，所以他裡面是用串的，必須一個一個找，比對level和type都正確之後，就強制轉型成struct in_pktinfo (man 7 ip)
 
如此即可拿到device index，然後(好煩XDD)，用ioctl SIOCGIFNAME去轉出interface name
 
=====
 
後記：
 
前面有提到iov，iov其實是一種進階socket的用法，一般來說，socket讀取的東西只會放在一個buffer裡面，若多個地方去接收，程式必須自己去maintain重組或memcpy之類的事。
 
iov可以代入一個陣列，並且recvmsg會自動幫你把東西放對位置，大幅減少memory的access，只需要用pointer就可以搞定，詳細用法之後有機會會再放個程式碼
。