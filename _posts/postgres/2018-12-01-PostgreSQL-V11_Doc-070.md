---
layout: post
title: Глава 70. Как планировщик использует статистику
description: ""
tags: [PostgreSQL, PostgreSQL_Book_11]
image:
  feature: abstract-11.jpg
  #credit: dargadgetz
  #creditlink: http://www.dargadgetz.com/ios-7-abstract-wallpaper-pack-for-iphone-5-and-ipod-touch-retina/
share: true
modified: 2018-12-03 T15:14:43-04:00
---

Глава 70. Как планировщик использует статистику


Данная глава основана на материалах, рассмотренных ранее (см. Раздел 14.1 и Раздел 14.2), и по-
дробнее рассказывает о том, как планировщик использует статистику для определения количества
строк, которое может вернуть каждая часть запроса. Это важная составляющая процесса создания
плана запроса, предоставляющая большую часть исходного материала для расчёта стоимости.
Целью данной главы является не подробное документирование кода, а общее описание его работы.
Возможно, это поможет тем, кто пожелает в дальнейшем ознакомиться с кодом.
70.1. Примеры оценки количества строк
В приведённых ниже примерах используются таблицы базы данных регрессионного тестирования
PostgreSQL. Приведённые листинги получены в версии 8.3. Поведение более ранних (или поздних)
версий может отличаться. Заметьте также, что поскольку команда ANALYZE использует случайную
выборку при формировании статистики, после любого нового выполнения команды ANALYZE ре-
зультаты незначительно изменятся.
Давайте начнём с очень простого запроса:
EXPLAIN SELECT * FROM tenk1;
QUERY PLAN
-------------------------------------------------------------
Seq Scan on tenk1 (cost=0.00..458.00 rows=10000 width=244)
Как планировщик определяет мощность tenk1, рассматривается выше (см. Раздел  14.2), но для
полноты здесь говорится об этом ещё раз. Количество страниц и строк берётся в pg_class:
SELECT relpages, reltuples FROM pg_class WHERE relname = 'tenk1';
relpages | reltuples
----------+-----------
358 |
10000
Это текущие цифры, полученные при последнем выполнении команд VACUUM или ANALYZE, при-
менённых к этой таблице. Затем планировщик выполняет выборку фактического текущего числа
страниц в таблице (это недорогая операция, для которой не требуется сканирование таблицы). Ес-
ли оно отличается от relpages, то reltuples изменяется для того, чтобы привести это значение
к текущей оценке количества строк. В показанном выше примере значение relpages является ак-
туальным, поэтому количество строк берётся равным reltuples.
Давайте обратимся к примеру с диапазонным условием в предложении WHERE:
EXPLAIN SELECT * FROM tenk1 WHERE unique1 < 1000;
QUERY PLAN
--------------------------------------------------------------------------------
Bitmap Heap Scan on tenk1 (cost=24.06..394.64 rows=1007 width=244)
Recheck Cond: (unique1 < 1000)
-> Bitmap Index Scan on tenk1_unique1 (cost=0.00..23.80 rows=1007 width=0)
Index Cond: (unique1 < 1000)
Планировщик рассматривает условие предложения WHERE и находит в справочнике функцию из-
бирательности для оператора < в pg_operator. Это значение содержится в столбце oprrest, и в
данном случае значением является scalarltsel. Функция scalarltsel извлекает гистограмму для
unique1 из pg_statistic. Для вводимых вручную запросов удобнее просматривать более простое
представление pg_stats:
2189Как планировщик ис-
пользует статистику
SELECT histogram_bounds FROM pg_stats
WHERE tablename='tenk1' AND attname='unique1';
histogram_bounds
------------------------------------------------------
{0,993,1997,3050,4040,5036,5957,7057,8029,9016,9995}
Затем обрабатывается часть гистограммы, которая соответствует условию «< 1000». Таким обра-
зом и определяется избирательность. Гистограмма делит диапазон на равные частотные группы,
поэтому нужно лишь определить группу, содержащую наше значение, и подсчитать её долю и
долю групп, предшествующих данной. Очевидно, что значение 1000 находится во второй группе
(993-1997). Если предположить, что внутри каждой группы распределение значений линейное, мы
можем вычислить избирательность следующим образом:
selectivity = (1 + (1000 - bucket[2].min)/(bucket[2].max - bucket[2].min))/num_buckets
= (1 + (1000 - 993)/(1997 - 993))/10
= 0.100697
т. е. сумма элементов одной целой группы и пропорциональной части элементов второй, делённая
на число групп. Теперь примерное число строк может быть рассчитано как произведение избира-
тельности и мощности tenk1:
rows = rel_cardinality * selectivity
= 10000 * 0.100697
= 1007 (округлённо)
Далее, давайте рассмотрим пример с условием на равенство в предложении WHERE:
EXPLAIN SELECT * FROM tenk1 WHERE stringu1 = 'CRAAAA';
QUERY PLAN
----------------------------------------------------------
Seq Scan on tenk1 (cost=0.00..483.00 rows=30 width=244)
Filter: (stringu1 = 'CRAAAA'::name)
Планировщик вновь проверяет условие в предложении WHERE и определяет функцию избиратель-
ности для =, и этой функцией является eqsel. Для оценки равенства гистограмма бесполезна, вме-
сто неё для оценки избирательности используется список наиболее распространённых значений
(Most Commom Values, MCV). Давайте рассмотрим MCV и соответствующие дополнительные столб-
цы, которые пригодятся позже:
SELECT null_frac, n_distinct, most_common_vals, most_common_freqs FROM pg_stats
WHERE tablename='tenk1' AND attname='stringu1';
null_frac
| 0
n_distinct
| 676
most_common_vals |
{EJAAAA,BBAAAA,CRAAAA,FCAAAA,FEAAAA,GSAAAA,JOAAAA,MCAAAA,NAAAAA,WGAAAA}
most_common_freqs | {0.00333333,0.003,0.003,0.003,0.003,0.003,0.003,0.003,0.003,0.003}
Так как значение CRAAAA оказалось в списке MCV, избирательность будет определяться просто
соответствующим элементом в списке частот наиболее распространённых значений (Most Common
Frequencies, MCF):
selectivity = mcf[3]
= 0.003
Как и в предыдущем примере, оценка числа строк берётся как произведение мощности и избира-
тельности tenk1:
rows = 10000 * 0.003
= 30
Теперь рассмотрим тот же самый запрос, но с константой, которой нет в списке MCV:
2190Как планировщик ис-
пользует статистику
EXPLAIN SELECT * FROM tenk1 WHERE stringu1 = 'xxx';
QUERY PLAN
----------------------------------------------------------
Seq Scan on tenk1 (cost=0.00..483.00 rows=15 width=244)
Filter: (stringu1 = 'xxx'::name)
Это совершенно другая задача — как оценить избирательность значения, которого нет в списке
MCV. При её решении используется факт отсутствия данного значения в списке в сочетании с
частотой для каждого значения из списка MCV.
selectivity = (1 - sum(mvf))/(num_distinct - num_mcv)
= (1 - (0.00333333 + 0.003 + 0.003 + 0.003 + 0.003 + 0.003 +
0.003 + 0.003 + 0.003 + 0.003))/(676 - 10)
= 0.0014559
Т. е. нужно сложить частоты значений из списка MCV, отнять полученное число от единицы, и
полученное значение разделить на количество остальных уникальных значений. Эти вычисления
основаны на предположении, что значения, которые не входят в список MCV, имеют равномерное
распределение. Заметьте, что в данном примере нет неопределённых значений, поэтому о них
беспокоиться не нужно (иначе их долю также пришлось бы вычитать из числителя). Оценка числа
строк затем производится как обычно:
rows = 10000 * 0.0014559
= 15 (округлённо)
Предыдущий пример с unique1 < 1000 был большим упрощением того, что в действительности
делает scalarltsel. Но после того, как мы увидели пример использования списка MCV, мы можем
внести некоторые дополнения. Что касается самого примера, в нём все было правильно, поскольку
unique1 это уникальный столбец, у него нет значений в списке MCV (очевидно, в данном случае
нет значения, которое является более распространённым, чем любое другое). Для неуникального
столбца обычно создаётся как гистограмма, так и список MCV, при этом гистограмма не включает
значения, представленные в списке MCV. Данный способ позволяет выполнить более точный под-
счёт. В этой ситуации scalarltsel напрямую применяет условие «< 1000» к каждому значению
списка MCV и суммирует частоты значений MCV, для которых условие является верным. Это даёт
точную оценку избирательности для той части таблицы, которая содержит значения из списка
MCV. Подобным же образом используется гистограмма для оценки избирательности для той части
таблицы, которая не содержит значения из списка MCV, а затем эти две цифры складываются для
оценки общей избирательности. Например, рассмотрим
EXPLAIN SELECT * FROM tenk1 WHERE stringu1 < 'IAAAAA';
QUERY PLAN
------------------------------------------------------------
Seq Scan on tenk1 (cost=0.00..483.00 rows=3077 width=244)
Filter: (stringu1 < 'IAAAAA'::name)
Мы уже видели данные списка MCV для stringu1, а это его гистограмма:
SELECT histogram_bounds FROM pg_stats
WHERE tablename='tenk1' AND attname='stringu1';
histogram_bounds
--------------------------------------------------------------------------------
{AAAAAA,CQAAAA,FRAAAA,IBAAAA,KRAAAA,NFAAAA,PSAAAA,SGAAAA,VAAAAA,XLAAAA,ZZAAAA}
Проверяя список MCV, находим, что условие stringu1 < 'IAAAAA' соответствует первым шести
записям, но не соответствует последним четырём, поэтому избирательность для значений, соот-
ветствующих значениям в списке MCV, такова:
selectivity = sum(relevant mvfs)
= 0.00333333 + 0.003 + 0.003 + 0.003 + 0.003 + 0.003
= 0.01833333
2191Как планировщик ис-
пользует статистику
Сумма всех частот из списка MCF также сообщает нам, что общая часть представленной списком
MCV совокупности записей равняется 0.03033333, и поэтому представленная гистограммой часть
равняется 0.96966667 (в этом случае тоже нет неопределённых значений, иначе их пришлось бы
также исключить). Видно, что значение IAAAAA попадает почти в конец третьего столбца гисто-
граммы. Основываясь на простых предположениях относительно частоты различных символов,
планировщик получает число 0.298387 для части значений, представленных в гистограмме, кото-
рые меньше чем IAAAAA. Затем объединяем оценки части значений из списка MCV и значений, не
содержащихся в нём:
selectivity = mcv_selectivity + histogram_selectivity * histogram_fraction
= 0.01833333 + 0.298387 * 0.96966667
= 0.307669
rows
= 10000 * 0.307669
= 3077 (округлённо)
В этом конкретном примере, корректировка со стороны списка MCV достаточно мала, потому что
распределение значений столбца довольно плоское (статистика, показывающая конкретные зна-
чения как более распространённые, чаще всего получается вследствие статистической погреш-
ности). В более типичном случае, когда некоторые значения являются значительно более распро-
странёнными по сравнению с другими, этот более сложный метод повышает точность вследствие
точного определения избирательности наиболее распространённых значений.
Теперь давайте рассмотрим случай с более чем одним условием в предложении WHERE:
EXPLAIN SELECT * FROM tenk1 WHERE unique1 < 1000 AND stringu1 = 'xxx';
QUERY PLAN
--------------------------------------------------------------------------------
Bitmap Heap Scan on tenk1 (cost=23.80..396.91 rows=1 width=244)
Recheck Cond: (unique1 < 1000)
Filter: (stringu1 = 'xxx'::name)
-> Bitmap Index Scan on tenk1_unique1 (cost=0.00..23.80 rows=1007 width=0)
Index Cond: (unique1 < 1000)
Планировщик исходит из того, что два условия независимы, таким образом, отдельные значения
избирательности можно перемножить:
selectivity = selectivity(unique1 < 1000) * selectivity(stringu1 = 'xxx')
= 0.100697 * 0.0014559
= 0.0001466
rows
= 10000 * 0.0001466
= 1 (округлённо)
Заметьте, что число строк, которые предполагается вернуть через сканирование битового индекса,
соответствует условию, используемому при работе индекса; это важно, так как влияет на оценку
стоимости для последующих выборок из таблицы.
В заключение исследуем запрос, выполняющий соединение:
EXPLAIN SELECT * FROM tenk1 t1, tenk2 t2
WHERE t1.unique1 < 50 AND t1.unique2 = t2.unique2;
QUERY PLAN
--------------------------------------------------------------------------------------
Nested Loop (cost=4.64..456.23 rows=50 width=488)
-> Bitmap Heap Scan on tenk1 t1 (cost=4.64..142.17 rows=50 width=244)
Recheck Cond: (unique1 < 50)
-> Bitmap Index Scan on tenk1_unique1 (cost=0.00..4.63 rows=50 width=0)
Index Cond: (unique1 < 50)
-> Index Scan using tenk2_unique2 on tenk2 t2 (cost=0.00..6.27 rows=1 width=244)
2192Как планировщик ис-
пользует статистику
Index Cond: (unique2 = t1.unique2)
Ограничение, накладываемое на tenk1, unique1 < 50, производится до соединения вложенным
циклом. Это обрабатывается аналогично предыдущему примеру с диапазонным условием. На этот
раз значение 50 попадает в первый столбец гистограммы unique1:
selectivity = (0 + (50 - bucket[1].min)/(bucket[1].max - bucket[1].min))/num_buckets
= (0 + (50 - 0)/(993 - 0))/10
= 0.005035
rows
= 10000 * 0.005035
= 50 (округлённо)
Ограничение для соединения следующее t2.unique2 = t1.unique2. Здесь используется уже из-
вестный нам оператор =, однако функцию избирательности получаем из столбца oprjoin представ-
ления pg_operator, и эта функция — eqjoinsel. Функция eqjoinsel находит статистические дан-
ные как для tenk2, так и для tenk1:
SELECT tablename, null_frac,n_distinct, most_common_vals FROM pg_stats
WHERE tablename IN ('tenk1', 'tenk2') AND attname='unique2';
tablename | null_frac | n_distinct | most_common_vals
-----------+-----------+------------+------------------
tenk1
|
0 |
-1 |
tenk2
|
0 |
-1 |
В этом случае нет данных MCV для unique2, потому что все значения будут уникальными. Таким
образом, используется алгоритм, зависящий только от числа различающихся значений для обеих
таблиц и от данных с неопределёнными значениями:
selectivity = (1 - null_frac1) * (1 - null_frac2) * min(1/num_distinct1, 1/
num_distinct2)
= (1 - 0) * (1 - 0) / max(10000, 10000)
= 0.0001
Т. е., вычитаем долю неопределённых значений из единицы для каждой таблицы и делим на мак-
симальное из чисел различающихся значений. Количество строк, которое соединение, вероятно,
сгенерирует, вычисляется как мощность декартова произведения двух входных значений, умно-
женная на избирательность:
rows = (outer_cardinality * inner_cardinality) * selectivity
= (50 * 10000) * 0.0001
= 50
Если бы имелись списки MCV для двух столбцов, функцией eqjoinsel использовалось бы прямое
сравнение со списками MCV для определения общей избирательности той части данных, которая
содержит значения списка MCV. Оценка остальной части данных при этом выполнялась бы пред-
ставленным выше способом.
Заметьте, что здесь выводится для inner_cardinality значение 10000, то есть исходный размер
tenk2. Если изучить вывод EXPLAIN, может показаться, что оценка количества строк вычисляется
как 50 * 1, то есть число внешних строк умножается на ориентировочное число строк, получаемых
при каждом внутреннем сканировании индекса в tenk2. Но это не так, ведь размер результата
соединения оценивается до того, как выбирается конкретный план соединения. Если всё работает
корректно, оба варианта вычисления этого размера должны давать один и тот же ответ, но из-за
ошибок округления и других факторов иногда они значительно различаются.
Для интересующихся более подробной информацией: оценка размера таблицы (до выполнения
условий в предложении WHERE) реализована в файле src/backend/optimizer/util/plancat.c.
Основная логика для вычисления избирательности предложений находится в src/backend/
optimizer/path/clausesel.c. Специфичные для отдельных операторов функции избирательности,
в основном, расположены в src/backend/utils/adt/selfuncs.c.
2193Как планировщик ис-
пользует статистику
70.2. Примеры многовариантной статистики
70.2.1. Функциональные зависимости
Многовариантную корреляцию можно продемонстрировать на очень простом наборе данных —
таблице с двумя столбцами, содержащими одинаковые значения:
CREATE TABLE t (a INT, b INT);
INSERT INTO t SELECT i % 100, i % 100 FROM generate_series(1, 10000) s(i);
ANALYZE t;
Как рассказывается в Разделе 14.2, планировщик может определить мощность t, исходя из числа
страниц и строк, полученного из pg_class:
SELECT relpages, reltuples FROM pg_class WHERE relname = 't';
relpages | reltuples
----------+-----------
45 |
10000
Распределение данных очень простое: в каждом столбце содержится всего 100 различных значе-
ний, равномерно распределённых.
Следующий пример показывает результат оценивания условия WHERE по столбцу a:
EXPLAIN (ANALYZE, TIMING OFF) SELECT * FROM t WHERE a = 1;
QUERY PLAN
-------------------------------------------------------------------------------
Seq Scan on t (cost=0.00..170.00 rows=100 width=8) (actual rows=100 loops=1)
Filter: (a = 1)
Rows Removed by Filter: 9900
Планировщик рассматривает условие и определяет, что его избирательность равна 1%. Сравнивая
эту оценку и фактическое число строк, мы видим, что оценка очень точна (на самом деле абсолют-
на точна, так как таблица очень маленькая). Если изменить условие WHERE, чтобы использовался
столбец b, будет получен такой же план. Но посмотрите, что получится, если мы применим оди-
наковое условие к двум столбцам, объединив их оператором AND:
EXPLAIN (ANALYZE, TIMING OFF) SELECT * FROM t WHERE a = 1 AND b = 1;
QUERY PLAN
-----------------------------------------------------------------------------
Seq Scan on t (cost=0.00..195.00 rows=1 width=8) (actual rows=100 loops=1)
Filter: ((a = 1) AND (b = 1))
Rows Removed by Filter: 9900
Планировщик оценивает избирательность каждого условия индивидуально, и получает ту же оцен-
ку в 1%, что и выше. Затем он предполагает, что условия независимы, так что он перемножает
избирательности и выдаёт окончательную оценку избирательности, равную всего 0.01%. Это зна-
чительная недооценка, так как фактическое число строк, соответствующих условию, (100) на два
порядка больше.
Эту проблему можно решить, создав объект статистики, который укажет команде ANALYZE вычис-
лить многовариантную статистику функциональной зависимости по двум столбцам:
CREATE STATISTICS stts (dependencies) ON a, b FROM t;
ANALYZE t;
EXPLAIN (ANALYZE, TIMING OFF) SELECT * FROM t WHERE a = 1 AND b = 1;
QUERY PLAN
-------------------------------------------------------------------------------
Seq Scan on t (cost=0.00..195.00 rows=100 width=8) (actual rows=100 loops=1)
Filter: ((a = 1) AND (b = 1))
Rows Removed by Filter: 9900
2194Как планировщик ис-
пользует статистику
70.2.2. Многовариантное число различных значений
Подобная проблема возникает с оценкой мощности наборов с несколькими столбцами, например,
с оценкой числа групп, которые могут быть выданы предложением GROUP BY. Когда в GROUP BY
указан один столбец, оценка числа различных значений (которую можно увидеть как ожидаемое
число строк, выдаваемое узлом HashAggregate) очень точная:
EXPLAIN (ANALYZE, TIMING OFF) SELECT COUNT(*) FROM t GROUP BY a;
QUERY PLAN
-----------------------------------------------------------------------------------------
HashAggregate (cost=195.00..196.00 rows=100 width=12) (actual rows=100 loops=1)
Group Key: a
-> Seq Scan on t (cost=0.00..145.00 rows=10000 width=4) (actual rows=10000
loops=1)
Но оценка числа групп в запросе с двумя столбцами в GROUP BY без многовариантной статистики,
как и в предыдущем примере, отличается от правильной на порядок:
EXPLAIN (ANALYZE, TIMING OFF) SELECT COUNT(*) FROM t GROUP BY a, b;
QUERY PLAN
--------------------------------------------------------------------------------------------
HashAggregate (cost=220.00..230.00 rows=1000 width=16) (actual rows=100 loops=1)
Group Key: a, b
-> Seq Scan on t (cost=0.00..145.00 rows=10000 width=8) (actual rows=10000
loops=1)
Если переопределить объект статистики, чтобы он включал подсчёт числа различных значений
для двух столбцов, оценка станет гораздо лучше:
DROP STATISTICS stts;
CREATE STATISTICS stts (dependencies, ndistinct) ON a, b FROM t;
ANALYZE t;
EXPLAIN (ANALYZE, TIMING OFF) SELECT COUNT(*) FROM t GROUP BY a, b;
QUERY PLAN
--------------------------------------------------------------------------------------------
HashAggregate (cost=220.00..221.00 rows=100 width=16) (actual rows=100 loops=1)
Group Key: a, b
-> Seq Scan on t (cost=0.00..145.00 rows=10000 width=8) (actual rows=10000
loops=1)
70.3. Статистика планировщика и безопасность
Доступ к таблице pg_statistic разрешён только суперпользователям, так что обычные пользо-
ватели не могут получить из неё сведения о содержимом таблиц других пользователей. Но неко-
торые функции оценки избирательности будут использовать пользовательский оператор (опера-
тор, фигурирующий в запросе, или связанный) для анализа сохранённой статистики. Например,
чтобы определить применимость сохранённого самого частого значения, функция оценки избира-
тельности должна задействовать соответствующий оператор = для сравнения константы в запро-
се с этим сохранённым значением. Таким образом, данные pg_statistic в принципе могут пере-
даваться пользовательским операторам. А особым образом сконструированный оператор может
выводить наружу передаваемые ему операнды преднамеренно (например, записывая их в журнал
или помещая в другую таблицу) либо непреднамеренно (показывая их значения в сообщениях об
ошибках). В любом случае это даёт возможность пользователю, не имеющему доступа к таблице
pg_statistic, увидеть содержащиеся в ней данные.
Для предотвращения этого все встроенные функции оценки избирательности действуют по следу-
ющим правилам. Чтобы сохранённая статистика могла использоваться при планировании запроса,
текущий пользователь должен иметь либо право SELECT для таблицы или задействованных столб-
2195Как планировщик ис-
пользует статистику
цов, либо у оператора должна быть характеристика LEAKPROOF (точнее, она должна быть у функции,
реализующей этот оператор). В противном случае оценка избирательности будет осуществляться
так, как если бы статистики не было вовсе, и планировщик продолжит работу с общими или вто-
ричными предположениями.
Если пользователь не имеет требуемого права доступа к таблице или столбцам, то во многих слу-
чаях при выполнении запроса в конце концов возникнет ошибка «нет доступа», так что этот меха-
низм будет незаметен на практике. Но если пользователь читает данные из представления с ба-
рьером безопасности, планировщик может захотеть проверить статистику нижележащей таблицы,
которая недоступна пользователю непосредственно. В этом случае оператор должен быть герме-
тичным; иначе статистика не будет использоваться. Это не будет иметь внешних проявлений кро-
ме того, что план запроса может быть неоптимальным. В случае подозрений, что вы столкнулись с
этим, попробуйте запустить запрос от имени пользователя с расширенными правами и проверьте,
не выбирается ли другой план запроса.
Это ограничение применяется только тогда, когда планировщику может потребоваться выполнить
пользовательский оператор с одним или несколькими значениями из pg_statistic. При этом пла-
нировщику разрешено использовать общую статистическую информацию, например, процент зна-
чений NULL или количество различных значений в столбце, вне зависимости от прав доступа.
Реализуемые в дополнительных расширениях функции оценки избирательности, которые могут
обращаться к статистике, вызывая пользовательские операторы, должны следовать тем же прави-
лам безопасности. За практическими указаниями обратитесь к исходному коду PostgreSQL.
2196
