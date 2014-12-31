# DTLS & multithreaded UDP server part 2

我現在做的是企業無線網路設備，但嚴格說起來，我跟wifi其實一點關係都沒。要有效率的布建企業無線網路，大部分的SI都會推薦wireless controller搭配AP。AP不是我負責，我專職在controller，其上所有的protocol都由我經手，例如：VRRP, etc.，而當中最這重要的protocol是CAPWAP (RFC5415)：裡面定義了兩條channel：control plan用來管一堆AP，而data plan則是用來傳遞來自station的data。
 
詳細內容這邊就不介紹了，畢竟那個RFC看一看就有的東西，這篇的主角是control plan，在RFC中定義其所使用的protocol是DTLS (RFC4347)，如果講DTLS可能沒有甚麼人知道，但如果說是Datagram TLS，大部分搞網路的人應該都聽過，顧名思義就是用UDP去作TLS的handshake。這篇的重點不是如何handshake，而在於如何利用DTLS的方式去完成multithreaded UDP server。
 
在6.2有提到，UDP和TCP構造上的不同導致multi-thread很難做，既然如此，我們就利用TCP的TLS機制去完成類似TCP的UDP server，用的套件是openssl。
 
上次畫過的圖就不再畫了，直接來個新版的示意圖，詳細的code就不放了，因為code有點多，從SSL setup到certificate install再到BIO setting，很複雜的。這邊只提供流程和key point

![](http://i.imgur.com/I50n5Cw.jpg)

這示意圖其實看TCP根本就超像了，唯一一點小差別是pthread\_create出來後還必須自己開一個socket fd去bind原來的server addr，所以一定得setsockopt `SO_REUSEADDR` & `SO_REUSEPORT`，原因在於TCP的data fd是透過accept做出來的，但SSL_accept並沒有做出fd的功能。
 
`SSL_accept`的目的是讓SSL\*、BIO\*和fd三者取得關聯，所以fd得自己建，且`SSL_read`和`SSL_write`吃的參數並不是fd，而是SSL*。
 
所幸，那個fd依然可以透過select去排多工，如果要一條thread建數個session也是可以辦到的。至於`BIO_CTRL_DGRAM_SET_RECV_TIMEOUT`可以設timeout，但我沒成功過，所以我都是用select去做timeout，也許有人成功可以告訴我箇中奧妙。






