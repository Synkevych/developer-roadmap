# PostgreSQL commands and tips

## Explain Analyze

- Используется для проверки скорости выполнения запроса и показа выбраного постгресом метода для поиска по указаномым параметрам.

```sql
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

**corellation** - последовательность записи данных, если например индексы записаны не последовательно а в случайном порядке, такая ситуация приводит к ухутшению производительности

**index cardinality** - последовательность записи данных, если например индексы записаны не последовательно а в случайном порядке, такая ситуация приводит к ухутшению производительности

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

`sudo -u postgres psql database_name` - connect ot database by user postgres  

`sudo -i -u postgres` - login with user postgres;

`systemctl status postgresql@12-main` - shows errors if they presented in starting service

`pg_lsclusters` - show all available clusters their port and data directory;

`systemctl start postgresql@12-main` - start postgres version 12;

`sudo vim /etc/postgresql/12/main/postgresql.conf` for changing configuration file;

### How to use of Index efficiency

`createdb test_indexing`

`psql test_indexing`

`create table test_indexing( id serial, name text);`

`insert into test_indexing (name) select 'bob' from generate_series(1,2500000);`

`select * from test_indexing where id = 4;`

`explain analyze select * from test_indexing where id = 4;`

`create index idx_id on test_indexing (id);`

`create index idx_name on test_indexing (name);`

`create table test_index2 (id serial primary key, value integer);`

`\di+` show size of indexes 

`\dt+` show size of table

### Uderstanding Vacuum

При выполнеии операции удалении записей из таблицы они только помечаються как удаленные но база даных не может использовать их ячейки.
**Vacuum** процесс помечает такие ечейки как доступные для использования. 
**Vacuum Full** удаляет польностью данные и фрагментирует данные таблицы

```sql
create table vacuum_test(id int) with (autovacuum_enabled = off); # create new table with disabled vacuum

select pg_size_pretty(pg_relation_size('vacuum_test')); # show size of current table

select * from vacuum_test order by id DESC limit 10; # get last 10 records from database

update vacuum_test set id = id + 1; # update table
```

### Scalability and replication

Vartical Scaling (increase CPU, RAM, ROM), not code changes is necessary
Horizontal scaling (increase mashines), add servers gradually as per requirements 

```sql
insert into test_index2(value)
select trunc (random() * 10)
from generate_series (1,100000);

create index idx_value on test_index2 (value);
explain analyze select count(*) from test_index2 where value = 1; # bitmap Heap  scan 

explain select id from test_indexing where id = 1; # used index only scan

create index idx_xy on table(x) include(y); # useful for index only scan
```

**index on foreing key** - improves joins performance, improve performance when making changes to the parent table, 

**index cardinality** - таблица с большим количеством повторяющихся записей имеет низкую мощность(name), и наоборот если повторяющихся значений нет то высокую(id)

```sql
create table items(
item_no serial not null, order_no integer, product_name varchar,
descr varchar, created_ts timestamp,
constraint fk_items foreign key (order_no)
references orders (order_no)
match simple on update cascade on delete cascade);

with order_rws as ( 
insert into orders (order_no, order_date)
select generate_series(1, 1000000) t, now()
returning order_no)
insert into items (item_no, order_no, product_name, descr, created_ts)
select generate_series(1,4) item_no, order_no, 'product',
repeat('the description of the product' , 10), now()
from order_rws;

select * from orders ord join items itm on ord.order_no = itm.order_no 
where ord.order_no = 12;

