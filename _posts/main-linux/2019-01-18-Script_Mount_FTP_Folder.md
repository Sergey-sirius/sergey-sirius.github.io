---
layout: post
title: Монтируем папку FTP в файловую систему Bash  SCRIPT
description: "Bash Монтируем папку FTP в файловую систему Bash  SCRIPT:"
modified: 2019-01-18 00:00:03
tags: [main]
image:
  path: /images/matrix.jpg
  feature: matrix.jpg
---

** Монтируем папку FTP в файловую систему Bash  SCRIPT **

Для начала установим пакет:

```css
$ sudo apt install curlftpfs
```

Затем подмонтируем интересующий нас ftp-ресурс:

```css
$ mkdir temp-ftpfs
$ curlftpfs ftp://$USER:$PASSWD@$HOST/ temp-ftpfs
$ cd temp-ftpfs
```