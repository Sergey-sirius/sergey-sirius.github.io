---
layout: post
title: Как Войти В Docker-Контейнер
description: "Как Войти В Docker-Контейнер"
modified: 2019-01-20 00:00:01
tags: [docker]
image:
  path: /images/matrix.jpg
  feature: matrix.jpg
---

** Как Войти В Docker-Контейнер **

Чтобы войти в контейнер, нужно узнать имя или ID контейнера, выполните команду: 
```css
$ docker ps

CONTAINER ID  IMAGE    COMMAND  CREATED      STATUS      PORTS  NAMES
72ca2488b353  my_image          X hours ago  Up X hours         my_container
```

Войдите в Docker-контейнер по имени или ID контейнера и запустите интерактивную оболочку bash:
```css
$ docker exec -it 72ca2488b353 bash
```

Если внутри контейнера не удастся найти оболочку bash, вы увидите что-то вроде следующего сообщения:
```css
oci runtime error: exec failed: container_linux.go:265: starting container process caused «exec: \»bash\»: executable file not found in $PATH»
```

В этом случае вы можете войти в Docker-контейнер и затупить простую оболочку sh:
```css
$ docker exec -it 72ca2488b353 sh
```