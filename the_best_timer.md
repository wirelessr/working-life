# The best timer

我很少把話說這麼死，但我的確發現了Linux中最強的timer API。在4.6中提到的timer都有個共通的問題，那就是scalibilty實在不太夠，在龐大的軟體架構中，timer的物件掛到上百個其實是家常便飯的，而timer_create有兩個重大問題
 
1. reentrance：這個單字不斷被提及，最主要的問題就是signal-based的timer會不可中斷，若是signal handler裡面在做I/O甚至access shared data，這問題就大條了，常常會讓process hang到死。
 
2. scalibility：這其實才是最主要的問題，timer_create是使用RT系列的signal，從SIGRTMIN到SIGRTMAX，了不起就一百個(通常沒這麼多，x86上是34-64共31個)
 
因此，這次要介紹的timer必須具備幾個條件
 
1. cost低
2. 超大上限
3. 容易管理和使用
 
而最後從淘汰賽存留下來的是`timerfd_create`, `timerfd_settime`, `timerfd_gettime`，應該有似曾相識的感覺吧，其實把fd兩個字拿掉，跟timer_create系列也沒差多少。
 
但fd這兩個字非常讚，timer\_create是透過註冊signal來trigger，time's up時就是signal handler hold，而`timerfd_create`則是註冊一個fd，而time's up時則是使fd變成readable。
 
這有甚麼好處? 好處可多了!!!
 
首先，fd的數量可以非常非常非常多，另外一個則是readable這件事可以驅動select或poll，也就是說，在大型的軟體架構下，只需要在其中去排poll或select就可以搞定無論是timer還是一般的event，這使得code變得超級乾淨清爽簡潔的。
 
用法就不贅述了，跟timer_create其實沒差多少。
 
給個結論好了，以後我只要需要用到timer，我一定只會使用timerfd_create。


