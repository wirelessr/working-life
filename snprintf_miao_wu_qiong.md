# snprintf 妙無窮

在寫程式做LOG的時候，常是建立一個字串陣列，依序把內容放進去
例如封包數量統計：
> Tx: 978978 packets   
> Rx: 787878 packets   


C語言中沒有像是C++的STL string一樣方便的container，我以前碰到又是數字又有文字的東西其實很苦惱，常常用笨方法來解決：利用`atoi` & `strcat`來做到C++ string `<<` 的效果。
 
後來發現，sprintf 更好用，就像printf一樣，只是把標準輸出導入字元陣列。嘿，還滿好用的，只是問題伴隨而來，如果存的太忘情，會有overflow的issue，能被compiler抓出來的問題都是小問題，就怕這種run-time的error，超難debug，而且如果是連續的記憶區塊，改到別人的值還不會察覺。
 
這時snprintf 就應運而生
> int snprintf(char \*str, size_t size, const char * restrict format, ...)

他可以限制字串的長度，避免overflow。標準的用法是
> snprintf(str, sizeof(str), "aaaaaaaaaaaaaaaa");

請務必記得長度的參數要套用sizeof，一來可以避免寫死二來是有效避免overflow，而回傳的int是預計存入的長度，例如我上面的一串a，回傳的值就會是16。
 
前一陣子有看到一種做法可以用snprintf 達到run-time調整字串陣列長度的效果
```
char* str;
int size = snprintf( NULL, 0, "aaaaaaaaaaaaa");
str = (char*)malloc(size++);
snprintf( str, size, "aaaaaaaaaaaaa" );
```
這作法好不好用就見仁見智，我是會使用strlen啦，這樣比較直覺。另外值得一提的是，與其用strncpy我會更傾向使用snprintf，原因在於strncpy不會在溢位後自動補上結尾的`\0`，如果自己忘記補，那strncpy跟strcpy的沒甚麼兩樣，一樣會產生bug。

