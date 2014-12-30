# C container

我在研究所專攻C++，開始工作時寫的都是C，漸漸思考也僵化了，這並不是說C不好，而是C在程式語言的基礎上有太多落後的地方：
 
- 沒有物件，只好用struct和function pointer替代
- 沒有template，只好用MACRO硬套
- 沒有inheritance，硬幹也行啦，只是effort超高
- 沒有polymorphism，也是可以硬幹，這effort高到突破天際了
- 沒有override operator，這還真無解
 

所以C語言在邏輯思考上，沒有辦法做到更完全的抽象化，也少了很多天馬行空。C++最讓我懷念的地方是他的STL中有一系列很棒的container，其中我最愛的是map：為一個entry找一個key，就可以透過key直接反查entry，非常實用，而它底層使用的data structure是紅黑樹，所以效率也滿高的。
 
我試圖在C語言中找到一個替代方案，所幸C語言中也有類似的container，但功能還是有點差距，例如，我在物件中直接override operator就可以做到map需要的compare function，但在C語言中就必須自己實作一個function pointer當參數丟進API，這在使用的方便性上就差滿多的。
 
以下要介紹的是C語言提供的binary tree container，直接給範例
```
intStrMap *a = malloc(sizeof(intStrMap));
a->key = strdup("two");
a->val = 2;
tsearch(a, &root, compare); /* insert */

intStrMap *b = malloc(sizeof(intStrMap));
b->key = strdup("three");
b->val = 3;
tsearch(b, &root, compare); /* insert */

intStrMap *find_a = malloc(sizeof(intStrMap));
find_a->key = strdup("two");
void *r = tfind(find_a, &root, compare); /* find */
printf("val = %d\n", (*(intStrMap**)r)->val);
```
這裡有兩個API：tsearch (insert)、tfind (search)，這種命名實在要，命明明是search卻做insert的事，類似的API還有linear container：lsearch和lfind。另外一個比較重要的是要實作compare function
```
int compare(const void *l, const void *r)
{
	return strcmp(((intStrMap*)l)->key, ((intStrMap*)r)->key);
}
```
透過這種方式，C語言也算是提供了不錯的container，至於效率，出乎意料的與map差不多，主要是C語言本身的效能就比C++好些，而我的測試data沒用的太複雜，因此有這種結果也不算意外。




