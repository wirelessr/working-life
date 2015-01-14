# Autotools

在講autotools之前一定要先有個前言：我以前一直覺得寫一個軟體刻出一個能夠build的Makefile就好，但實際在寫程式，尤其是大程式後深覺要寫Makefile不難，但要寫一個好的Makefile很難。
 
make, make install, make uninstall, make dist, make clean, make distclean，族繁不及備載，以上都是open source必備的make操作，安裝、打包、反安裝等都一應俱全，這如果要手刻Makefile真的會想死。另外如果有cross compile的需求，指的是用不同的平台去build，例如mips, x86, ppc，就必須動態去改c compiler, c++ compile, linker，族繁不及備載，以上種種，讓我不太想去寫一個GNU Makefile，轉而傾向自動化產生Makefile。
 
也就是這章的重點，autotools，這包含了autoconf, automake, autoheader, libtool，族繁不及備載，這一系列的工具互相搭配，造就了自動產生Makefile的環境。
 