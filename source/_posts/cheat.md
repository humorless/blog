---
title: Linux 小抄大全
date: 2017-03-15 14:53:31
tags: cheat
---

遇到指令不會用時，大家應該多少用過 cheatsheet 這個字當關鍵字吧，那如果直接安裝在電腦上呢？

```
# 安裝 cheat 這個指令
$sudo pip install cheat

```
<!--more-->

用 cheat 查到的，常常都是該指令最經典的用法，比方說：
```
$cheat python
# Desc: Python is a high-level programming language.

# Basic example of server with python
# Will start a Web Server in the current directory on port 8000
# go to http://127.0.0.1:8000

# Python v2.7
python -m SimpleHTTPServer
# Python 3
python -m http.server 8000

# SMTP-Server for debugging, messages will be discarded, and printed on stdout.
python -m smtpd -n -c DebuggingServer localhost:1025

# Pretty print a json
python -mjson.tool

```

什麼？你問「cheat 還有哪些用法？」
```
# cheat 它的功能是"小抄" ，既然不確定怎麼用的話，就先 cheat 自己吧。
$cheat cheat
```
