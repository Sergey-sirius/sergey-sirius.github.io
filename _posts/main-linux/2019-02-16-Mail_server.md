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

****Настройка Почтового сервера сервера

В принципе дальнейшие команды подразумевают использование sudo


Пробежимся по основным настройкам 

*1. Хосты hostname and FQDN

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
$ apt install unbound
```
Обновите корневой ключ DNSSEC и перезагрузите Unbound сервис:
```css
$ su -c "unbound-anchor -a /var/lib/unbound/root.key" - unbound
$ systemctl reload unbound
```

если установлен пакет «dnsutils»
```css
$ dig @127.0.0.1 e-mag.com.ua +short +dnssec
  
  external your ip 91.219.83.33
```

Если команда dig сработала, теперь вы можете установить Unbound в качестве основного преобразователя DNS для вашей почтовой системы:
```css
$ apt install resolvconf
$ echo "nameserver 127.0.0.1" >> /etc/resolvconf/resolv.conf.d/head

$ nslookup e-mag.com.ua | grep Server
  Server:	127.0.0.1
```

**Теперь у вас есть собственный DNS-распознаватель. Давайте продолжим с настройкой DNS ...

Проверить записи DNS провайдера

```css
mail.e-mag.com.ua. 86400 IN A    91.219.83.33

imap.e-mag.com.ua. 86400 IN CNAME mail.e-mag.com.ua.
pop.e-mag.com.ua. 86400 IN CNAME mail.e-mag.com.ua.
smtp.e-mag.com.ua. 86400 IN CNAME mail.e-mag.com.ua.

e-mag.com.ua. 86400 IN MX 0 mail.e-mag.com.ua.
```
… И мы также хотим, чтобы он обрабатывал электронные письма для «domain2» и «domain3»:

```css
domain2.tld. 86400 IN MX 0 mail.e-mag.com.ua.
domain3.tld. 86400 IN MX 0 mail.e-mag.com.ua.
```
(последние три записи должны быть созданы в соответствующих им файлах зон)

Обратный DNS 

Обратная запись DNS (также называемая «PTR-запись») сопоставляет полное доменное имя с IP-адресом. 
Многие популярные провайдеры электронной почты проверяют записи PTR других почтовых серверов и отказывают в получении электронной почты от них, 
если они не могут найти подходящее доменное имя для их IP-адреса. Вам понадобятся PTR-записи для всех IP-адресов вашего сервера. 
Все они должны указывать на mail.e-mag.com.ua. 
В большинстве случаев обратные записи DNS могут быть установлены через веб-интерфейс вашего хостера или через службу поддержки сервера.

Пример опроса сервера доменных имен google на доменную зону ya.ru:
```css
$ dig @google-public-dns-a.google.com. ya.ru
```

Узнать какие есть почтовые сервера имеющие MX записи в DNS: 
```css
$ dig e-mag.com.ua mx

```
PTR
```css
$ dig -x 91.219.83.33
; <<>> DiG 9.11.3-1ubuntu1.3-Ubuntu <<>> -x 91.219.83.33
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NXDOMAIN, id: 46021
;; flags: qr rd ra ad; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;33.83.219.91.in-addr.arpa.	IN	PTR

;; AUTHORITY SECTION:
91.in-addr.arpa.	3508	IN	SOA	pri.authdns.ripe.net. dns.ripe.net. 1550417273 3600 600 864000 3600

;; Query time: 81 msec
;; SERVER: 127.0.0.1#53(127.0.0.1)
;; WHEN: Sun Feb 17 23:14:23 EET 2019
;; MSG SIZE  rcvd: 114

```
SPF records
Записи SPF были изобретены для поддержки борьбы со спам-серверами. 
К сожалению, это оказалось неправильным представлением, и вы больше не можете полагаться на эти записи. 
SPF описывает, каким почтовым серверам разрешено отправлять электронную почту для домена, а какие нет. 
В некоторых случаях (например, списки рассылки) концепция SPF не работает. Тем не менее, некоторые провайдеры ожидают, 
что вы установите рекорд SPF - если вы этого не сделаете, вы получите некоторое количество баллов за ваш «рейтинг вероятности спама». 

Итак, давайте сделаем компромисс и обеспечим нейтральную запись SPF:
```css
e-mag.com.ua. 3600 IN TXT v=spf1 a:mail.e-mag.com.ua ?all

domain2.tld. 3600 IN TXT v=spf1 include:e-mag.com.ua ?all
domain3.tld. 3600 IN TXT v=spf1 include:e-mag.com.ua ?all
```

DMARC records

Записи DMARC устанавливают правила для сторонних почтовых серверов, которые сообщают им, как обращаться с не аутентифицированными 
или неправильно аутентифицированными электронными письмами. Если спамер отправляет поддельное электронное письмо и использует 
ваш домен e-mag.com.ua, принимающий сервер свяжется с DNS и запросит запись DMARC. 
Если он обнаруживает, что SPF или DKIM отказывают (что будет иметь место), сервер будет действовать в соответствии с записью DMARC. 
Это хорошая идея, чтобы отклонить такие электронные письма:

```css
_dmarc.e-mag.com.ua.   3600    IN TXT    v=DMARC1; p=reject;

