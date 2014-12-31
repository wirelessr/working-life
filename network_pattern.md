# Network Pattern

Network pattern與design pattern不太一樣，design pattern在描述的議題通常是比較一般性的，無論是開發甚麼樣的軟體，都可能會面臨的問題；而network pattern就比較偏門，僅限網路相關的軟體架構才有可能碰到，所以network pattern的主題會比較偏向於解決同步/異步、多工/解多工和concurrent(不會翻譯)，幾乎所有的network pattern都圍繞在這幾個主題打轉。

而這其中比較廣為人知的兩個pattern是reactor和proactor。除此之外，還有apache server裡面的三個著名的MPM(Multi-Processing Module)，prefork、worker和event。