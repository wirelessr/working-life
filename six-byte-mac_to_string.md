# six-byte-MAC to string

既然7.1都提到了硬刻vs.查表，就來提供一下本人親身耍蠢的經歷好了。在講故事前先前情提要一下，six-byte-MAC是一種儲存MAC的方式，我們都知道，MAC其實是由xx:xx:xx:xx:xx:xx組成，也就是12個hex。因此在程式內部通常是用`char mac[6];`來放。
 
但這畢竟是給電腦看的，要轉成給人看的東西還是得倒回來變成string，一開始，我是想用sprintf(macString, "02X:02X:02X:02X:02X:02X", ......);的方式來做，但有個問題，MAC即使是F開頭也只會有兩碼，若是用sprintf，原本應該要是FA的就會變成FFFFFFFA了。因此，我花了一點時間硬刻出了轉string的功能。

```
void mac2string(unsigned char* mac,unsigned char* str)
{
	int i, y;
	char c;
	
	for(i=0, y=0; i<6; ++i) {
		if((((mac[i] >> 4) & 0x0f) >= 0x0) && (((mac[i] >> 4) & 0x0f) <= 0x9))
			c = ((mac[i] >> 4) & 0x0f) + '0';
		else
			c = ((mac[i] >> 4) & 0x0f) - 0xa + 'a';
		str[y++] = c;

		if(((mac[i] & 0x0f) >= 0x0) && ((mac[i] & 0x0f) <=0x9))
			c = (mac[i] & 0x0f) + '0';
		else
			c = (mac[i] & 0x0f) - 0xa + 'a';
		str[y++] = c;

		if(i != 5)
			str[y++] = ':';
	}
	str[y] = '\0';
}
```
很驚人，是吧！但就連作者我本人現在看了也不知道這再寫甚麼，後來我冷靜的思考了一下，其實，只要查表就好了。

```
void mac2str(char *mac, char *str)
{
	int i, y = 0;
	int c;
	char hex_to_char[16] = {'0', '1', '2', '3', '4', '5', '6', '7', '8', '9', 
							'A', 'B', 'C', 'D', 'E', 'F'};
	for(i = 0; i < 6; i++)
	{
		c = (mac[i] >> 4) & 0x0f;
		str[y++] = hex_to_char[c];
		
		c = mac[i] & 0x0f;
		str[y++] = hex_to_char[c];
		
		str[y++] = ':';
	}
	str[y-1] = '\0';
}
```
以上，就是本人親身示範耍蠢的實例。但寫程式的人常常會一頭栽進"硬刻"的狂熱中，這是你我都容易犯的錯誤，慎之，戒之。








