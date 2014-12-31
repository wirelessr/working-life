# 鎖定thread在特定CPU

這篇其實是multithreaded UDP server的續章，為什麼需要『鎖定thread在特定CPU』？

因為在Linux上，每一個thread其實就是一個獨立的process，因此在context switch上，一條thread一定得要佔一個人頭，但multicore的環境中，了不起就N個核同時一起跑，開越多條thread並不會越快。另外，若是任由kernel自行去分配thread到空閒CPU，常會導致I/O time去搶奪CPU time。
 
這也就是這篇提出來最主要的原因，作法其實並不困難。首先，必須要先知道系統有幾個核
 
> procs = (int)sysconf( _SC_NPROCESSORS_ONLN );
 
接著就是分配的工作
``` 
cpu_set_t set;
CPU_ZERO( &set );
CPU_SET( proc_num, &set );
sched_setaffinity( gettid(), sizeof( cpu_set_t ), &set );
 ```
補充一點，要compile過通常要加上
 
> \#define _GNU_SOURCE
 
再另外說明一下好了，一定得要gettid，因為所有的thread共用同一個pid，做法是
 
> syscall( __NR_gettid )

or

> syscall(SYS_gettid);
 
這些code的片段組起來就可以做到。

