# Message queue, semaphore, and shared memory大混戰

這篇要講的主題乍看還滿恐怖的，牽扯了三個IPC。這是我碰到的一個架構上的bug，我稍微敘述一下事發過程。
 
###情境
 
有一個event producer會在有必要時通知另外一個consumer做事，透過message queue的方式將event type, event data, and participant告知consumer；而consumer就根據這些資訊去做該做的事。
 
###問題
 
當軟體架構越來越龐大時，consumer一定會變肥，而producer並不是做事的人，所以成長幅度有限。這造成一個可怕的後果，總有一天message queue會被塞爆，因為consumer的處理速度跟不上event產生的速度，尤其在linux下default message queue的size是16384 byte，若一個event特大，像剛剛提到的那樣，那麼根本放不了多少event。
 
###解決之道
 
將通知的機制拆開，由semaphore紀錄待處理的event數量，而shared memory儲存event的內容，可以是shared memory array或根本就只是個file，越方便查找越好。
 
原來的message queue的通知機制依然保留，但event就只有當semaphore從0變為1時才會產生，producer只要告知consumer該起來做事即可，反正就是把event通通塞進shared memory就對了。
 
如此producer和consumer就可以透過semaphore來彼此同步並且大幅度減少event的數量，更有甚者，event的size也可以變小，甚至只要一byte，那麼default message queue也很夠用。







