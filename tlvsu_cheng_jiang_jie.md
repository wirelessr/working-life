# TLV速成講解

Network programming時常常會聽到TLV，這是Type Length Value的簡稱，也是描述資料最簡單又最清楚的形式。
 
 
這邊提供一個很不錯的寫法：
 ```
struct tlvs
{    
    int t;    
    int l;    
    char v[0];
};
 ```
 
首先定義一個結構用來存放TLV，這邊的v用了一點小技巧，也就是v[0]。
```
Q: 請問sizeof(struct tlvs)是多少?
A: 8 bytes (x86)，最後的v是沒有長度的
``` 
 
這有甚麼好處留待晚點說，先給一段使用的範例
``` 
int add_tlv(char *dst, enum TLV_TYPE type, int len, char *value);
void handle_tlv(char *data, int size);
 
 
offset += add_tlv(buf+offset, TYPE_0, strlen(STR1), STR1);   
offset += add_tlv(buf+offset, TYPE_1, strlen(STR2), STR2);   
offset += add_tlv(buf+offset, TYPE_2, strlen(STR3), STR3);
 
 
handle_tlv(buf, offset);
``` 
 
用法就是這麼單純，allocate一塊buf給定開始填值的位置和要填的TLV即可；至於handle_tlv則是會根據解出來的T是甚麼，把L和V丟入註冊的handle function，例如：
```
typedef void (*type_foo)(char *data, int len);
type_foo foo[] =
{    
    type0_bar,    
    type1_bar,    
    type2_bar,
};
 ```
 
重頭戲是`add_tlv`和`handle_tlv`的implementation
 
``` 
int add_tlv(char *dst, enum TLV_TYPE type, int len, char *value)
{   
    struct tlvs *tlv = (struct tlvs *)dst;
 
    tlv->t = type;   
    tlv->l = len;   
    memcpy(tlv->v, value, len);
 
    return (sizeof(struct tlvs)+len);
} 
``` 
剛提到，sizeof(struct tlvs)是8，因此add_tlv一次會填入的size是`sizeof(struct tlvs)+len`，若struct tlvs內的v不是v[0]而是v[1]或char *，那麼就還要扣掉一個pointer的長度，code會變得很醜。
 
 
 ```
void handle_tlv(char *data, int size)
{   
    int offset = 0;
 
    while(size)    {       
        struct tlvs *tlv = (struct tlvs *)(data+offset);       
        int data_len = sizeof(struct tlvs) + tlv->l;
 
        foo[tlv->t](tlv->v, tlv->l);
 
        offset += data_len;
 
        size -= data_len;   
    }
}
 ```
 
handle_tlv的作法也是類似的，只是方向相反。

這篇純粹紀錄一下v[0]這個必殺大招，若是要將範例用在network上還要記得ntoh和hton。


