---
title: ttl
date: 2017-07-23 12:17:39
tags: [ttl, SQL, Redis, mongoDB, cache] 
---

公司的程式碼裡，有幾張 mysql 表格的 schema 長成這樣子： 
```
CREATE TABLE mytable (
  exist boolean DEFAULT NULL,
  updated datetime DEFAULT NULL ON UPDATE CURRENT_TIMESTAMP,
  ...
)
```

而該張 table 對應的更新 SQL 語句，最後也會有下列的這一段：
```
UPDATE mytable SET exist = 0, updated = FROM_UNIXTIME(%d) WHERE exist = 1 AND updated <= DATE_SUB(NOW(), INTERVAL 10 MINUTE);
```
對應的語意是這樣子：『如果這張表格 (table) 裡的某個列 (row) ，已經整整十分鐘沒有任何的寫入或是更新的話，就將這個列的屬性設定為不存在 (exist = 0)』

會需要用這種寫法：因為這張表格 (table) ，它儲存的資訊是一種「快取」 (cache) 性質的資訊，任何的列 (row) 如果超過十分鐘沒有任何更新或是寫入的話，表示這個列在原始的資料來源已經不存在了。換言之，這個寫法可以算是一種 hack ，因為 mysql 裡的 row 並沒有 time to live (ttl) 的語法。遇到要表達 ttl 語意的情況時，就會需要多幾個欄位和對應的 SQL 語句，才能表現。另一方面， Redis 和 mongoDB 因為設計時，本來就有考慮主要的用途之一要做為 cache 使用，自然也就會提供 ttl 的語法了。
