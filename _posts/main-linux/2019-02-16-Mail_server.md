---
layout: post
title: Настройка Mail сервера на Ubuntu
description: "Postfix and other ...."
modified: 2019-02-16
tags: [Postfix, Ubuntu, Mail]
image:
  path: /images/matrix.jpg
  feature: matrix.jpg
---

Настройка Почтового сервера сервера

В принципе дальнейшие команды подразумевают использование sudo


Пробежимся по основным настройкам 

1. Хосты hostname and FQDN

resolv.conf "забывает" настройки nameservers
Можно ввести атрибут запрета на изменение установка 
запрета - $ sudo chattr + i имя_файла
снятие запрета - $ sudo chattr - i имя_файла

хост hp

```css
$ hostnamectl set-hostname --static hp

```

/etc/hosts
```css
$ cat hosts
127.0.0.1	localhost.localdomain	localhost
127.0.1.1	hp.e-mag.com.ua hp
::1		localhost6.localdomain6	localhost6

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
ff02::3 ip6-allhosts
```

```css
$ hostname
hp
```

```css
$ hostname --fqdn
hp.e-mag.com.ua
```

```css
$ echo $(hostname -f) > /etc/mailname

$ cat /etc/mailname
hp.e-mag.com.ua
```
Имя хоста в вашей командной оболочке будет адаптировано после перезагрузки

2 DNS а зачем ? 

Сервисы в вашей системе сильно зависят от DNS-запросов. 
Я настоятельно рекомендую установить Unbound в качестве локального преобразователя 
DNS и -cache! Некоторые провайдеры серверов ограничивают доступ к своим предварительно определенным DNS-распознавателям, что может вызвать проблемы. Особенно Rspamd делает много DNS-запросов в зависимости от загрузки почтовой системы. Кроме того, блочные списки Spamhaus часто можно использовать только с собственными распознавателями DNS (мы собираемся использовать Spamhaus с постэкраном).

```css

```
