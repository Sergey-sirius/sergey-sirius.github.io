---
layout: post
title: Мониторинг серверов система Tick Stack
description: "Мониторинг серверов система Tick StackUbuntu"
modified: 2019-05-31 00:00:05
tags: [mount hdd, main]
image:
  path: /images/matrix.jpg
  feature: matrix.jpg
---

** Мониторинг серверов система Tick Stack **

[](/files/main-linux/tick_stack.png)

1. Telegraf
Telegraf — управляемый плагином сервер для сбора показателей и отчетности. Telegraf интегрируется с разнообразными метриками, событиями и логами, обращаясь непосредственно к контейнерам и системам, в которых он работает, достает метрики из сторонних API или даже прослушивает показатели через сервисы ConsumerD и Kafka. Он также имеет выходные плагины для отправки показателей во множество других хранилищ данных, служб и query-сообщений, включая InfluxDB, Graphite, OpenTSDB, Datadog, Librato, Kafka, MQTT, NSQ.

2. InfluxDB
InfluxDB — Time Series Database, построенная с нуля, для обработки высоких нагрузок на запись и запрос. InfluxDB — это специализированное высокопроизводительное хранилище данных, написанное специально для временных данных (мониторинг DevOps, метрики приложений, данные датчика IoT и аналитика в реальном времени). Оно позволяет сохранять пространство на своем компьютере, настраивая InfluxDB для хранения данных в течение определенного периода времени и по его истечению — автоматического удаления любых нежелательных данных из системы. InfluxDB использует язык запросов, подобный SQL, для взаимодействия с данными.

3. Chronograf
Chronograf — это административный интерфейс пользователя и механизм визуализации платформы. Он упрощает настройку и обслуживание системы мониторинга и оповещения. Простой в использовании, он включает в себя шаблоны и библиотеки, позволяющие быстро создавать информационные борды с визуализацией данных в режиме реального времени и генерировать правила для алертов и автоматизации.

4. Kapacitor
Kapacitor — собственный механизм обработки данных. Он может обрабатывать как потоковые, так и пакетные данные от InfluxDB. Kapacitor позволяет подключать собственную пользовательскую логику или пользовательские функции для обработки предупреждений с динамическими порогами, сопоставления показателей для шаблонов, вычисления статистических аномалий и выполнения определенных действий на основе этих предупреждений, таких как динамическая перебалансировка нагрузки. Kapacitor интегрируется с HipChat, OpsGenie, Alerta, Sensu, PagerDuty, Slack и т. д.


Инсталяция
https://portal.influxdata.com/downloads

1 Telegraf v1.8.3
a) Docker
    docker pull telegraf

b) ubuntu
  wget https://dl.influxdata.com/telegraf/releases/telegraf_1.8.3-1_amd64.deb
  sudo dpkg -i telegraf_1.8.3-1_amd64.deb

c) RedHat & CentOS
    wget https://dl.influxdata.com/telegraf/releases/telegraf-1.8.3-1.x86_64.rpm
    sudo yum localinstall telegraf-1.8.3-1.x86_64.rpm

d) Windows Binaries (64-bit)
    wget https://dl.influxdata.com/telegraf/releases/telegraf-1.8.3_windows_amd64.zip
    unzip telegraf-1.8.3_windows_amd64.zip

e) Linux Binaries (64-bit)
    wget https://dl.influxdata.com/telegraf/releases/telegraf-1.8.3_linux_amd64.tar.gz
    tar xf telegraf-1.8.3_linux_amd64.tar.gz

f) Linux Binaries (32-bit)
    wget https://dl.influxdata.com/telegraf/releases/telegraf-1.8.3_linux_i386.tar.gz
    tar xf telegraf-1.8.3_linux_i386.tar.gz

g) Linux Binaries (ARM)
    wget https://dl.influxdata.com/telegraf/releases/telegraf-1.8.3_linux_armhf.tar.gz
    tar xf telegraf-1.8.3_linux_armhf.tar.gz

2. InfluxDB v1.7.1
a) Docker
    docker pull influxdb

