# Network virtualize lab DIY

Network virtualize以白話來講就是將邏輯網路和實體網路切開看，說起來很抽象，簡單說就是以network layer為邊界，將網路分成上下兩層，應用程式等功能都在上層開發，而路由、選逕、ECMP、QoS等工作則是在下層。
 
為什麼會有network virtualize這名詞誕生，主要還是因為雲端技術拓展迅速導致在大型data center內的VLAN不夠切了；VLAN被802.1Q的header length限定大小，最多只能有4094個VLAN (扣掉0和FFF)，許多技術因此應運而生，例如NVGRE或VXLAN。
 
這篇文章要講如何建造一個能夠實作各種network virtualize技術的環境，如果，你是有數十台甚至數百台PC的人請略過這篇，我手上只有一台PC和兩台NB，卻得要模擬十幾台router & switch & controller & end-point。
 
所以我選擇VM來達成，事實上，data center也是以VM來切割overlay的。我嘗試過幾種方法，這邊稍微列出我碰到的障礙：
 
1. user mode linux：在kernel 3.13上，job control搞不定，job control就是^C可以終止、^Z可以暫停，諸如此類的管理機制
2. kvm：感覺我對一堆文字的東西滿沒天分的，他的參數實在太複雜，而且要把虛擬網路弄出來還得透過vde2，放棄
3. virtualbox：好物！虛擬網路非常單純好懂，造就virtualbox根本完全沒有上手難度
 

根據以上，最後我選了virtualbox，附帶提一下，guest OS我用ubuntu 14.04 server版，驚為天人阿，128MB的RAM就可以帶起來了，而且只要硬碟4G不到就可以把我要的功能全部裝完，重點是，開機不到30秒！我4G RAM的PC剛好可以用12個VM榨乾，這已經可以做到幾乎全部我想建立的topology了。
 
switch的部分我用bridge-utils和vlan去做，當然也可以選用open vswitch，但他太肥而且之後我們機器也沒要port他就沒去用；router我是用xorp，這應該是guest裡面最肥的套件，但超強大，連multicast都可以搞定。其他零零總總就不一一列了。

![](http://i.imgur.com/BX7WNDH.jpg)

快速上手的訣竅是先建一個VM-template，把要裝的套件全部給他倒進去，設定全部不要設！接著重點來了，一定要裝virtualbox guest additions，因為要讓guest去mount本機的資料。
 
在rc.local寫下
 ```
mount.vboxsf VM /root/share
HOST=`cat /root/share/mapping | grep $(cat /sys/class/net/eth0/address) | cut -d ' ' -f 2`
hostname -b $HOST
[ -e "/root/share/$(hostname)/setup" ] && /root/share/$(hostname)/setup
 ```
首先先mount一個shared folder "VM"，這裡面會放一堆各個VM的設定檔，因為我要做到全自動開機環境，然後根據mac去查表並且設定hostname，一方面是好管理，host上會開一堆ssh，如果沒hostname會想死；另一方面是，有hostname就可以在shared folder根據hostname去創folder，然後把各個VM的設定放進去。
 
例如
```
echo 0 > /proc/sys/net/ipv4/icmp_echo_ignore_broadcasts
echo 1 > /proc/sys/net/ipv4/ip_forward
 
ip addr add 192.168.11.111/24 dev eth0
ip addr add 192.168.12.111/24 dev eth1
ip addr add 192.168.101.111/24 dev eth2
ip addr add 192.168.102.111/24 dev eth3
```
這是我某一台router的部分設定。

然後把設定都擺好後就把VM全部點開，就可以玩任何想玩的試驗了，例如ping或iperf之類的，之後把這些設定打包，就可以再換下一個topology。
 
最後提一下，virtualbox內有很多虛擬網路的選項，我只用了兩個
 
1. 橋接：用來給host連SSH和擴站用的，因為NB也在同一個LAN下，所以要透過橋接來擴充(如果不只12個VM的話)
2. 內部網路：這是network lab的core，而且這有個好處，只要給不同的ID就可以直接切開，類似VLAN，非常方便
 

初學者搞大topology有幾個點要注意：
 
1. NAT模式下host的LAN跟guest是無法通的，一定要bridge，即使port forwarding也沒用。
2. 透過橋接後，內部網路即使ID相同，也不會通。
3. guest-only network是在幹嘛我沒有試過，但感覺很麻煩我就沒去用他了，因為幾乎所有的需求靠內部網路就能滿足。







