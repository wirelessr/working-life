# FAQ

這篇將介紹一些我在使用autotools碰到的問題，因為都是小問題，各開一篇又好像怪怪的，就整合放在一起。

Q: 當在configure.ac中使用了AC_DEFINE定義一個MACRO，在source code中如何使用?    
A: #include "config.h"，就可以直接用#ifdef或#if去判斷

Q: 為什麼在build code時會跳出** undefined reference to `rpl_malloc'**?    
A: 因為autoconf在檢查malloc時找不到malloc的位置，解法只要將configure.ac內的`AC_FUNC_MALLOC`拔掉即可