---
layout: post
title: Установка Docker
description: "Install Docker"
modified: 2019-01-20 00:00:04
tags: [docker, main]
image:
  path: /images/matrix.jpg
  feature: matrix.jpg
---

** Ставим Docker ubuntu **

```css
sudo apt-get install apt-transport-https ca-certificates curl gnupg2 software-properties-common

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

sudo apt-key fingerprint 0EBFCD88

sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

sudo apt-get update

sudo apt-get install docker-ce

apt-cache madison docker-ce

sudo docker run hello-world

```
