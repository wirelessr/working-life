# sendmsg & source addr

這篇的目的算是綜合前兩篇，首先會介紹sendmsg如何下IP_PKTINFO指定封包的source addr (與3.1相關)。
 
至於指定source addr的目的也就是為了指定outgoing interface (同3.2)。
 
不囉嗦，直接看code吧
```
struct msghdr msgh = {0};
struct cmsghdr *cmsg;
struct iovec iov;
char cmsgbuf[CMSG_SPACE(sizeof(struct in_pktinfo))];
struct in_pktinfo pktinfo = {0}, *pktinfo_ptr;

iov.iov_base = buf;
iov.iov_len = len;
msgh.msg_iov = &iov;
msgh.msg_iovlen = 1;
msgh.msg_control = cmsgbuf;
msgh.msg_controllen = sizeof(cmsgbuf);
msgh.msg_name = to; /* struct sockaddr: src addr */
msgh.msg_namelen = tolen;

cmsg = CMSG_FIRSTHDR(&msgh);

cmsg->cmsg_level = SOL_IP;
cmsg->cmsg_type = IP_PKTINFO;
cmsg->cmsg_len = CMSG_LEN(sizeof(struct in_pktinfo));
pktinfo.ipi_spec_dst = ((struct sockaddr_in *)from)->sin_addr;
pktinfo_ptr = (struct in_pktinfo *)CMSG_DATA(cmsg);
memcpy(pktinfo_ptr, &pktinfo, sizeof(struct in_pktinfo));
sendmsg(s, &msgh, 0);
```
setsockopt IP_PKTINFO的方法同3.1，不贅述了。memset(structure, 0, structure_size)可以用{0}的做法來代替，會好看一些些，也可以避免忘記初始化，或者用calloc取代malloc。我看過組語了，兩者做的事情是一樣的，{0}其實就是gcc偷偷幫你做的memset。
 
sendmsg和recvmsg一樣，要先填好一個msg header塞進去，只是請注意ip_pktinfo的填入方式，找這填入方式花了我很多時間！我本來以為是將recvmsg倒過來就好，但事情不是憨人想的這麼簡單。
 
稍微總結一下：
 
3.2透過iptables去指定outgoing interface，算是一種使用外力來導向封包的做法。
 
> app layer----->route----->output chain----->postroute chain----->data link layer
 
當route選定時，要再導向就只能透過iptables在output chain幫封包打mark，然後重新route一次，若是在app layer，也就是socket上就直接授予route information (例如source addr)就可以直接route對了。因此本篇提供另外一種導向封包的internal做法。
