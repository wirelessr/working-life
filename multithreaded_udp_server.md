# Multithreaded UDP server

常用TCP socket開發的人都知道也都會用multithreaded或multiproccessed去maintain session

![](http://i.imgur.com/fIEXgga.jpg)

大致上的流程就是這樣，當收到一個request就pthread_create或fork出一個child去做read/write，比較常見的點是放在accept後，因為TCP socket的特性，在accept後可以拿到一個新的fd，因此child可以直接用新的fd read/write，很方便。
 
這種server-client的架構在apache server的MPM(Multi-Processing Module)中稱為prefork，另外還有兩種，分別是worker和event。值得一提的是，在apache的實作中，prefork算是最常見的，因為它的歷史最悠久，但也最為人所詬病，原因無他，當今天session數量非常龐大，那server就必須耗用大量的process去作handler，每一個process都是一筆不小的開銷，因此scalibity被限制住了。worker就是為了解決這個問題而被提出的新架構，而event是一種運用asynchronous I/O的技術。

回歸正題，正因TCP socket可以透過accept為每個session得到新的fd，因此做multi-processing的難度大大降低，但UDP就沒這麼好運了，UDP是connectionless，因此沒有session的概念，這也造成沒有accept這類的API可以提供session的維護。但multi-processing是有效提升server performance的做法，故本篇的重點在於如何在UDP server上做multi-processing。
 
扯的有點多，我先提出concept。
 
server一樣會bind在一個listen port上，而client會試圖connect，到此為止都與TCP相同；不同點在於，一旦client connect成功，馬上就可以sendto，而server則是bind完後直接要recvfrom，當server recvfrom時，根據source IP和port作為pseudosession的identifier，若是新的session，就pthread_create，並且將packet enqueue，若是已知session則直接enqueue。
 
server enqueue後透過pthread_cond_broadcast喚醒所有的child；而child則是一開始先進去pthread_cond_timedwait，收到parent送來的喚醒信號則peek queue的head，看看是不是自己的IP & port，若是，就dequeue處理，若否就繼續pthread_cond_timedwait。
 
之所以用pthread_cond_timedwait則是因為當timeout時，這條session的thread才有辦法被回收，流程圖如下

![](http://i.imgur.com/fIEXgga.jpg)


詳細的pthread library使用細節我就不提了，畢竟這隨便google就有或者請參考[POSIX Threads Programming](https://computing.llnl.gov/tutorials/pthreads/)。
 
這樣就完成一個簡易的multithreaded UDP server，其實，UDP socket可以仿TCP socket做出listen和accept的效果，不過這就留待下一節解析吧。

