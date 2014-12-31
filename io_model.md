# I/O Model

這篇先來暖身一下，講講這幾個網路編程常聽到卻又好像似懂非懂的名詞：synchronous/asynchronous、blocking/non-blocking。首先，這邊講的I/O都是user space的I/O，而且著重在網路編程，在名詞介紹前一定要先有個概念，user space的I/O牽扯的兩個步驟：等資料來 & 從kernel copy到user space。
 
而那幾個名詞就是在這兩個步驟上有些許差異，blocking I/O是Linux socket programming的default行為

![](http://i.imgur.com/ynKtmeI.jpg)

blocking I/O就是當你call了recvfrom之後，一定要等，等到收完才叫結束，然後recefrom的return value就是收進來多少，若是一直沒資料進來，你的process就會被卡死在那。
 
在講non-blocking之前，我們先講I/O multiplexing，常見的API是select或poll，因為我跟poll實在不熟，所以就講select吧！這因為blocking mode在沒收到資料時就會卡住，若是一次需要收多個socket fd時很要命，所以這時候就出現I/O多工(是這樣翻吧)，也就是select，一次可以聽許多個fd，若是有資料進readable fd set就可以去讀，非常實用，而select也可以設定timeout跳脫

![](http://i.imgur.com/FrViJZ9.jpg)

select本身也是一種blocking，但是當資料準備好，就可以return，再換成recvfrom，這時候就可以直接收了，因為select都已經告訴你準備好了。
 
接著是non-blocking，因為blocking的特性就是卡住process，有時候除了I/O之外process還想做別的事阿！這時候就會將socket設成non-blocking (O_NONBLOCK)

![](http://i.imgur.com/yY8zCIx.jpg)

我個人是覺得non-blocking有點蠢啦，因為就只是不斷去問好了沒，如果好了就收，沒好就跳掉，所以我自己寫code都是用select。
 
以上講的都是synchronous I/O，之所以稱為同步是因為user space和kernel是搭配合作的，一定是從user space trigger一個I/O，然後kernel回應這個請求。以前我一直以為non-blocking是asynchronous I/O，但從上面的解說應該很明顯，它不是！
 
那到底啥是非同步呢?
 
簡單敘述一下概念，就是user space跟kernel說：『我要資料，好了就丟給我』，然後user space就自己去玩自己的了，而kernel 也很聽話，當資料進來，kernel 就收集資料，並且預先將資料copy到user space，準備齊全之後跟caller說：『先生，快遞，請取貨』，完全就是一個送貨到府、使命必達

![](http://i.imgur.com/cnu0rP3.jpg)

signal是由aio_read指定的，所以也不會送錯人，因為user space和kernel 其實是各忙各的，彼此互不相干所以稱為非同步

。








