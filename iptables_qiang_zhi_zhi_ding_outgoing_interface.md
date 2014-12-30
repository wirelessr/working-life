# iptables 強制指定 outgoing interface

強制指定outgoing interface其實寫routing table也能辦到，但是如果條件不是只有ip和interface的話，對route來說就無能為力了 (下面的範例條件包含mac, ip & port)。
 
因此必須透過iptables，以下是個範例
 
> iptables -A OUTPUT -d 10.1.1.10 -m mac --mac-source 00:11:22:33:44:55 -p udp --dport 40000 -j MARK --set-mark 11    
> ip rule add fwmark 11 table 777    
> ip route add table 777 dev eth3    
 
稍微講解一下
 
首先我對本機出來的封包搜尋條件，若目的地是10.1.1.10:40000且是從00:11:22:33:44:55這mac送出來的封包就打一個mark: 11。
 
然後建一條ip rule，若封包被上了mark 11就去查找route table 777。
 
最後在route table 777裡面加一條route (也是唯一的一條)，直接從eth3出去，這樣就完成了。若不是要指定outgoing interface，而是要指定gateway的話，將`dev eth3`改成`via 192.168.1.111`即可。