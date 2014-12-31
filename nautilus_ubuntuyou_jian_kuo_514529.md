# Nautilus (Ubuntu右鍵擴充)

我Win PC壞了，所以全面移轉到ubuntu上工作。windows有很多diff tool都有右鍵整合，ubuntu上最常用的meld沒有，所以我寫了一個script放到右鍵來做這件事
```
#!/bin/bash
args=`echo "$NAUTILUS_SCRIPT_SELECTED_FILE_PATHS" | sed ':a;N;$!ba;s/\n/ /g'`
meld $args
```
把上面的scipt放到`~/.gnome2/nautilus-scripts`，然後選兩個file/folder按右鍵就會出現一個Scipts的選項，選剛剛放進去的那個scipt就OK了。

以上作法適用於Ubuntu 12.04，但現在的LTS是14.04。

14.04作法有改變，改把下面的script放到`~/.local/share/nautilus/scripts`，然後用同樣的做法就可以比較folder或file
```
#!/bin/bash
command -v meld >/dev/null 2>&1 || { echo >&2 "I require meld but it's not installed. Aborting."; exit 1; }
if [ -n "$3" ]
then
    meld "$1" "$2" "$3"
    elif [ -n "$2" ]
then
    meld "$1" "$2"
elif [ -n "$1" ]
then
    meld "$1"
else
    meld                                                                                                                                       
fi
```
不只是script放的點，連內定變數的用法也改成正規的$1，$2，$3，以上做法同樣適用於13.10。



