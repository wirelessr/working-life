# Expect妙用無窮

情境是，我必須用ssh連線遠端的設備，然後下一些指令後收集結果，並且把該設備reboot。過一段時間後，重新連線、收集結果再reboot
 
這件事很簡單，但手動操作實在很蠢也很浪費時間。因為要做的事不多，所以寫成script應該很容易吧？大錯特錯！
 
在一開始就會大卡關：script怎麼幫你的ssh連線輸入密碼？    
當`ssh –l user@host`這指令一下，終端機的控制權就被ssh的process搶走，亦即是，你即使把密碼寫在script中，script也不會幫你執行。
 
直到控制權從ssh交還時，script才能夠繼續運行。有兩個解法，一種是基於ssh的公鑰/私鑰機制來解決，使ssh不需要詢問密碼即可連線，但這不是我這邊的解法，因為我要連線的設備無法直接控制檔案系統。也就是說，我無法把私鑰硬塞給它強迫連線。
 
因此，我採用第二種解法：透過***expect***這軟體，在Ubuntu中並沒有內建，請：
> sudo apt-get install expect
     
做法其實很單純
> expect -c ‘spawn ssh user@host ; expect assword ; send “password\n” ; expect “>” ; send “commands\n reboot\n” ; interact’    

簡單敘述一下用法，spawn後面接你要執行的第一條指令，而expect後面接可能會讀到的一行資訊，讀到後會呼叫後面的send把要輸入的東西送進去。
 
也就是說，腳本會被expect預先設計好，當ssh搶走終端機的控制權時，也可以透過預先設定的腳本進行動作。
 
expect可以運用在很多地方：    
1. 批次登設備
2. 定時下載FTP的東西
3. 批次用 "vi" i改一堆文檔
4. etc.    

其他請發揮想像力，反正expect是一個非常powerful的軟體。
可以搭配很多種不一樣的指令組合做出很多變化