_dmarc.domain2.tld.   3600    IN TXT    v=DMARC1; p=reject;
_dmarc.domain3.tld.   3600    IN TXT    v=DMARC1; p=reject;
```
Вы можете создать свои собственные записи политики DMARC на https://elasticemail.com/dmarc/.


Set up TLS certificates

Современный почтовый сервер не может работать серьезно без сертификатов TLS. 
Для этой цели мы будем использовать сертификаты Let Encrypt, поскольку они бесплатны и принимаются всеми браузерами, 
почтовыми клиентами и операционными системами. 
Если у вас уже есть действительные сертификаты, вы можете использовать их вместо.

Получить новые сертификаты

Используйте официальный клиент командной строки «certbot» для получения новых сертификатов для вашей почтовой системы:
```css
$ apt install certbot

$ certbot certonly --standalone --rsa-key-size 4096 -d mail.e-mag.com.ua -d imap.e-mag.com.ua -d smtp.e-mag.com.ua

```

Согласившись с условиями использования и указав свой адрес электронной почты, вы мгновенно получите действительные сертификаты. 
Они сохраняются по адресу: /etc/letsencrypt/live/mail.e-mag.com.ua и действительны для трех имен доменов вашей почтовой системы:
Ваши дополнительные домены «domain2» и «domain3» не должны быть включены в этот сертификат. 

Есть несколько новых файлов, которые вам понадобятся:

- cert.pem: 		Ваш почтовый сертификат
- chain.pem:		Сертификат CA
- fullchain.pem:	сертификат почтового сервера + сертификат CA
- privkey.pem:		Закрытый ключ для сертификата почтового сервера

Далее будут использоваться только два файла сертификатов.


Auto-Renewal - Автоматическое продление cronjob:

```css
$ crontab -e

@weekly certbot renew --pre-hook "systemctl stop nginx" --post-hook "systemctl start nginx" --renew-hook "systemctl reload nginx; 
systemctl reload dovecot; 
systemctl reload postfix" --quiet

```

MySQL database setup


Создайте новую базу данных с именем «vmail»:
```css
create database vmail CHARACTER SET 'utf8';

grant select on vmail.* to 'vmail'@'localhost' identified by 'vmaildbpass';

use vmail;

CREATE TABLE `domains` (
    `id` int unsigned NOT NULL AUTO_INCREMENT,
    `domain` varchar(255) NOT NULL,
    PRIMARY KEY (`id`),
    UNIQUE KEY (`domain`)
);

CREATE TABLE `accounts` (
    `id` int unsigned NOT NULL AUTO_INCREMENT,
    `username` varchar(64) NOT NULL,
    `domain` varchar(255) NOT NULL,
    `password` varchar(255) NOT NULL,
    `quota` int unsigned DEFAULT '0',
    `enabled` boolean DEFAULT '0',
    `sendonly` boolean DEFAULT '0',
    PRIMARY KEY (id),
    UNIQUE KEY (`username`, `domain`),
    FOREIGN KEY (`domain`) REFERENCES `domains` (`domain`)
);

CREATE TABLE `aliases` (
    `id` int unsigned NOT NULL AUTO_INCREMENT,
    `source_username` varchar(64) NOT NULL,
    `source_domain` varchar(255) NOT NULL,
    `destination_username` varchar(64) NOT NULL,
    `destination_domain` varchar(255) NOT NULL,
    `enabled` boolean DEFAULT '0',
    PRIMARY KEY (`id`),
    UNIQUE KEY (`source_username`, `source_domain`, `destination_username`, `destination_domain`),
    FOREIGN KEY (`source_domain`) REFERENCES `domains` (`domain`)
);

CREATE TABLE `tlspolicies` (
    `id` int unsigned NOT NULL AUTO_INCREMENT,
    `domain` varchar(255) NOT NULL,
    `policy` enum('none', 'may', 'encrypt', 'dane', 'dane-only', 'fingerprint', 'verify', 'secure') NOT NULL,
    `params` varchar(255),
    PRIMARY KEY (`id`),
    UNIQUE KEY (`domain`)
);

quit;
```

vmail-user and vmail-directory

Все электронные письма и скриптовые файлы сохраняются в специальном каталоге / var / vmail. 
Только связанный пользователь vmail имеет к нему доступ. Dovecot будет использовать эту учетную запись для выполнения 
своих операций в файловой системе. Создать vmail-каталог:
```css
$ mkdir /var/vmail

$ adduser --disabled-login --disabled-password --home /var/vmail vmail

$ mkdir /var/vmail/mailboxes
$ mkdir -p /var/vmail/sieve/global

$ chown -R vmail /var/vmail
$ chgrp -R vmail /var/vmail
$ chmod -R 770 /var/vmail

```

```css

```

```css

```

```css

```

```css

```
