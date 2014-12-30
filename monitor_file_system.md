# Monitor file system

今天的課題是監控檔案系統，當然，這還是建構在linux based的章節，windows programming我從來沒接觸過。在一個大型的軟體系統中，通常都會有許多config file存在，用來設定軟體系統的各種參數，大抵來說，linux上的open source幾乎都是daemon帶起來時順便把.conf載入，然後就不管它了。但在商業用的系統上，更多時候這個.conf是會被改動的，而且，系統必須aware它的變動。
 
直接給sample應該比較合programmer的胃口
```
#define EVENT_SIZE		(sizeof(struct inotify_event))
#define EVENT_BUF_LEN	(1024 * (EVENT_SIZE+16))
fd = inotify_init();

wd = inotify_add_watch(fd, "/tmp", IN_CLOSE_WRITE | IN_MOVE_TO);

length = read(fd, buffer, EVENT_BUF_LEN);

while( i < length)
{
	struct inotify_event *event = (struct inotify_event *) &buffer[i];
	if(event->len)
	{
		if(event->mask & IN_CLOSE_WRITE || event->mask & IN_MOVE_TO)
		{
			if(!strcmp(event->name, "XXXX"))
			{
				printf("do something you want\n");
			}
		}
	}
}
```
我想監看/tmp/XXXX的變動，並且做對應的事，程式碼如上。我用兩個option：`IN_CLOSE_WRITE`和`IN_MOVED_TO`，這樣可以應付任何形式的檔案更動，包含新建檔案、從別的地方搬過來、複製過來，甚至rename。
 
程式結束記得要close fd和wd，這是寫code的基本。



