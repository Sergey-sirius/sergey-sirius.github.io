---
layout: post
title: Установка Jekyll
description: ""
modified: 2014-12-24
tags: [Jekyll]
image:
  path: /images/matrix.jpg
  feature: matrix.jpg
---

Прежде чем мы установим Jekyll, нам нужно убедиться, что у нас есть все необходимые зависимости.

```css
sudo apt-get install ruby ruby-dev build-essential
```

2. Ставим jekyll

```css
gem install jekyll
```

3. создаем проект

```css
jekyll new siriusblog
```

4. bundle install

```css
sudo apt install ruby-bundler
bundle install
```

5. запуск сервиса 

```css
bundle exec jekyll serve
```

проверяем http://127.0.0.1:4000/


