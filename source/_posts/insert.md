---
title: insert
date: 2017-06-03 05:03:24
tags: ["temp table", "insert on duplicate key update", "SQL"]
---

同事為了快速開發，讀寫資料庫的程式碼並沒有多加考慮「資料庫」的特性，而是直接把資料庫當作 memory 來操作。這樣子的寫法，在資料量少的時候還好，資料量大的時候，就需要重構了。這段程式碼處理的問題是： 有一張 table ，有 4000 多個 rows 。每一段時間，就會得到大部分相同但是少部分不同的資料，要寫入這張表。相同的 row 要更新、不存在的 row 要插入。原始的寫法則是：「對於要更新的資料的每一筆，用它的 primary key ，也就是 `hostname` 這個 column 去 query 這張 table ，如果存在就更新，如果不存在就插入。」

<!--more-->

## 重構的想法

原始的作法造成了效能的瓶頸。改進的方式則是要依據幾個重要的想法：
1. 因為 application server 和 database server 之間有透過網路傳輸，所以「要更新的資料」，應該要儘量批次地送到 database server 。而不是一個一個 row 來傳送。
2. 對於資料的運算，可以發生在 application server ，也可以發生在 database server。如果運算可以利用 SQL 語言來描述的話，就可以發生在 database server，也就可以先把資料送到 database 再做運算。

我查到的資料裡，還有發現 MySQL 為了這一類的問題，還有一個專用的指令 `insert on duplicate key update` ，不過，由於我的資料表有 auto increment key ，我就選擇用 temp table 的作法。

## 重構的程式碼

```sql
# 準備全新的 temporary table 
DROP TEMPORARY TABLE IF EXISTS temp;
CREATE TEMPORARY TABLE temp LIKE hosts;

# 將資料從「外部」填入 temp 表之中
# 利用迴圈來建構 SQL 的 values part。
# 需要注意 mysql max_allowed_packet 的常數。
INSERT INTO temp
(hostname, exist, activate, platform, platforms, idc, ip, isp, province, city, updated)
       VALUES    (?, ?, ?, ?, ?, ?, ?, ?, ?, ?, NOW());

# 開始交易
BEGIN;

# 利用 inner join 找出有重複的 row，做 update 的操作。
UPDATE hosts
INNER JOIN temp
ON   hosts.hostname =   temp.hostname
SET hosts.exist = 1, hosts.activate = temp.activate, hosts.platform = temp.platform,
    hosts.platforms = temp.platforms, hosts.idc = temp.idc, hosts.ip = temp.ip,
    hosts.isp = temp.isp, hosts.province = temp.province, hosts.city = temp.city,
    hosts.updated = temp.updated;

# 利用 anti join 找出沒有重複出現的 row ，做 insert 的操作。
INSERT INTO hosts(hostname, exist, activate, platform, platforms, idc, ip, isp, province, city, updated)
SELECT temp.hostname, temp.exist, temp.activate, temp.platform, temp.platforms, temp.idc,
       temp.ip, temp.isp, temp.province, temp.city, temp.updated
FROM temp LEFT JOIN hosts
ON  temp.hostname = hosts.hostname
WHERE hosts.hostname IS NULL;

# 結束交易
COMMIT;

DROP TEMPORARY TABLE temp;

```
## 安全性議題

當我把重構完成之後，就請同事幫我做 code review 。很快就找到了一個很大的問題，沒有考慮 security 。因為來源的資料是系統外的，有可能會有 SQL injection 。所以資料在插入資料表之前，應該要做 prepared statement ，才能確保安全性。

## 實際效能測試 

做效能測試之後，發現有兩種解法，效能都是可以接受的。
1. 在插入大量的資料進入 temp table 時，使用 multiple insert 的語法。只用一個 SQL 指令就完成插入。(當然資料要先做字元的逸脫處理。)　用這樣子的作法的話，要考慮 4000 個 rows 的資料一次送到 database server 時， mysql 的 `max_allowed_packet` 常數是否夠大。
2. 在插入大量的資料進入 temp table 時，逐條插入，但是使用 prepared statement 。這樣子的效能會略慢一些些，但是基本上也是夠快的。

## 除錯與驗証
 
開發的過程中，由於使用了 temp table ，而 MySQL 的 temp table 是只屬於 session 的。此外，MySQL 又會做 connection pooling ，所以開發的過程中，一度遇到「 temp table 不存在」的錯誤訊息，怎麼看都看不出 SQL 哪裡出錯，結果原來是因為 connection pooling 的關系，不同的 connection 無法共享 temp table 。 

由於有 connection pooling ，所以還是得要看 mysql general log 才能確定 connection 的 id 是否正確。
```bash
# in mysql, set these two parameters to show general log
SET GLOBAL general_log = 'ON';
SET GLOBAL general_log_file = '/var/log/mysql/general.log';

# in shell, execute it. 
docker exec -t $MYSQL_docker_name tail -f /var/log/mysql/general.log > mysql.log
```
