---
layout: post
title: Ubuntu отключаем поддержку netplan
description: "Disable netplan"
modified: 2019-02-18
tags: [ubuntu, netplan, net]
image:
  path: /images/matrix.jpg
  feature: matrix.jpg
---

Отключить Netplan 

Netpaln - это средство настройки сети по умолчанию, появившееся в Ubuntu 17.10,
 и если вы хотите отключить его, чтобы вернуться к старым способам настройки сети, 
используйте следующую процедуру. Netplan - это абстракция конфигурации сети YAML 
для различных бэкэндов (NetworkManager, networkd). Это утилита для простой настройки 
сети в системе. Его можно использовать, написав описание YAML необходимых сетевых 
интерфейсов с указанием того, для чего они должны быть настроены. Из этого описания 
будет сгенерирована необходимая конфигурация для выбранного инструмента рендеринга.



Edit the /etc/default/grub file
```css
sudo nano /etc/default/grub
```

Add the following line

```css
GRUB_CMDLINE_LINUX="netcfg/do_not_use_netplan=true"
```

Save and exit the file

Now update the grub using the following command

```css
sudo update-grub
```


Вам необходимо установить пакет ifupdown

```css
sudo apt install ifupdown
```

Теперь вы можете добавить все детали интерфейса в /etc/network/interfaces 
файл и перезагрузите компьютер / сервер Ubuntu.

