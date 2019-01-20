---
layout: post
title: Монтирование новых жестких дисков в системе Ubuntu
description: "Подключаем новые диски в систему Ubuntu"
modified: 2019-01-20 00:00:05
tags: [mount hdd, main]
image:
  path: /images/matrix.jpg
  feature: matrix.jpg
---

** Подключаем новые диски в систему Ubuntu **

1. Список доступных дисков
```css
$ sudo fdisk -l | grep 'Disk /dev/sd'
```

2. Если требуется провести разметку воспользуйтесб командой fdisk 
```css
$ sudo fdisk /dev/sdb
```

3. Форматирование раздела #1 (цифра конечно может быть и другой, в зависимости от структуры Ваших разделов
```css
$ sudo mkfs.ext4 /dev/sdb1
```

4. Создадим точку монтирования -- каталог /hdd в директории /media
```css
$ sudo mkdir /media/hdd
```

5. Монтируем раздел в созданный каталог:
```css
$ sudo mount /dev/sdb1 /media/hdd
```

6. Проверяем 
```css
$ sudo df -h
```

7. Для автоматизации монтирования при старте операционной системы в Ubuntu отвечает файл /etc/fstab.
В него то мы и добавим команду на монтирование раздела. Открываем файл /etc/fstab в редакторе
```css
$ sudo nano /etc/fstab
```

вставляем строку

/dev/sdb1 /media/hdd ext4 defaults 1 2

сохраняем (Ctrl+O) и закрываем редактор

8. Проверка

8а. 1 способ. -- Перезагрузить Ubuntu и команда df -h покажет что раздел /dev/sdb1 был смонтирован.
8б. 2 способ. -- Нужно отмонтировать раздел 
```css
    $ sudo umount /media/hdd 
    $ sudo mount -a
```

команда df -h покажет что раздел /dev/sdb1 был смонтирован

