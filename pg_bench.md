
\#PostgreSQL \#perfomance tools
## Pgbench

*TPS – это показатель пропускной способности базы данных, который показывает количество транзакций, обработанных базой данных за одну секунду.*

**Environment**
PGDATABASE
PGHOST
PGPORT
PGUSER
PG_COLOR 

**Synopsis**

pgbench -i [option...] [dbname]

pgbench [option...] [dbname]

**Проверить** 

параметр PostgreSQL max_connections может ограничивать количество разрешенных клиентских подключений

\#Note 

  1. [ТЕСТИРОВАНИЕ ПРОИЗВОДИТЕЛЬНОСТИ УПРАВЛЯЕМОЙ БАЗЫ ДАННЫХ POSTGRESQL С ПОМОЩЬЮ PGBENCH](https://www.8host.com/blog/testirovanie-proizvoditelnosti-upravlyaemoj-bazy-dannyx-postgresql-s-pomoshhyu-pgbench/)
  2. [pgbench official documentation PostgreSQL 15](https://www.postgresql.org/docs/15/pgbench.html)
  3. [pgbench документация PostgrePro ](https://postgrespro.ru/docs/postgrespro/15/pgbench)