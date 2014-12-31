# Atomic variable in user space

Atomic是一種同步機制，不需要使用explicit的鎖也可以做到變數之間的同步。kernel裡面有直接可以用的型別atomic_t。
 
在user space其實gcc從4.1.2就有提供built-in的API，型別可以support int8 … int64，給兩個prototype
> type \_\_sync\_fetch\_and\_add (type \*ptr, type value, ...)    
> type \_\_sync\_fetch\_and\_sub (type \*ptr, type value, ...)    
 
其實還有很多啦，但常用的就是加和減。範例如下
> int val = 0;    
> __syn_fetch_and_add(&val, 1);
 
這是一種輕量級的同步機制，如果只是對一個變數去做同步實在不需要用一塊critical section去保護他，在實測上，這種built-in的API效能比pthread_mutex_t好的多多了。
 
另外補充一點，提到atomic通常也會提到memory barrier，在compiler做編譯時有可能因為最佳化而改變了assembly的順序，若是這件事至關重大，你不希望compiler給你亂搞，就需要用到memory barrier的技術，gcc同樣提供了一個built-in的API
> __sync_synchronize()
 
在你希望確保順序性的地方用力給他call下去就對了。