# Wireshark dissector

這篇要介紹一下wireshark dissector的plugin怎麼新增，之所以需要掛新的dissector是因為自創portocol導致原生的wireshark filter解錯或者不會解。需要的原因是因為我在IP GRE(IP proto 47)的header後面有塞一段自定義的header，在1.8版的時候，要新掛一個dissector是很麻煩的，當初我為了偷時間，直接去抓一包wireshark source code然後硬是把原生的IP proto 47複寫，而且只挑GRE type是0x0800的做，然後用gtk build，弄出一包在windows上可以直接執行的binary。
 
這直接導致兩個苦果
 
1. 所有的0x0800都被合勤的header重載了，也就是wireshark看不懂原生的GRE
2. 因為ARP封包用GRE包的時候proto type會是0x0806，所以即使改wireshark code還是讀不懂ARP
 

現在的wireshark已經進到1.10版，我發現牠支援外掛Lua的dissector (Lua是一款羽量級的直譯式語言)，這邊就用vxlan當作介紹的範例吧！在開始之前先提一下VXLAN，VXLAN是一款最近推出的重量級的封裝協議，基於UDP: 8472做的，將一個完整的L2 frame(包含802.1q VLAN)外包上一個8-byte VXLAN header(1:flag, 3:reserved, 3:VNI, 1:reserved)，再在其外包上UDP:8472以下的完整頭，因此亦支援IP multicasting和QinQ VLAN (功能性大勝GRE封裝)。

```
do
 local p_vxlan = Proto("vxlan","Virtual eXtended LAN");

 local f_flags = ProtoField.uint8("vxlan.flags","Flags",base.HEX)
 local f_flag_i = ProtoField.bool("vxlan.flags.i","I Flag",8,
   {"Valid VNI Tag present", "Valid VNI Tag NOT present"}, 0x08)
 local f_rsvd1 = ProtoField.uint24("vxlan.rsvd1","Reserved",
   base.HEX)
 local f_vni = ProtoField.uint24("vxlan.vni","VNI",base.HEX)
 local f_rsvd2 = ProtoField.uint8("vxlan.rsvd2","Reserved",
   base.HEX)

 p_vxlan.fields = {f_flags, f_flag_i, f_rsvd1, f_vni, f_rsvd2}

 function p_vxlan.dissector(buf, pinfo, root)

   local t = root:add(p_vxlan, buf(0,8))

   local f = t:add(f_flags, buf(0,1))
   f:add(f_flag_i, buf(0,1))

   t:add(f_rsvd1, buf(1,3))
   t:add(f_vni, buf(4,3))
   t:add(f_rsvd2, buf(7,1))

   t:append_text(", VNI: 0x" .. string.format("%x", 
      buf(4, 3):uint()))

   local eth_dis = Dissector.get("eth")
   eth_dis:call(buf(8):tvb(), pinfo, root)
 end

 local udp_encap_table = DissectorTable.get("udp.port")
 udp_encap_table:add(8472, p_vxlan)
end
```
首先，將這個新的dissector命名為vxlan1，在wireshark的filter欄位選vxlan1就可代入；接著宣告整個VXLAN header的長相，Protofield和base有很多型態可以選，都寫在wireshark根目錄下的init.lua。比較特別的是除了分割欄位外，還可以將一個欄位在細切成多個子欄位，以VXLAN來說，他要看第8 bit的值所以mask下了0x08。

整個VXLAN header是buf的0到8 byte，buf的起始點是後面的條件決定的，等等會提，並將每個欄位對應的起始位置和offset告知，當然，可以對特定位置可以新增額外註解；當8-byte VXLAN header結束後剩餘部分當作Ethernet frame的開頭，最後是設定一個條件來決定buf的起始位置，以VXLAN來說就是UDP:8472，以我來說就是IP proto 47。

因為Lua是直譯式語言，所以寫爛了再使用到的時候才會知道，把上述code存成vxlan.lua，記得要在根目錄下的init.lua最後加入`dofile("vxlan.lua")`，這樣就完成了。直接在filter欄位下vxlan1即可使用。
 
順便附上一個比較特別的範例
```
local proto_id = buf(2,2):uint()

if proto_id == 0x0806 then
	local arp_dis = Dissector.get("arp")
	arp_dis:call(buf(24):tvb(), pinfo, root)
else
	local ip_dis = Dissector.get("ip")
	ip_dis:call(buf(24):tvb(), pinfo, root)
end
```
剛有提到，GRE type是0x0800就是普通的IPinIP封裝，若是0x0806就是ARP，因此我在dissector內有加入這樣的判斷以解決當初我硬改code的窘境。