b) ubuntu
    wget https://dl.influxdata.com/influxdb/releases/influxdb_1.7.1_amd64.deb
    sudo dpkg -i influxdb_1.7.1_amd64.deb  

c) RedHat & CentOS
    wget https://dl.influxdata.com/influxdb/releases/influxdb-1.7.1.x86_64.rpm
    sudo yum localinstall influxdb-1.7.1.x86_64.rpm

d) Windows Binaries (64-bit)
    https://dl.influxdata.com/influxdb/releases/influxdb-1.7.1_windows_amd64.zip
    unzip influxdb-1.7.1_windows_amd64.zip

e) Linux Binaries (64-bit)
    wget https://dl.influxdata.com/influxdb/releases/influxdb-1.7.1_linux_amd64.tar.gz
    tar xvfz influxdb-1.7.1_linux_amd64.tar.gz

f) Linux Binaries (32-bit)
    wget https://dl.influxdata.com/influxdb/releases/influxdb-1.7.1_linux_i386.tar.gz
    tar xvfz influxdb-1.7.1_linux_i386.tar.gz

g) Linux Binaries (ARM)
    wget https://dl.influxdata.com/influxdb/releases/influxdb-1.7.1_linux_armhf.tar.gz
    tar xvfz influxdb-1.7.1_linux_armhf.tar.gz

3. Chronograf v1.7.3

a) Docker
    docker pull quay.io/influxdb/chronograf:1.7.3
b) ubuntu  
    wget https://dl.influxdata.com/chronograf/releases/chronograf_1.7.3_amd64.deb
    sudo dpkg -i chronograf_1.7.3_amd64.deb
c) RedHat & CentOS
    wget https://dl.influxdata.com/chronograf/releases/chronograf-1.7.3.x86_64.rpm
    sudo yum localinstall chronograf-1.7.3.x86_64.rpm
d) Windows Binaries (64-bit)
    https://dl.influxdata.com/chronograf/releases/chronograf-1.7.3_windows_amd64.zip
    unzip chronograf-1.7.3_windows_amd64.zip
e) Linux Binaries (64-bit)
    wget https://dl.influxdata.com/chronograf/releases/chronograf-1.7.3_linux_amd64.tar.gz
    tar xvfz chronograf-1.7.3_linux_amd64.tar.gz
f) Linux Binaries (32-bit)
    wget https://dl.influxdata.com/chronograf/releases/chronograf-1.7.3_linux_i386.tar.gz
    tar xvfz chronograf-1.7.3_linux_i386.tar.gz
g) Linux Binaries (ARM)
    wget https://dl.influxdata.com/chronograf/releases/chronograf-1.7.3_linux_armhf.tar.gz
    tar xvfz chronograf-1.7.3_linux_armhf.tar.gz

4. Kapacitor v1.5.1

a) Docker
    docker pull kapacitor
b) ubuntu  
    wget https://dl.influxdata.com/kapacitor/releases/kapacitor_1.5.1_amd64.deb
    sudo dpkg -i kapacitor_1.5.1_amd64.deb
c) RedHat & CentOS
    wget https://dl.influxdata.com/kapacitor/releases/kapacitor-1.5.1.x86_64.rpm
    sudo yum localinstall kapacitor-1.5.1.x86_64.rpm
d) Windows Binaries (64-bit)
    wget https://dl.influxdata.com/kapacitor/releases/kapacitor-1.5.1_windows_amd64.zip
    unzip kapacitor-1.5.1_windows_amd64.zip
e) Linux Binaries (64-bit)
    wget https://dl.influxdata.com/kapacitor/releases/kapacitor-1.5.1_linux_amd64.tar.gz
    tar xvfz kapacitor-1.5.1_linux_amd64.tar.gz
f) Linux Binaries (32-bit)
    wget https://dl.influxdata.com/kapacitor/releases/kapacitor-1.5.1_linux_i386.tar.gz
    tar xvfz kapacitor-1.5.1_linux_i386.tar.gz
g) Linux Binaries (ARM)
    wget https://dl.influxdata.com/kapacitor/releases/kapacitor-1.5.1_linux_armhf.tar.gz
    tar xvfz kapacitor-1.5.1_linux_armhf.tar.gz


