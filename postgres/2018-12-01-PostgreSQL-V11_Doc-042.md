---
layout: page
title: Глава 42. Процедурные языки
description: ""
tags: [PostgreSQL]
image:
  feature: abstract-11.jpg
  #credit: dargadgetz
  #creditlink: http://www.dargadgetz.com/ios-7-abstract-wallpaper-pack-for-iphone-5-and-ipod-touch-retina/
share: true
modified: 2018-12-03 T15:14:43-04:00
---

Глава 42. Процедурные языки


PostgreSQL позволяет разрабатывать пользовательские функции не только на SQL и C, но и на
других языках. Эти языки в целом называются процедурными языками (PL, Procedural Language).
Если функция написана на процедурном языке, сервер баз данных сам по себе не знает, как интер-
претировать её исходный текст. Вместо этого он передаёт эту задачу специальному обработчику,
понимающему данный язык. Обработчик может либо выполнить всю работу по разбору, синтакси-
ческому анализу, выполнению кода и т. д., либо действовать как «прослойка» между PostgreSQL и
внешним исполнителем языка программирования. Сам обработчик представляет собой функцию
на языке C, скомпилированную в виде разделяемого объекта и загружаемую по требованию, как
и любая другая функция на C.
В настоящее время стандартный дистрибутив PostgreSQL включает четыре процедурных языка:
PL/pgSQL (Глава  43), PL/Tcl (Глава  44), PL/Perl (Глава  45) и PL/Python (Глава  46). Существуют и
другие процедурные языки, поддержка которых не включена в базовый дистрибутив. Информацию
о них можно найти в Приложении H. Кроме того, пользователи могут реализовать и другие языки;
основы разработки нового процедурного языка рассматриваются в Главе 56.
42.1. Установка процедурных языков
Прежде всего, процедурный язык должен быть «установлен» в каждую базу данных, где он будет
использоваться. Но процедурные языки, устанавливаемые в базу данных template1, автоматиче-
ски становятся доступными во всех впоследствии создаваемых базах, так как их определения в
template1 будут скопированы командой CREATE DATABASE. Таким образом, администратор баз дан-
ных может выбрать, какие языки будут доступны в определённых базах данных, и при желании
сделать некоторые языки доступными по умолчанию.
Для языков, включённых в стандартный дистрибутив, достаточно выполнить команду CREATE
EXTENSION имя_языка, чтобы установить язык в текущую базу данных. Описанная ниже ручная про-
цедура рекомендуется только для установки языков, не упакованных в виде расширений.
Установка процедурного языка вручную
Процедурный язык устанавливается в базу данных в пять этапов, и выполнять их должен админи-
стратор баз данных. В большинстве случаев необходимые команды SQL следует упаковать в виде
установочного скрипта «расширения», чтобы их можно было выполнить, воспользовавшись коман-
дой CREATE EXTENSION.
1. Разделяемый объект для обработчика языка должен быть скомпилирован и установлен в со-
ответствующий каталог библиотек. Это в принципе не отличается от сборки и установки до-
полнительных модулей с обычными функциями на языке C; см. Подраздел 38.10.5. Часто обра-
ботчик языка зависит от внешней библиотеки, в которой собственно реализован исполнитель
языка программирования; в таких случаях нужно установить и эту библиотеку.
2. Обработчик должен быть объявлен командой
CREATE FUNCTION имя_функции_обработчика()
RETURNS language_handler
AS 'путь-к-разделяемому-объекту'
LANGUAGE C;
Специальный тип возврата language_handler говорит СУБД, что эта функция не возвращает
какой-либо определённый тип данных SQL, и значит её нельзя использовать непосредственно
в операторах SQL.
3.
(Optional) Дополнительно обработчик языка может предоставить функцию обработки «внед-
рённого кода», которая будет выполнять анонимные блоки кода (команды DO), написанные на
этом языке. Если для языка есть обработчик внедрённого кода, объявите его такой командой:
CREATE FUNCTION имя_обработчика_внедрённого_кода(internal)
RETURNS void
1121Процедурные языки
AS 'путь-к-разделяемому-объекту'
LANGUAGE C;
4.
(Optional) Кроме того, обработчик языка может предоставить функцию «проверки», которая бу-
дет проверять корректность определения функции, собственно не выполняя её. Функция про-
верки, если она существует, вызывается командой CREATE FUNCTION. Если такая функция для
языка определена, объявите её такой командой:
CREATE FUNCTION имя_функции_проверки(oid)
RETURNS void
AS 'путь-к-разделяемому-объекту'
LANGUAGE C STRICT;
5.
Наконец, процедурный язык должен быть объявлен командой
CREATE [TRUSTED] [PROCEDURAL] LANGUAGE имя-языка
HANDLER имя_функции_обработчика
[INLINE имя_обработчика_внедрённого_кода]
[VALIDATOR имя_функции_проверки] ;
Необязательное ключевое слово TRUSTED (доверенный) указывает, что язык не предоставляет
пользователю доступ к данным, которого он не имел бы без него. Доверенные языки предна-
значены для обычных пользователей баз данных, не имеющих прав суперпользователя, и их
можно использовать для безопасного создания функций и процедур. Так как функции PL вы-
полняются внутри сервера баз данных, флаг TRUSTED следует устанавливать только для тех
языков, которые не позволяют обращаться к внутренним механизмам сервера или файловой
системе. Языки PL/pgSQL, PL/Tcl и PL/Perl считаются доверенными; языки PL/TclU, PL/PerlU и
PL/PythonU предоставляют неограниченную функциональность и их не следует помечать как
доверенные.
Примере 42.1 показывает, как выполняется процедура ручной установки для языка PL/Perl.
Пример 42.1. Установка PL/Perl вручную
Следующая команда говорит серверу баз данных, где найти разделяемый объект для функции-об-
работчика языка PL/Perl:
CREATE FUNCTION plperl_call_handler() RETURNS language_handler AS
'$libdir/plperl' LANGUAGE C;
Для PL/Perl реализованы обработчик внедрённого кода и функция проверки, так что их мы тоже
объявим:
CREATE FUNCTION plperl_inline_handler(internal) RETURNS void AS
'$libdir/plperl' LANGUAGE C;
CREATE FUNCTION plperl_validator(oid) RETURNS void AS
'$libdir/plperl' LANGUAGE C STRICT;
Следующая команда:
CREATE TRUSTED PROCEDURAL LANGUAGE plperl
HANDLER plperl_call_handler
INLINE plperl_inline_handler
VALIDATOR plperl_validator;
определяет, что ранее объявленные функции должны вызываться для функций и процедур с атри-
бутом языка plperl.
В стандартной инсталляции PostgreSQL обработчик языка PL/pgSQL уже собран и установлен в
каталог «библиотек»; более того, сам язык PL/pgSQL установлен во всех базах данных. Если при
сборке сконфигурирована поддержка Tcl, то обработчики для PL/Tcl и PL/TclU собираются и уста-
навливаются в каталог библиотек, но сам язык по умолчанию в базы данных не устанавливается.
Подобным образом, если сконфигурирована поддержка Perl, собираются и устанавливаются обра-
1122Процедурные языки
ботчики PL/Perl и PL/PerlU, а при включении поддержки Python устанавливается обработчик PL/
PythonU, но в базы данных эти языки по умолчанию не устанавливаются.
1123