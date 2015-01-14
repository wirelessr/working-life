# Autotools: conditional target


我就不去介紹如何寫一個sample了，那個應該隨便找都有，不外乎先用autoscanf產生一個configure.scan，然後改名成configure.ac，接著寫個Makefile.am，然後用autoreconf -i -f去產生configure，接著透過configure去產生Makefile.in和Makefile。
 
本篇要介紹的是Makefile中常見的conditional strunction要怎麼在autotools中實作，有四個檔案
1. main.c: call helloMsg();
2. hello.h: define helloMsg();
3. world.c: implement helloMsg() to printf hello world!
4. linuxc.c: implement helloMsg() to printf hello linux!
 

其實也就是動態決定今天printf要是hello world還是hello linux，這種動態抽換source或lib的技巧在C語言中很常見，直接講重點
 
在configure.ac中加入
```
AC_ARG_WITH([string], [  print hello string], [hello_string=${withval}])    
AM_CONDITIONAL([LINUXC], [test "x${hello_string}" = xlinuxc])
 ```
然後在Makefile.am中加入
```
if LINUXC
hello_SOURCES += linuxc.c
else
hello_SOURCES += world.c
endif
 ```
使用方法很單純，若是`./confiure`甚麼參數都沒帶，就會產生hello world的Makefile；若是`./configure --with-string=linuxc`，就會產生hello linuxc的Makefile。
 
這種做法很有趣，其實也還有另外一種做法是在configure.ac用`AC_DEFINE([LINUXC])`，而code則是隔在source file內，用`#ifdef LINUXC`去隔做法。

但是熟悉open source的人都會知道，第二種作法不推薦，因為在source file內有太多feature面的compile flag會造成code很難maintain，應該將code區隔成不同的source file來定義不同的實作，這才是正確作法。




