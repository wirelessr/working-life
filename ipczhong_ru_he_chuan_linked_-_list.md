# IPC中如何傳linked-list?

一個很有趣的問題，主要是被客戶報bug要解，然後發現問題是卡在IPC時有個linked-list傳不完，導致daemon就hang在那。
 
當process A要向process B要資料，這資料的DS是linked-list時作法有許多種，以我碰到的例子是用domain socket，然後一個node一個node的要。這樣有個很嚴重的問題，首先是B process裡面的資料不會固定下來等A要完，所以B的linked-list會變來變去，若是A根本不知道何時該停且code的邏輯又不好，A就會要不到或要不完而hang住。
 
其實要解也有很多解法，最迅速的是B就snapshot一份出來讓A去拿，只是，怎麼給也很有學問，一個node一個node要其實很浪費資源，會導致兩個process都有多次的IO，但一次allocate一大塊memory去塞又很奢侈，在embaded上根本不可能這樣做。
 
我的解法是B就將整份linked-list的內容寫到一個file上然後把file name回傳給A，A就直接去檔案抓就好，用完自己刪掉，檔案內容就是把整個linked-list上的struct用fwrite的方式依序整塊寫入file。然後A就用fread一塊一塊拿出來填入自己的資料結構內。
 
若是這樣就還有另外一個問題要解決，如何產生一個unique的file name？若不是unique可能就會蓋來蓋去。
 
我本來的想法是用linux的uuid去產生一個unique的4byte integer然後把它轉成文字變成file name，但後來想想這樣真智障，因為uuid的演算法有點複雜，對一個tmp file這樣搞實在很浪費時間，後來有找到另外一個solution，簡直超神。直接給例子：
```
char fileNameTemp[MAX] = "temp_name_XXXXXX";
int fd = mkstemp(fileNameTemp);
FILE *fp = fdopen(fd);
fprintf(fp, "Hello world!\n");
fclose(fp);
``` 
神奇用法，重點是那個`XXXXXX`，一定要六個，多一少一都不行，他是tmp file name的template。



