# Linux timer

只要是大型的project，programmer一定會需要timer，如何評估一個好的timer我通常會用幾個item去評斷
 
- 容易實作
- 可重複使用
- 精準
- 擴充性強
- 資源占用少

這幾個目標都能達到就是一個完美的timer。
 
目前接觸了許多code，大致上可以定義出幾種timer，這篇會用少量的篇幅分析也會配上少量的code，首先先列出我看過的timer，再次強調，這邊講的是linux
 
1. alarm
2. setitimer
3. timer_create
4. pthread_cond_timedwait
5. sleep...
 

先從最簡單的開始講好了，`sleep`，還真的有人用while和sleep來做timer咧，不得不說，他真的非常容易實作，出問題的機會也少，好像也不錯擴充，如果有去算difftime，那也還滿精準的。但不知道為什麼，我就是覺得這種timer很奇怪。嚴格說起來
> data structure + algorithm = program
 
用sleep當timer這種code實在有點廢阿！真要挑毛病的話就是他必須額外多一個process/thread去等時間到，而且當要掛timer的物件很多時，執行時間有可能就超過sleep的interval了，會導致有物件執行起來的預期時間不太對勁。若是一個物件給他一個process/thread去等sleep，這就不滿足資源占用少這個條件了。
 
`alarm`這也是個很有趣的東西，alarm通常是拿來抓run-time error的，但偏偏它可以設定秒數，所以要說是個timer好像也行，做法就是寫一個SIGALRM的signal handler，然後也許可以用個全域變數去區分今天跑起來的timer要幹嘛甚至也可以用個全域flag來取消timer。
 
拜託，平常千萬不要隨意使用signal handler，alarm就讓它發揮應有的功能吧！這種timer最大的問題是，無法兩個timer同時go，一次只會有一個作用，因為signal無法reentrance。
 
`setitime`這個API是alarm的進化版
> int getitimer(int which, struct itimerval *curr_value);
 
那個which可以決定今天要發哪個signal，話是這麼說，但也只有三種選擇
 
> ITIMER_REAL -> SIGALRM    
> ITIMER_VIRTUAL -> SIGVTALRM    
> ITIMER_PROF -> SIGPROF
 
至少可以三個timer同時run了，而取消的辦法則是setitimer重call一次且把時間設成0，勉勉強強，還算OK，如果趕時間，這樣做也無可厚非啦，不過，用setitimer還不如用timer_create。 
 
`timer_create`這個API就是一和二的集大成了
> int timer_create(clockid_t clockid, struct sigevent \*sevp,                        timer_t *timerid);
 
這API的參數就豐富了，clockid可以選CPU的時脈，看是單純count user space還是kernel space或total，也可以自帶signal和signal handler進去，能用的signal是signal RT系列的，從 SIGRTMIN~ SIGRTMAX。
 
簡單放一下用法，首先先把signal handler裝起來
```
void install_handler(int sig, void (*handle)(int s))
{
	struct sigaction sa;
	sa.sa_flags = SA_SIGINFO;
	sa.sa_sigaction = handler;
	sigemptyset(&sa.sa_mask);
	if(sigaction(sig, &sa, NULL) == -1)
		perror("sigaction");
}
```
接著把timer的時間設定好，記得return的timer要存起來
```
timer_t create_timer(int sig, int sec)
{
	timer_t timerid;
	struct sigevent sev;
	struct itimerspec its;
	
	sev.sigev_notify = SIGEV_SIGNAL;
	sev.sigev_signo = sig;
	sev.sigev_value.sival_ptr = &timerid;
	if(timer_create(CLOCK_REALTIME, &sev, &timerid) == -1)
		perror("timer_create");
	
	its.it_value.tv_sec = sec;
	its.it_value.tv_nsec = 0;
	its.it_interval.tv_sec = its.it_value.tv_sec;
	its.it_interval.tv_nsec = its.it_value.tv_nsec;
	
	if(timer_settime(timerid, 0, &its, NULL) == -1)
		perror("timer_settime");
	return timerid;
}
```
之後如果要取消，就可以直接用timer_delete砍掉，這種型態的timer已經稱的上是signal型態的完全體了。非常容易實做，code也容易復用，更重要的是，絕對精準，且完全沒有overhead，唯一的缺點是，signal handler絕對要注意reentrance的限制，例如：千萬不要在signal handler內做IO或mutex，不然自己的code怎麼dead lock的你都不會知道。
 


