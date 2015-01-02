# NVGRE loses DHCP OFFER

從前我一直認為，寫application的人不需要去懂kernel到底幹了啥，只要知道他能夠使命必達，而programmer就專心的把上層軟體架構想清楚即可。但事實證明我錯了，而我這次解的這問題也著實打了我好大一巴掌，所以之後我會更認真去看kernel的細節，至少網路相關的要很熟。
 
以下是張示意圖，左邊是wireless controller，右邊是AP；而controller會與AP建立一條tunnel，以便將所有連上AP的STA traffic都導回controller。

![](http://i.imgur.com/h1NuqZm.jpg)

###問題
 
Controller和AP用192.168.1.x的網段建tunnel，但tunnel卻是被bridge在br3上，而controller的br3上面有個DHCP server，負責發IP給br3上面的STA。
 
封包流向理當是：
> DHCP DISCOVER→wlan-1-1.3→gretap1→br0→eth0→→→→→vlan2.1→br0→gretap1→br3   
> DHCP OFFER→br3→gretap1→br0→vlan2.1→→→→→eth0→br0→gretap1→wlan-1-1.3

但DHCP OFFER卻送到一半就不見了，接著我用tcpdump去追封包掉在哪，發現br0上就已經抓不到了。但是在iptables和ebtables的hook點上確實有看到東西流過去，百思不得其解下我直接插printk在kernel的`br_dev_xmit`、`vlan_hard_start_xmit`、`dev_queue_xmit` (熟kernel的人應該知道我這根本把printk插在整個packet flow上...)，然後我發現，同一個skb的pointer很順利的從br3一路通到vlan2.1出去。
 
(／‵′)／~ ╧╧
 
那tcpdump在搞啥鬼，所以我做了一個很痛的決定，我在原本插printk的點全部改成`printk_hex` (我自己寫的，用來把封包從頭到尾dump出來)，最後總算發現是掉在gretap1內，也就是`ipgre_tunnel_xmit`。

深入去了解後發現到，skb指向IP header的指針根本指錯了，導致IP header說不需要fragmentation但GRE看到的卻是IP header內的total length大到連他媽都不認識，遠超過IP的MTU。

![](http://i.imgur.com/AoJJDtd.jpg)

但是我在ubuntu上面玩同樣的topology卻一點事也沒有。最後找到原因，一般的application如果要丟IP包或L2包必須要用`PF_PACKET`(或`AF_PACKET`，一樣的東西，kernel會轉)去丟；而`PF_PACKET`又分成兩socket type，`SOCK_PACKET`和`SOCK_RAW`。功能很像，但做的事不太一樣，這篇不去探討詳細差別，最主要是它們進kernel的點不一樣，一個是`packet_sendmsg_spkt`另一個是`packet_sendmsg`。
 
關鍵在於`packet_sendmsg_spkt`會去調整skb的data指針指向正確的network header，如圖黃色區塊：

![](http://i.imgur.com/ydA8gqX.jpg)

而`packet_sendmsg`則是user space丟下來是甚麼就是甚麼，controller上的DHCP server用的是`SOCK_RAW`，但在user space卻有將ethernet header填起來，導致應該指向IP header的指針指的其實是Ethernet header，最後，修正controller上的DHCP server的行為就一切正常了。




 
如示意圖