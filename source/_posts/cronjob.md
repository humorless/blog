---
title: cronjob
date: 2017-08-08 18:44:00
tags: ["cron", "scripts"]
---

話說我平常在工作上，有時候寫一些簡單的 script ，也有一些習慣。而這些習慣在我把 script 設定成 cronjob 的時候，就造成了許多 bugs 。
<!-- more -->
1. 使用 env 來啟動直譯器
```
#!/usr/bin/env python3

```
   腳本裡如果有上頭的這一行，直接把腳本放進 cronjob 會無法執行，必須直接指定直譯器的路徑。  
   例如： ` python3 /your/path/script.py ` 而比較省事的方式，就是直接在腳本裡指定直譯器的位置。

2. 直在在腳本裡，讀取同一個資料夾之下的 config 檔。
   
   在設定 cronjob 時，就會變成一定要寫成 
```
   * * * * *   cd /your/path/  && /your/path/script.sh
```
   否則腳本會找不到它需要的 config 檔。比較好的設定方式，是腳本要接受一個 command line argument 來指定 config 檔的位置。
   這樣子才可以寫成如下的格式
```
   * * * * *   /your/path/script.sh --config /your/path/config.yaml
```


最後，經過了設定 cronjob 的種種除錯之後，我體會到解決 cronjob 的問題，最重要的，還是一開始就這樣子設定：
```
   * * * * *   /your/path/script.sh  >>/tmp/script.log 2>&1 
```
