# Autotools: conditional target

在講autotools之前一定要先有個前言：我以前一直覺得寫一個軟體刻出一個能夠build的Makefile就好，但實際在寫程式，尤其是大程式後深覺要寫Makefile不難，但要寫一個好的Makefile很難。
 
make, make install, make uninstall, make dist, make clean, make distclean，族繁不及備載，以上都是open source必備的make操作，安裝、打包、反安裝等都一應俱全，這如果要手刻Makefile真的會想死。另外如果有cross compile的需求，指的是用不同的平台去build，例如mips, x86, ppc，就必須動態去改c compiler, c++ compile, linker，族繁不及備載，以上種種，讓我不太想去寫一個GNU Makefile，轉而傾向自動化產生Makefile。
 
也就是今天的重點，autotools，這包含了autoconf, automake, autoheader, libtool，族繁不及備載，這一系列的工具互相搭配，造就了自動產生Makefile的環境。
 
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




