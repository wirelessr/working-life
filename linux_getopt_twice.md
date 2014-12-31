# Linux getopt twice

TLV是描述內容最直覺也最容易實作的方式，事實上，若是T和V都是string的話，L其實就不那麼重要了，因為strlen可以輕鬆拿到L，更有甚者，Linux提供了getopt系列的API讓programmer更容易實做T&V。
 
這篇要講的故事是getopt系列的潛在問題。
 
我們都知道，getopt最常被使用的情境是從main當中取得argc和argv後去做一次性的parsing，但要讓getopt成為產生T&V的framework就勢必要call很多次，按照慣例先給個例子
```
void parse(int argc, char *argv[])
{
    int flags, opt;
    int nsecs, tfnd;

    nsecs = 0;
    tfnd = 0;
    flags = 0;
    while ((opt = getopt(argc, argv, "nt:")) != -1) {
        switch (opt) {
            case 'n':
                flags = 1;
                break;
            case 't':
                nsecs = atoi(optarg);
                tfnd = 1;
                break;
            default: /* '?' */
                fprintf(stderr, "Usage: %s [-t nsecs] [-n] name\n",
                        argv[0]);
                return;
        }
    }

    printf("flags=%d; tfnd=%d; optind=%d\n", flags, tfnd, optind);

    if (optind >= argc) {
        fprintf(stderr, "Expected argument after options\n");
        return;
    }

    printf("name argument = %s\n", argv[optind]);
}
```
這是我從man 3 getopt抄出來的範例，稍微小改一下讓他變成一個void function，專門用來讀-n和-t。用這個framewaork的main長這樣
```
int main(int argc, char *argv[])
{       
    parse(argc, argv);
}
```
若是我call
> king@king-VirtualBox:~$ ./a.out -n -t 100 ooo

會得到這樣的output
> > flags=1; tfnd=1; optind=4    
> > name argument = ooo

那麼，要是main長的像這樣
```
int main(int argc, char *argv[])
{       
    parse(argc, argv);
    parse(argc, argv);
}
```
請問output會長怎樣? 重複兩次嗎?
 
我會這樣問就一定沒這麼簡單，這output出乎意料之外的奇怪
> king@king-VirtualBox:~$ ./a.out -n -t 100 ooo   
> > flags=1; tfnd=1; optind=4   
> > name argument = ooo   
> > flags=0; tfnd=0; optind=4    
> > name argument = ooo   
 
不是重複兩次，有些相同卻也有些不同，這問題讓我看了一段時間。後來發現了man 3 getopt中有提到：若是include了unistd.h這個header file (必須要，不然沒有getopt)，會連帶多了幾個global variable
 
> extern char *optarg;          
> extern int optind, opterr, optopt;
 
而這也是getopt的實做之一，問題就出在當今天call了getopt一次，這些變數都會被使用，而且指到最後處理的地方，下一次再call getopt，就不會拿到正確parsing的結果。因此在parse()這個function內第一件事就是要加上
 
> optind = 0;
 
如此才能夠正確的初始化整個getopt的運作，後續不管parse call幾次都沒有問題了。使用glibc一定要很小心這些眉角，不然踩到洞也不知道。
 
