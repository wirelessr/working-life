# Config loader / command parser

Config loader: 讀取一份config file，裡面有許多"terms"，例如：
> AAA=1   
> BBB=2   
> CCC=3   

在linux programming中很常會碰到類似的config，這都會需要一個loader將其讀到程式變數中。
 
Command parser: 讀取一條來自外部的command，並根據其signature做特定處理，例如：
> tcpdump -s 0 -i eth0 port 5246 and host 192.168.1.1

以上例來說，tcpdump就具備有一個command parser去處理-i -s 之類的opt和後面一連串的參數。
 
這兩個東西其實很類似，就是對一堆字串做比對，並且做出對應的行為。這時候，大部份人的程式就會變成下面這樣
 ```
if(!strcmp(argv[1], "AAA"))
  /* do something */
else if(!strcmp(argv[1], "BBB"))
  /* do something */
else if(!strcmp(argv[1], "CCC"))
...
 ```
 
說實在話，這並不是不行，就功能面來說也完全沒問題，但當今天signature超過一百會發生甚麼事？ (請試想)
 
答案是，很可怕，不要問。
 
那是大部分寫程式的人所不想去maintain的夢靨。這篇文章主要是要告訴你，如何避掉這個問題。也許我直接放code會比較快
```
struct {
	char *key;
	int event_type;
	enum {
		WAIT,
		NO_WAIT
	} event_class;
	enum {
		ARG,
		HANDLE
	} value_type;
	union {
		int (*handle)(int argc, char **argv);
		int idx;
	};
	int (*process)(int socket_fd, int argc, char **argv);
}
config_table[] = {
	{"manual-add",	EVENT_MANUAL_ADD,	NO_WAIT,	ARG,	{.idx = 2},						NULL},
	{"add",			EVENT_ADD_AP,			WAIT,		ARG,	{.idx = 3},						ap_add},
	{"kick",		EVENT_DEL_AP,			NO_WAIT,	ARG,	{.idx = 3},						NULL},
	{"dbg-flag",	EVENT_SET_DBG_FLAG,		NO_WAIT,	HANDLE,	{.handle = dbg_flag_handle},	NULL},
	{"show",		EVENT_GET_AP,			WAIT,		ARG,	{.idx = 3},						ap_show}, //a.out show ap MAC
	{"dump",		EVENT_GET_AP_TO_FILE,	WAIT,		ARG,	{.idx = 3},						ap_dump}, //a.out dump ap all
	{"list",		EVENT_GET_STA_INFO_ALL,	WAIT,		ARG,	{.idx = 3},						sta_list}, //a.out list sta all
	{"reboot",		EVENT_REBOOT_AP,			NO_WAIT,	ARG,	{.idx = 3},						NULL},
	{NULL, 			0,								NO_WAIT,	ARG,	{NULL},							NULL} // end
},
	*conf_item = config_table;
	```
像上面，用一張表來記錄所有的input pattern和其對應的行為，遠比一堆的if-else方便許多，使用上也很簡單，更重要的是，後續接手的人很容易擴充功能，只需要填表就好。這個frmework最核心的部分是查表的code

```
int load_config(int argc, char **argv)
{
	char *key;
	int ret = SUCCESS;

	for (conf_item = config_table;
			conf_item->key;
			++conf_item)
	{
		if (strcmp(conf_item->key, argv[1]) != 0)
			continue;
		if (conf_item->value_type == ARG) {
			sprintf(event, "%s", argv[conf_item->idx]);
		}
		else {
			ret = conf_item->handle(argc, argv);
		}

		if(ret != SUCCESS)
			return ret;

		if(conf_item->event_class == NO_WAIT)
		{
			if(ipcSendEventNoWait(conf_item->event_type, event , strlen(event)) < 0)
			{
				return ERR_DEAD;
			}
		}
		else /* WAIT */
		{
			int socket_fd = 0;
			char entry[7000];
			int ret = 0;

			if((socket_fd = ipcSendEventWait(conf_item->event_type, event , strlen(event))) >= 0)
			{
				return conf_item->process(socket_fd, argc, argv);
			}
			else
			{
				return ERR_DEAD;
			}
		}
		return SUCCESS;
	}
	return ERR_NOT_FOUND;
}
```
用一個for迴圈去traverse就結束了，皆大歡喜的結局，不是嗎？ 實作的人也方便，後續維護的人也輕鬆，需要擴充的人也自在，正所謂
 
> data structure + algorithm = program
 
我們寫程式常常會犯一個錯，就是直接硬刻出想要的功能，但實際上可能用了最蠢的方式，換個角度思考一下，就可以找到更理想的做法。


	
	
	
	
	






