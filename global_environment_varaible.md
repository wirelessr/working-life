# Global Environment Varaible

這篇文章要介紹一個非常重要的手法，其中牽扯到一些計組，如果不懂bss, data, section, etc.的人請自行先去惡補。另外，本篇手法我確信Linux的GNU GCC一定可行，其他compiler或platform有興趣的人可以自行嘗試。

(據GCC的說法是可跨平台啦，因為我沒試過所以我保持疑問態度)

###故事背景

在一個很大的程式當中，常常會散落許多的#define在各個c檔或h檔，而且幾乎都與platform或OS相關也都是寫死的(當然，compiler的preprocessor一定是寫死的)。例如：
>　#define PROC_NET_VLAN "/proc/net/vlan/config"

然後在某處可能會去`fopen(PROC_NET_VLAN, "r");`

諸如此類的寫法一定散的到處都是。此時請考量到一個case，若在正常的linux上是放/proc/net/vlan/config沒錯，但別的distribution放在別的地方呢？難道在compile的時候還必須要`-DPROC_NET_VLAN="/proc/net/fake_vlan/config"`嗎？

這不make sense嘛，為什麼我不能compile出一包code卻可以同時放在不同的OS上跑？

答案是可行的，而這也是本篇要介紹的做法。一樣先給範例
```
register_string_initenv(ABCD, "test abcd");

int main()
{
    do_initenv_initialization();

    printf("env's ABCD = %s\n", ABCD);

    return 0;
}
```
在範例中我宣告一個ABCD為test abcd，就像是以往的`#define ABCD "test abcd"`，而我在main中直接去print ABCD的值，你猜怎麼著？

> ./a.out    
> > env's ABCD = test abcd

> ABCD=xyz ./a.out    
> > env's ABCD = xyz  

> export ABCD=18av    
> ./a.out    
> > env's ABCD = 18av    

我可以透過給定環境變數的值而改變當初的定義，關鍵就是`do_initenv_initialization`和`register_string_initenv`的實作，先講register_string_initenv
```
#define __str_initenv__decl \
    __attribute__ ((unused, __section__ ("str_initenv")))

typedef struct {
    char **store;
    char *os_env_name;
} __str_initenv__rec;

#define register_string_initenv(var, def) \
    char * var = def; \
    __str_initenv__rec __str_initenv_##var __str_initenv__decl = \
    { &var, #var };
```
這邊我定義了一個struct叫做`__str_initenv__rec`，用來存放環境變數的名字和值；一旦使用了register_string_initenv，會產生兩個global variable，一個是定義的`ABCD`，另外一個是`__str_initenv_ABCD`，分別放在data和section內，其中__str_initenv_ABCD已被初始化成給定的預設值。用objdump看起來會是
> objdump -t a.out | grep ABCD
> > 0804a01c g     O .data  00000004              ABCD    
> > 0804a020 g     O str_initenv    00000008              __str_initenv_ABCD

接下來是do_initenv_initialization
```
extern void *__start_str_initenv, *__stop_str_initenv;

static inline void
do_initenv_initialization(void)
{
    __str_initenv__rec *str;

    /* string initenv */
    str = (__str_initenv__rec *) &__start_str_initenv;

    do {
        char *os_env;

        os_env = getenv(str->os_env_name);
        if (os_env) {
            *(str->store) = strdup(os_env);
        }
    } while (++str < (__str_initenv__rec *) &__stop_str_initenv);
}
```
特別要提的是，當使用了`__section__`，GCC會送你兩個pointer分別指向你產生的section的開始和結束，就是上面用到的`void *__start_str_initenv` & ` *__stop_str_initenv`。do_initenv_initialization沒甚麼好講的，就把所有註冊的環境變數掃一次，把值重新填回section內，之後用到的人就會抓到環境變數內real的值。

實際使用上，我建議把我上面寫的code放在一個header file，然後把所有用register_string_initenv的點都集中到另一個header file，並且確保在一個make target內只會include到一次 (不然會duplicated declaration)，在需要使用到的c檔用extern的方式去取，這樣就能把compile的相依性降到最低。





