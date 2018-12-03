---
layout: post
title: Глава 40. Триггеры событий
description: ""
tags: [PostgreSQL, PostgreSQL_Book_11]
image:
  feature: abstract-11.jpg
  #credit: dargadgetz
  #creditlink: http://www.dargadgetz.com/ios-7-abstract-wallpaper-pack-for-iphone-5-and-ipod-touch-retina/
share: true
modified: 2018-12-03 T15:14:43-04:00
---

Глава 40. Триггеры событий

В дополнение к триггерам, рассмотренным в Главе 39, PostgreSQL также предоставляет триггеры
событий. В отличие от обычных триггеров, которые подключаются к конкретной таблице и рабо-
тают только с командами DML, триггеры событий определяются на уровне базы данных и работа-
ют с командами DDL.
Как и обычные триггеры, триггеры событий можно создавать на любом процедурном языке, под-
держивающим триггеры событий, а также на C, но не на чистом SQL.
40.1. Обзор механизма работы триггеров событий
Триггер события срабатывает всякий раз, когда в базе данных, в которой он определён, про-
исходит связанное с ним событие. В настоящий момент поддерживаются следующие события:
ddl_command_start, ddl_command_end, table_rewrite и sql_drop. Поддержка дополнительных со-
бытий может быть добавлена в будущих выпусках.
Событие ddl_command_start происходит непосредственно перед выполнением команд CREATE,
ALTER, DROP, SECURITY LABEL, COMMENT, GRANT и REVOKE. Проверка на существование объекта пе-
ред срабатыванием триггера не производится. В качестве исключения, однако, это событие не
происходит для команд DDL, обращающихся к общим объектам кластера базы данных — ба-
зам данных, табличным пространствам, ролям, а также к самим триггерам событий. Событие
ddl_command_start также происходит непосредственно перед выполнением команды SELECT INTO,
так как она равнозначна команде CREATE TABLE AS.
Событие ddl_command_end происходит непосредственно после выполнения команд из того же на-
бора. Чтобы получить дополнительную информацию об операциях DDL, повлёкших произошедшее
событие, вызовите функцию pg_event_trigger_ddl_commands(), возвращающую множество, из ко-
да обработчика события ddl_command_end (см. Раздел 9.28). Заметьте, что этот триггер срабатыва-
ет после того, как эти действия имели место (но до фиксации транзакции), так что в системных
каталогах можно увидеть уже изменённое состояние.
Событие sql_drop происходит непосредственно перед событием ddl_command_end для команд, ко-
торые удаляют объекты базы данных. Для получения списка удалённых объектов используйте
возвращающую набор строк функцию pg_event_trigger_dropped_objects() в триггере события
sql_drop (см. Раздел 9.28). Обратите внимание, что триггер выполняется после удаления объектов
из таблиц системного каталога, поэтому их невозможно больше увидеть.
Событие table_rewrite происходит только после того, как таблица будет перезаписана в резуль-
тате определённых действий команд ALTER TABLE и ALTER TYPE. Хотя перезапись таблицы мо-
жет быть вызвана и другими управляющими операторами, в частности CLUSTER и VACUUM, событие
table_rewrite для них не вызывается.
Триггеры событий (как и прочие функции) не могут выполняться в прерванной транзакции. Поэто-
му, если команда DDL завершается ошибкой, соответствующие триггеры ddl_command_end не сра-
ботают. И наоборот, если триггер ddl_command_end завершился с ошибкой, последующие триггеры
событий не сработают, также как и сама команда не будет выполняться. Похожим образом, если
триггер ddl_command_end завершится ошибкой, действие команды DDL будет отменено, также как
это происходит при возникновении ошибки внутри транзакции.
Полный список команд, которые поддерживаются триггерами событий, можно найти в Разде-
ле 40.2.
Для создания триггера события используется команда CREATE EVENT TRIGGER. Предварительно
нужно создать функцию, со специальным возвращаемым типом event_trigger. Данная функция
не обязана возвращать значение (и может не возвращать). Возвращаемый тип служит лишь ука-
занием на то, что функция будет вызываться из триггера события.
Если есть несколько триггеров на одно и то же событие, то они будут вызываться в алфавитном
порядке по имени триггера.
1087Триггеры событий
В определении триггера можно использовать условие WHEN, чтобы, например, триггер
ddl_command_start срабатывал только для отдельных команд, которые нужно перехватить. Триг-
геры событий часто используются для ограничения диапазона DDL-команд, доступных пользова-
телям.
40.2. Матрица срабатывания триггеров событий
В Таблице 40.1 перечислены команды, для которых поддерживаются триггеры событий.
Таблица 40.1. Поддержка триггеров событий командами DDL
Тег команды
ddl_command_
start
ddl_command_
end
sql_drop
table_
rewrite
Замечания
ALTER
AGGREGATE X X - -  
ALTER
COLLATION X X - -  
ALTER
CONVERSION X X - -  
ALTER DOMAIN X X - -  
ALTER
EXTENSION X X - -  
ALTER FOREIGN
DATA WRAPPER X X - -  
ALTER FOREIGN
TABLE X X X -  
ALTER
FUNCTION X X - -  
ALTER
LANGUAGE X X - -  
ALTER
OPERATOR X X - -  
ALTER
OPERATOR
CLASS X X - -  
ALTER
OPERATOR
FAMILY X X - -  
ALTER POLICY X X - -  
ALTER SCHEMA X X - -  
ALTER
SEQUENCE X X - -  
ALTER SERVER X X - -  
ALTER TABLE X X X X  
ALTER TEXT
SEARCH
CONFIGURATION X X - -  
ALTER TEXT
SEARCH
DICTIONARY X X - -  
1088Триггеры событий
Тег команды
ddl_command_
start
ddl_command_
end
sql_drop
table_
rewrite
Замечания
ALTER TEXT
SEARCH PARSER X X - -  
ALTER TEXT
SEARCH
TEMPLATE X X - -  
ALTER TRIGGER X X - -  
ALTER TYPE X X - X  
ALTER USER
MAPPING X X - -  
ALTER VIEW X X - -  
CREATE
AGGREGATE X X - -  
COMMENT X X - - Только для
локальных
объектов
CREATE CAST X X - -  
CREATE
COLLATION X X - -  
CREATE
CONVERSION X X - -  
CREATE DOMAIN X X - -  
CREATE
EXTENSION X X - -  
CREATE
FOREIGN DATA
WRAPPER X X - -  
CREATE
FOREIGN TABLE X X - -  
CREATE
FUNCTION X X - -  
CREATE INDEX X X - -  
CREATE
LANGUAGE X X - -  
CREATE
OPERATOR X X - -  
CREATE
OPERATOR
CLASS X X - -  
CREATE
OPERATOR
FAMILY X X - -  
CREATE POLICY X X - -  
CREATE RULE X X - -  
CREATE SCHEMA X X - -  
1089Триггеры событий
Тег команды
ddl_command_
start
ddl_command_
end
sql_drop
table_
rewrite
Замечания
CREATE
SEQUENCE X X - -  
CREATE SERVER X X - -  
CREATE
STATISTICS X X - -  
CREATE TABLE X X - -  
CREATE TABLE
AS X X - -  
CREATE
TEXT SEARCH
CONFIGURATION X X - -  
CREATE
TEXT SEARCH
DICTIONARY X X - -  
CREATE TEXT
SEARCH PARSER X X - -  
CREATE
TEXT SEARCH
TEMPLATE X X - -  
CREATE
TRIGGER X X - -  
CREATE TYPE X X - -  
CREATE USER
MAPPING X X - -  
CREATE VIEW X X - -  
DROP
AGGREGATE X X X -  
DROP CAST X X X -  
DROP
COLLATION X X X -  
DROP
CONVERSION X X X -  
DROP DOMAIN X X X -  
DROP
EXTENSION X X X -  
DROP FOREIGN
DATA WRAPPER X X X -  
DROP FOREIGN
TABLE X X X -  
DROP FUNCTION X X X -  
DROP INDEX X X X -  
DROP LANGUAGE X X X -  
DROP OPERATOR X X X -  
1090Триггеры событий
Тег команды
ddl_command_
start
ddl_command_
end
sql_drop
table_
rewrite
Замечания
DROP OPERATOR
CLASS X X X -  
DROP OPERATOR
FAMILY X X X -  
DROP OWNED X X X -  
DROP POLICY X X X -  
DROP RULE X X X -  
DROP SCHEMA X X X -  
DROP SEQUENCE X X X -  
DROP SERVER X X X -  
DROP
STATISTICS X X X -  
DROP TABLE X X X -  
DROP TEXT
SEARCH
CONFIGURATION X X X -  
DROP TEXT
SEARCH
DICTIONARY X X X -  
DROP TEXT
SEARCH PARSER X X X -  
DROP TEXT
SEARCH
TEMPLATE X X X -  
DROP TRIGGER X X X -  
DROP TYPE X X X -  
DROP USER
MAPPING X X X -  
DROP VIEW X X X -  
GRANT X X - - Только для
локальных
объектов
IMPORT
FOREIGN
SCHEMA X X - -  
REVOKE X X - - Только для
локальных
объектов
SECURITY
LABEL X X - - Только для
локальных
объектов
SELECT INTO X X - -  
40.3. Триггерные функции событий на языке C
1091Триггеры событий
Этот раздел описывает низкоуровневые детали интерфейса для триггерной функции. Эта инфор-
мация необходима только при разработке триггерных функций событий на языке C. При исполь-
зовании языка более высокого уровня, эти детали обрабатываются автоматически. В большинстве
случаев необходимо рассмотреть использование процедурного языка прежде чем начать разраба-
тывать триггеры событий на C. В документации по каждому процедурному языку объясняется как
создавать триггеры событий на этом языке.
Триггерные функции событий должны использовать «version 1» интерфейса диспетчера функций.
Когда функция вызывается диспетчером триггеров событий, ей не передаются обычные аргумен-
ты, но передаётся указатель «context», ссылающийся на структуру EventTriggerData. Функции на
C могут проверить вызваны ли они диспетчером триггеров событий или нет выполнив макрос:
CALLED_AS_EVENT_TRIGGER(fcinfo)
который разворачивается в:
EventTriggerData
Если возвращается истина, то fcinfo->context можно безопасно привести к типу
EventTriggerData * и использовать указатель на структуру EventTriggerData. Функция не должна
изменять структуру EventTriggerData или любые данные, которые на неё указывают.
struct EventTriggerData определена в commands/event_trigger.h:
typedef struct EventTriggerData
{
NodeTag
type;
const char *event;
/* имя события */
Node
*parsetree; /* дерево разбора */
const char *tag;
/* тег команды */
} EventTriggerData;
со следующими членами структуры:
type
Всегда T_EventTriggerData.
event
Описывает
событие,
для
которого
вызывается
функция.
Возможные
значения:
"ddl_command_start", "ddl_command_end", "sql_drop", "table_rewrite". Суть этих событий
описывается в Разделе 40.1.
parsetree
Указатель на дерево разбора команды. Детали можно посмотреть в исходном коде PostgreSQL.
Структура дерева разбора может быть изменена без предупреждений.
tag
Тег команды, для которой сработал триггер события. Например "CREATE FUNCTION".
Функция триггера события должна возвращать указатель NULL (но не SQL значение null, то есть
не нужно устанавливать isNull в истину).
40.4. Полный пример триггера события
Вот очень простой пример функции для триггера события, написанной на C. (Примеры триггеров
для процедурных языков могут быть найдены в документации на процедурные языки.)
Функция noddl выдаёт ошибку при каждом вызове. Триггер с этой функцией определяется для
события ddl_command_start. Это предотвращает работу любых DDL-команд (за исключением тех,
о которых говорилось в Разделе 40.1).
Теперь исходный код триггерной функции:
1092Триггеры событий
#include "postgres.h"
#include "commands/event_trigger.h"
PG_MODULE_MAGIC;
PG_FUNCTION_INFO_V1(noddl);
Datum
noddl(PG_FUNCTION_ARGS)
{
EventTriggerData *trigdata;
if (!CALLED_AS_EVENT_TRIGGER(fcinfo)) /* internal error */
elog(ERROR, "not fired by event trigger manager");
trigdata = (EventTriggerData *) fcinfo->context;
ereport(ERROR,
(errcode(ERRCODE_INSUFFICIENT_PRIVILEGE),
errmsg("command \"%s\" denied", trigdata->tag)));
PG_RETURN_NULL();
}
После компиляции исходного кода (см. Подраздел 38.10.5) объявляем функцию и триггеры:
CREATE FUNCTION noddl() RETURNS event_trigger
AS 'noddl' LANGUAGE C;
CREATE EVENT TRIGGER noddl ON ddl_command_start
EXECUTE FUNCTION noddl();
Теперь проверим работу триггера:
=# \dy
List of event triggers
Name |
Event
| Owner | Enabled | Function | Tags
-------+-------------------+-------+---------+----------+------
noddl | ddl_command_start | dim
| enabled | noddl
|
(1 row)
=# CREATE TABLE foo(id serial);
ОШИБКА: command "CREATE TABLE" denied
В этой ситуации, для запуска DDL-команд, нужно либо удалить триггер события, либо отключить
его. Может быть удобным отключить триггер на время выполнения транзакции:
BEGIN;
ALTER EVENT TRIGGER noddl DISABLE;
CREATE TABLE foo (id serial);
ALTER EVENT TRIGGER noddl ENABLE;
COMMIT;
(Вспомним, что триггеры событий не обрабатывают DDL-команды для самих триггеров событий.)
40.5. Пример событийного триггера, обрабатывающего
перезапись таблицы
Благодаря существованию события table_rewrite, можно реализовать политику перезаписи таб-
лиц, допускающую перезапись только в определённое время обслуживания.
1093Триггеры событий
Следующий пример демонстрирует реализацию такой политики.
CREATE OR REPLACE FUNCTION no_rewrite()
RETURNS event_trigger
LANGUAGE plpgsql AS
$$
---
--- Реализация локальной политики перезаписи таблиц:
---
перезапись public.foo не допускается,
---
другие таблицы могут перезаписываться между 1 часом ночи и 6 часами утра,
---
если только их размер не превышает 100 блоков
---
DECLARE
table_oid oid := pg_event_trigger_table_rewrite_oid();
current_hour integer := extract('hour' from current_time);
pages integer;
max_pages integer := 100;
BEGIN
IF pg_event_trigger_table_rewrite_oid() = 'public.foo'::regclass
THEN
RAISE EXCEPTION 'you''re not allowed to rewrite the table %',
table_oid::regclass;
END IF;
SELECT INTO pages relpages FROM pg_class WHERE oid = table_oid;
IF pages > max_pages
THEN
RAISE EXCEPTION 'rewrites only allowed for table with less than % pages',
max_pages;
END IF;
IF current_hour NOT BETWEEN 1 AND 6
THEN
RAISE EXCEPTION 'rewrites only allowed between 1am and 6am';
END IF;
END;
$$;
CREATE EVENT TRIGGER no_rewrite_allowed
ON table_rewrite
EXECUTE FUNCTION no_rewrite();
1094
