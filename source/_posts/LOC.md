---
title: 程式語言「表達能力」的排序
date: 2017-04-02 14:06:34
tags: ["programming language expressiveness"]
excerpt: "程式語言的表達能力排行"
---

[原始的文章](http://redmonk.com/dberkholz/2013/03/25/programming-languages-ranked-by-expressiveness/)來自於 RedMonk 網站。作者相當有創意地提出了統計的方法，來比較不同程式語的的表達能力。

有可能將不同的程式語言以「表達能力」或說是「效率」做個排序嗎？換言之，你能夠比較「透過某些程式語言來寫出某個概念有多容易」嗎？其中一種可以用來做出這種比較的方案是：**每一次的原始碼提交 (commit) 有幾行原始碼的改變**。這個方案可以提供一種觀點：一種程式語言可以讓你在同樣的行數空間內擁有多大的表達能力。因為在程式裡，臭蟲 (bug) 的數量是正比於程式碼的行數，而非這些程式碼表達了多少的概念，所以一種語言如果可以在同樣的行數內表達更多的概念就可以被認為是表達能力更強。


![expressiveness weight](/2017/04/02/LOC/expressiveness_weighted.png)
第一張圖是用每一個 commit 裡的 LOC(lines of code) 的中位數來做排序

![expressiveness iqr](/2017/04/02/LOC/expressiveness_iqr.png)
再考慮更深入一些，理想的程式語言應該是：
  1. 夠簡單、容易學，而且大部分的開發者都可以高效率地使用它
  2. 在各式各樣的領域裡，都有接近相同的表達能力。

所以，我們應該利用「每一個 commit 的 LOC 的四分位差 (百分之第25位和百分之第75位的差距)」來代表上述兩點的衡量標準。
第二張圖是用每一個 commit 裡的 LOC(lines of code) 的四分位差來做排序
