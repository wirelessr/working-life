# Ubuntu 14.04 Wi-Fi connection failed!

問題是這樣的
 
有一個SSID的加密用的是Enterprise WPA2，意即是需要透過802.1X認證
，802.1X可以選擇很多種加密認證方式，TTLS、PEAP諸如此類，也可以指定certificate。我們公司的設定是沒有certificate的，所以我在那欄選了none。
 
結果發現，完全連不上AP，自己跳下抓封包抓log才知道在hostapd端認證沒過，直接就被block掉。原因出在ubuntu 14.04的network manager有bug，即使今天certificate欄位選none，他default還是要跑certificate認證。
 
解法是去/etc/NetworkManager/system-connections這個path下找該SSID的設定檔，把system-ca-certs=true這行給幹掉，這樣就一切正常。

本來這篇是想附個封包和附個hostapd的log的，但時日已久，已經找不到了，而且ubuntu 14.04.2已經修掉這個問題，所以我也無法重現，以後有機會再補上吧。