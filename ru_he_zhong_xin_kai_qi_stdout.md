# 如何重新開啟stdout?

生活中常常會需要把stdout/stderr給關掉，以避免程式沒寫好然後莫名跳了一堆錯誤訊息造成客戶困擾，當今天關閉了如何重新開啟呢?
 
會有這樣的需求其實也很有趣，因為fork出一個process會繼承parent的一切attribute，包含stdout/stderr的fd: 1  & 2，因此若是parrent想關但child想開就會有這樣的需要
 
範例：
> null_fd = open("/dev/null", O_WRONLY);   
> dup2(null_fd, 1);
 
首先parent把stdout導到垃圾區去，這樣就不會有東西印出來，或者call了daemon這個API，接著在child端要開回來，第一步是先把fd: 1給關閉，然後趕快產生一個新的fd。
 
根據linux的特性，fd會從最小的開始分配，所以1被關了下一個給出去的就是1
> close(1);    
> fd = open("/dev/tty", O_WRONLY);   
> stdout = fdopen(fd, "w");
 
這樣就可以了，不過我是開在/dev/tty，這是unix/linux的default terminal，上面的code跑完後就可以正常printf了
> printf("Hello world!\n");
> > Hello world!