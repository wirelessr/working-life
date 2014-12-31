# Garbage collection in C

Garbage collection (GC) 是一種在高階語言中常見的技術，例如JAVA，當一個allocate的memory不再被refererce時就會被系統自動回收掉，美其名是系統回收，但其實是user space寫出來的maitain機制做的，並不是真的被kernel回收。
 
而C/C++沒有這樣的機制，所以必須透過explicit的方式操作memory，常見的就是malloc/free、new/delete，相信寫過C/C++的都會同意這其實是一件很要命的事情，任何條件分支都必須要確實地注意回收記憶體，不然就是leak。
 
C語言難度已經很高了，但C++又更高，因為C++具有exception的機制，有時候超出你的預期就跳出了。
 
今天要介紹一個不錯的library使得C/C++亦具備GC：libgc.a (libgc.so)，在ubuntu上只要`sudo apt-get install libgc-dev`就可以裝起來了。直接給個範例：
```
#include <stdio.h>
#include <string.h>
#include <stdlib.h>

#define LOOP_TIMES 1000000
#define NODE_SIZE 1000

#ifndef GGGG
#define MALLOC malloc
#else
#define MALLOC GC_malloc
#endif

int main()
{
	int i;
	
	for(i = 0; i < LOOP_TIMES; i++)
	{
		char *garbage = MALLOC(NODE_SIZE);
	}
	printf("cost: %d bytes\n", NODE_SIZE * LOOP_TIMES);
	
	sleep(100);
	
	return 0;
}
```

如果直接跑這個程式沒有透過GGGG這個compile flag就會在ps aux看到以下
 
> %MEM 24.1   a.out
 
若是用`gcc sample.c -DGGGG -lgc`，這樣去做就根本不會看到佔用memory，1%都沒有！當然，libgc如果真這麼萬能，那早就沒有人用malloc/free了，他也是有缺點的，欲知詳情請看
[Pros and Cons](http://www.linuxjournal.com/article/6679?page=0,3)。



