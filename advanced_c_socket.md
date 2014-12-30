# Advanced C Socket

常見的socket使用方式幾乎都是開一個`SOCK_DGRAM`或`SOCK_STREAM`，接著拿來bind, listen, connect, accept，之後就是sendto, recvfrom或send, recv，很少有人會去了解背後的意義，也少有人去了解如何從sockopt獲得更多的幫助。

在advanced C socket這章，主要會去介紹一些socket的進階用法。如何利用sockopt與網路功能做更多整合，是本章著重的重點之一。