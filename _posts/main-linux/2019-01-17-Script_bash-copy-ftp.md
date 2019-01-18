---
layout: post
title: Копируем на FTP скриптом bash
description: "Bash FTP script copy"
modified: 2019-01-17 00:00:02
tags: [main]
image:
  path: /images/matrix.jpg
  feature: matrix.jpg
---

**FTP COPY Bash  SCRIPT:**

```css

#!/bin/sh

SERVER=192.168.0.1
USER=sergey-novikov
SOURCE_DIRS="/bin /boot /dev /etc /home /lib64 /opt /root /sbin /usr /var"
BACKUP_DIR=Backups

ftp -n $SERVER << EOS
user $USER
verbose
prompt
cd $BACKUP_DIR
mdelete *.tar.gz
binary
put "| tar cvzf - $SOURCE_DIRS" `date +%y%m%d`.tar.gz
quit
EOS

```