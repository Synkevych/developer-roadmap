# PostgreSQL commands and tips

## Explain Analyze

- Используется для проверки скорости выполнения запроса и показа выбраного постгресом метода для поиска по указаномым параметрам.

```sh
explain analyze select * from pg_class;
```

**Seq Scan** - самая простая операция из всех возможных – PostgreSQL открывает файл с таблицей, читает строки одну за другой и возвращает их пользователю или расположенному выше узлу дерева explain.

**Index Scan** - открывает индекс; в индексе, если находит, где (в данных таблицы) могут быть строки, соответствующие данному условию: открывает таблицу; получает строку(-и), указанную(-ые) индексом;
если строки могут быть возвращены – то есть, если они видимы в текущей сессии – они возвращаются.

**Index Only Scan** - этот тип сканирования говорит о postgres что происходит выборка данных (колонок) только из индекса и все нужные данные могут быть получени из таблицы индекса. Для выполнения этой операции обязательно чтобы данные строки не подвергались изменениям в последнее время. (Если включен autovacuum такой проблемы не должно быть).

```sh
explain select id from test_indexing where id = 2;
 Index Only Scan using idx_id on test_indexing  (cost=0.43..8.45 rows=1 width=4)
   Index Cond: (id = 2)
(2 rows)
```

**Bitmap Index Scan** - всегда состоит минимум из 2 узлов. Сначала идет Bitmap Index Scan а потом Bitmap Heap Scan. Bitmap Index Scan создает битовую карту вашей таблицы 0, или 1 если на странице находится нужная строка. После этого результат передается к узлу Bitmap Heap Scan который читает их в более последовательно  манере. Имея несколько битовых карт можно легко производить логические операции типа OR, AND или Not.

**\di+** команда выводит размер индексных таблиц

**\dt+** команда показывает размер таблиц

**corellation** - последовательность записи данных, если например индексы записаны не последовательно а в случайном порядке, такая ситуация приводит к ухутшению производительности

```sh
select tablename, attname, correlation
from pg_stats where tablename in ('t_test', 't_random') order by 1,2;

 tablename | attname | correlation
-----------+---------+-------------
 t_random  | id      | 0.016970534
 t_random  | name    |           1
(2 rows)
```

**Cluster command** - позволяет кластиризировать таблицу на основе информации из индехной таблицы. При этом порядок новых записей будет прежним, система не пытается автоматически сохранять порядок новых или изменненых строк в соответствии с индексом. Во время кластеризации таблица блокируется для всех других операций.

```sh
cluster t_random using idx_random;
```

**Partial Indexes** позволяет создавать индех например на записях с определенными значением булевого поля или определённого значения перечисления (enum)

```sh
drop index idx_name;
create index idx_name on test_indexing (name)
where name not in ('bob','alice');
```

**Fill Factor** - определяет степень заполнености страницы (блоков данных) таблиц на диске и то, сколько нужно оставить свободного места в блоках для записи туда изменённых версий кортежей. Таблицы в которых записи не изменяются никогда(или крайне редко) смысла менять fill factor нет, что есть правильно (для индексов fill factor уже по умолчанию равен 90)

**pg_stat_statement** - показывает типы запросов которые медленные и как часто эти запросы выполняются.shared_preload_libraries = 'pg_stat_statement'

## Commands

`systemctl status postgresql@12-main` - shows errors if they presented in starting service

`pg_lsclusters` - show all available clusters their port and data directory;

`systemctl start postgresql@12-main` - start postgres version 12;

`sudo -i -u postgres` - login with user postgres;

`sudo vim /etc/postgresql/12/main/postgresql.conf` for changing configuration file;