create index idx_ord on items (order_no);
```

### Statistics

`pg_stat_statements` - which types of queries are slow and how often these queries are called

## Questions

### Загальні 

- Для чого потрібні індекси?
Швидкий пошук данних на основі значення в цих таблицях, цілісність даних, прискорення виконання певних дій.
- Які є види індексів? 
кластеризовані - відсортовані значення з даними, некластирезовані, складові, унікальні, покриваючі
- Чи прискорюють індекси всі операції з базами?
- Що таке транзакція? 
Об'єднує послідовно декілька дій в одну операцію "все або нічого" і забезпечує атомарність операції, BEGIN - COMMIT
- Що таке DB view?
- Чим відрізняються materialized db view від non-materialized db view?
- Як можна зберегти дані в різні таблиці та гарантувати, що всі вони або запишуться, або ні?
- Чи можна будувати індекси за кількома полями? Чи важливий порядок цих полів в індексу?
- Які ви знаєте constraints під час створення стовпців?
- У чому різниця між SQL та NoSQL базами даних?
В типах даних, цілісності даних, різних техніках збереження та управління даними 
- Як би ви імпортували великі масиви даних у БД (1–2 мільйони рядків у CSV-файл)?
Обгорнувши операцію в одну транзакцію.
- Що таке N+1 та як уникати?

### Реляційні бази даних

- Які відмінності між джоінами FULL OUTER JOIN, CROSS JOIN, NATURAL JOIN, INTERSECT та EXPECT?
- Які специфічні типи даних є в PostgreSQL?
- Що таке view? З якою метою використовується?
- Що таке materialized view?
- Що таке recursive view?
- Що таке збережена процедура і навіщо вона потрібна?
- Що таке партиціонування та яку проблему воно вирішує?
- Чи вмієте працювати з чистими SQL-запитами?
- Яким чином можна працювати з геолокацією в PostgreSQL?
- Які є способи резервного копіювання даних? Що таке pg_dump? У якому вигляді можна створювати резервні копії?

### PostgreSQL

- Що таке **sharding**

Sharding - або горизонтальне розділення бази даних, це принцип проектування, згідно з яким рядки таблиці бази даних зберігаються окремо, а не за стовпцями (як при нормалізації). Кожен розділ є частиною сегмента, який і може перебувати на окремому сервері бази даних або в фізичному місцезнаходження. Перевага полягає в тому, що кількість рядків в кожній таблиці зменшується (це зменшує розмір індексу, що підвищує продуктивність пошуку). Якщо сегментування засноване на деякому реальному аспекті даних (наприклад, європейські клієнти проти американських клієнтів), то можна легко і автоматично вивести відповідне членство в окреий фрагмент і запросити тільки його.

- Що таке транзакція

**Транзакція** - група послідовних операцій в базі даних яка представляє собою логічну одницю роботи з даними.

- Що таке WAL? 

**WAL (write-ahead log, журнал попереднього запису)** - стандартний метод забезпечення цілісності даних, спосіб покращення та відновлення даних після збоїв. Файли - бінарні логи які можна прочитити за допомогою pg_xlog_dump.

- Типи індексів в PostgreSQL

Індекс – це структура даних, що дозволяє зменшити час доступу до окремих рядків таблиць, незалежно від їхнього фізичного розміщення. Наявність індексу може суттєво підвищити швидкість виконання деяких запитів, однак індекси мають оновлюватися системою при кожному внесенні змін у базову таблицю, що створює додаткове навантаження на систему. Типи індексів: **B-дерево, хеш, GiST, SP-GiST, GIN и BRIN**. За замовчуванням команда CREATE INDEX створює індекси типу B-дерево які ефективні в більшості випадків.

### Масштабирование базы данных

Нужно для того чтобы справлятся с нагрузкой. Перед тем, как приступать к масштабированию, необходимо провести анализ медленных запросов и убедиться, что сервер базы настроен оптимально.


Реплікація - механізм синхронізації вмісту декількох копій об'єкта, під цим процесом розуміється копіювання даних з одного джерела на інший і навпаки. При реплікації зміни, зроблені в одній копії об'єкта, можуть бути поширені в інші копії. Существуют такие техники для решении этой проблемы:

- Master-Slave позволяет использовать два или больше одинаковых серверов вместо одного. В такой комбинации для записи новых данных следует использовать Master, а для чтения Slave. За счет того что данные намного чаще читаються чем пишуться есть возможность использования даной техники.

- Репликация - создает полный дубликат базы данных, и у вас вместо одного сервера их будет несколько. Но также у вас появляется проблема по облуживанию этих серверов и актуализации данных. Некоторые базы (SQLite) не позволяют создать репликации. 

- Шардинг (иногда шардирование) — это другая техника масштабирования работы с данными. Суть его в разделении (партиционирование) базы данных на отдельные части, так, чтобы каждую из них можно было вынести на отдельный сервер. Даная техника позволяет разбить таблицу на несколько разных физически данных, и увеличить скорость обработки за счет того что часто вам не нужны все данные с таблицы а только несколько определенных значений, и выборка их из отдельной таблицы будет быстрее. 

### Відновлення даних з дампу в PostgreSQL

```sql
drop database gisdata;
create database gisdata;
\c gisdata;
create extension postgis;
psql -h localhost -U jrodos -d gisdata < gisdata.sql -p 5432

psql -h localhost -d Prognose -U jrodos -p 5432
psql -U postgres -W

```

```sh
/usr/lib/postgresql/12/bin/createdb # create db script location

sudo apt remove postgresql-12
sudo apt-get remove --purge application
psql -h localhost -U jrodos -d gisdata < gisdata.sql -p 5432
```

<details>
<summary><b>Resources:</b></summary>
<br>

> [Postgres vs MongoDB | Олег Бартунов](https://youtu.be/SNzOZKvFZ68)

</details>