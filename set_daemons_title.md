# Set daemon's title

最近看open source時看到一個滿不錯的手法
 
> root@prowlan-Veriton-M4610:~/Documents/env# ps aux | grep 27086    
> > root     27086  0.0  0.0   2136   280 pts/0    S    11:52   0:00 **./a.out: test**      

> root@prowlan-Veriton-M4610:~/Documents/env# ps aux | grep 27086    
> > root     27086  0.0  0.0   2136   280 pts/0    S    11:52   0:00 **./a.out: hello**    

> root@prowlan-Veriton-M4610:~/Documents/env# ps aux | grep 27086    
> > root     27086  0.0  0.0   2136   280 pts/0    S    11:52   0:00 **./a.out: lalalalala**    
 
可以使ps aux後面的command做出想要吐出來的字串，詳細的做法我就不贅述了，畢竟code也不是太難懂。在main function去init title，然後想改title的地方call set title即可。
```
char **environ;
char *title_start;
char *title_end;
int title_size;

void init_title(int argc, char *argv[], char *envp[], char *name)
{
	int i;
	
	for(i = 0; envp[i]; i++);
	
	environ = (char **)malloc(sizeof(char *) * (i + 1));
	
	for(i = 0; envp[i]; i++)
		environ[i] = strdup(envp[i]);
	environ[i] = NULL;
	
	title_start = argv[0];
	
	for(i = 0; i < argc; i++)
		if(!i || title_end == argv[i])
			title_end = argv[i] + strlen(argv[i]) + 1;
	
	for(i = 0; envp[i]; i++)
		if(title_end == envp[i])
			title_end = envp[i] + strlen(envp[i]) + 1;
	
	strcpy(title_start, name);
	title_start += strlen(name);
	title_size = title_end - title_start;
}

void set_title(const char *fmt, ...)
{
	char buf[255];
	va_list ap;
	
	memset(title_start, 0, title_size);
	
	va_start(ap, fmt);
	vsprintf(buf, fmt, ap);
	va_end(ap);
	
	if(strlen(buf) > title_size - 1)
		buf[title_size - 1] = '\0';
	
	strcat(title_start, buf);
}
```
這樣的好處是甚麼？
 
實務上，RD很難在沒有任何log或track的daemon上進行debug (WHY NOT logging??? 並不是每台embaded的file system都可以額外騰出空間去放log)，因此run-time的debug通常靠的是strace，更了不起的是gdb (gdb其實也有他的難處，因為attach上去daemon就等同於葛屁了，真正的"run-time" bug會miss掉)，而strace是一堆system call的組成，當然如果RD的本領夠高超，這其實很夠了，但我相信並不是每個人都這麼強大。所以，readable的context能夠更快幫助RD進入狀況。
 
這功能強大的地方在於，不需要額外空間額外記憶體就可以track code flow (抓deadlock超實用)。
 
一的小限制是，他會改到environment variable內的PATH，所以如果code裡面有用system或exec系列且使用相對路徑的話，會！失！效！但用相對路徑去做事這比較會發生在不成熟的RD身上，總之請小心這唯一限制。



