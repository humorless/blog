---
title: get latest commit hash from git repo
date: 2017-03-15 13:37:22
tags: [git, dependency]
---

「取得 git 遠端 repository 最新的 commit hash 」是我去年在公司做的工作的一小部分，由於當時太偷懶，沒有認真思考「依賴」的問題，我現在得重寫一些程式了。

去年的解法是這樣子，因為公司的 git repository 是放在 gitlab 上。我觀察了 gitlab 和 github 都有相似的 atom.xml 。不僅路徑一樣、輸出的格式也一樣，我就乾脆使用 gitlab 的 RSS 訂閱 API 來查 git 遠端 repository 最新的 commit hash 。然而， gitlab 在中國是常常連不上的。為了中國的問題，不得不把 git repository 換成是中國境內的 git 服務商，換成了 coding.net 的服務。然後，我就必須要重寫程式了，因為 coding.net 並沒有 RSS 訂閱的功能。

<!--more-->

這個事情，仔細來思考，最根本的錯誤，就是我居然讓軟體的功能去依賴 gitlab 的一個 API 。所以一旦更換成別家的 git repository ，功能就得重新實現。理解了這個需求之後，我現在的重新實現，則是要讓這個「取得 git 遠端 repository 最新的 commit hash 」的功能，直接依賴於 git 指令。指令出乎意料的單純： ` git ls-remote $GITURL`
```
$ git ls-remote https://github.com/humorless/humorless.github.io.git
cb7a2998571cb25693867afcb24a7331f597768e        HEAD
1f7888215c04ac157edfeb6f30cff9eaebb774bd        refs/heads/backup
cb7a2998571cb25693867afcb24a7331f597768e        refs/heads/master
 
```

接下來的問題，就變成了我要如何讓後端的程式與 git 來做整合。公司的後端程式都是用 docker 來做布署，所以其實是可以在 docker 裡安裝 git 就可以使用 `git ls-remote`。那如果不安裝 git 行嗎？ 其實也是可以，一種是整合 git 的 library。或是更偷懶，直接用 `curl` 。( curl 的目標很容易取得，用 `tcpdump` 看一下 git ls-remote 執行過程即可。)

```
curl -X GET https://github.com/humorless/humorless.github.io.git/info/refs?service=git-upload-pack
001e# service=git-upload-pack
000000edcb7a2998571cb25693867afcb24a7331f597768e HEADmulti_ack thin-pack side-band side-band-64k ofs-delta shallow no-progress include-tag multi_ack_detailed no-done symref=HEAD:refs/heads/master agent=git/2:2.6.5~peff-sha1dc-1707-ga50b0b4
003f1f7888215c04ac157edfeb6f30cff9eaebb774bd refs/heads/backup
003fcb7a2998571cb25693867afcb24a7331f597768e refs/heads/master
0000
```

curl 的解法自然可能有比較差的可攜性，最後我大概不會採用它。而這次栽在 gitlab RSS 訂閱的事也再一次地提醒了自己：「寫程式時，務必要想清楚解法的依賴」。
