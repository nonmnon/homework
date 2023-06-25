## Homework #5

## Домашнее задание
## Бэкапы Постгреса
- Делам бэкап Постгреса используя WAL-G или pg_probackup и восстанавливаемся на другом кластере



- Задание повышенной сложности*
под нагрузкой*
бэкап снимаем с реплики**

+++++++++++++++++++++++++++++++++++++

--Создан экземпляр PostgreSQl version 15

gmfcbkaccnt@vmex4:~$ `sudo -u postgres pg_lsclusters`

Ver Cluster Port Status Owner    Data directory              Log file

15  main    5432 online postgres /var/lib/postgresql/15/main /var/log/postgresql/postgresql-15-main.l

postgres=# select version();
  

 PostgreSQL 15.3 (Debian 15.3-1.pgdg110+1) on x86_64-pc-linux-gnu, compiled by gcc (Debian 10.2.1-6) 10.2.1 20210110, 64-bit
(1 row)

--Создана тестовая БД bench
postgres=# `\l`

                                             List of databases

   Name    |  Owner   | Encoding | Collate |  Ctype  | ICU Locale | Locale Provider |   Access privileges   

-----------+----------+----------+---------+---------+------------+-----------------+-----------------------

 bench     | postgres | UTF8     | C.UTF-8 | C.UTF-8 |            | libc            | 

 postgres  | postgres | UTF8     | C.UTF-8 | C.UTF-8 |            | libc            | 

 template0 | postgres | UTF8     | C.UTF-8 | C.UTF-8 |            | libc            | =c/postgres          +

           |          |          |         |         |            |                 | postgres=CTc/postgres

 template1 | postgres | UTF8     | C.UTF-8 | C.UTF-8 |            | libc            | =c/postgres          +

           |          |          |         |         |            |                 | postgres=CTc/postgres

postgres=# `SELECT pg_size_pretty(pg_database_size('bench'));`

 pg_size_pretty 

\----------------

 2323 MB

(1 row)`

--Устанавливаем пакеты WAL-G

$ `cd /var/lib/postgresql/distr`

$ `wget https://github.com/wal-g/wal-g/releases/download/v2.0.1/wal-g-pg-ubuntu-20.04-amd64.tar.gz && tar -zxvf wal-g-pg-ubuntu-20.04-amd64.tar.gz && sudo mv wal-g-pg-ubuntu-20.04-amd64 /usr/local/bin/wal-g`

postgres@vmex4:~/distr$ `sudo ls -lrt /usr/local/bin/wal-g`
-rwxr-xr-x 1 postgres postgres 43903416 Aug 25  2022 /usr/local/bin/wal-g

--Создаем директорию для бакапов на диск
postgres@vmex4:~/distr$` mkdir backup`

postgres@vmex4:~/distr$ `ls -lrtd backup`
drwxr-xr-x 2 postgres postgres 4096 Jun 12 19:38 backup

-- Создаем файл конфигурации для wal-g

postgres@vmex4:~/distr$ `pg_lsclusters`

postgres@vmex4:~$ `pwd`

/var/lib/postgresql

postgres@vmex4:~$ `id`

uid=109(postgres) gid=116(postgres) groups=116(postgres),115(ssl-cert)

postgres@vmex4:~$  `pg_lsclusters`

Ver Cluster Port Status Owner    Data directory              Log file
15  main    5432 online postgres /var/lib/postgresql/15/main /var/log/postgresql/postgresql-15-main.log

postgres=`show data_directory;`

data_directory        
\-----------------------------
 /var/lib/postgresql/15/main
(1 row)

$ cat > /var/lib/postgresql/.walg.json << EOF

{

    "WALG_FILE_PREFIX": "/backup",
    "WALG_COMPRESSION_METHOD": "brotli",
    "WALG_DELTA_MAX_STEPS": "5",
    "PGDATA": "/var/lib/postgresql/15/main",
    "PGHOST": "/var/run/postgresql/.s.PGSQL.5432"
}
EOF

--Директория для log файлов

$ `mkdir /var/lib/postgresql/15/main/log`

--Правим конфигурационные файлы PostgreSQL

postgres=\# `show archive_command;`

archive_command

\-----------------

 (disabled)
(1 row)



$ `echo "wal_level=replica" >> /var/lib/postgresql/15/main/postgresql.auto.conf`

$`echo "archive_mode=on" >> /var/lib/postgresql/15/main/postgresql.auto.conf`

$`echo "archive_command='wal-g wal-push \"%p\" >> /var/lib/postgresql/15/main/log/archive_command.log 2>&1' " >> /var/lib/postgresql/15/main/postgresql.auto.conf `

$`echo "archive_timeout=60" >> /var/lib/postgresql/15/main/postgresql.auto.conf `

$ `echo "restore_command='wal-g wal-fetch \"%f\" \"%p\" >> /var/lib/postgresql/15/main/log/restore_command.log 2>&1' " >> /var/lib/postgresql/15/main/postgresql.auto.conf`

-- Применяем новые параметры конфиг файла 

$ `killall -s HUP postgres`

Либо перезапускаем кластер PostgreSQL
`pg_ctlcluster 15 main stop`

`pg_ctlcluster 15 main start` 

postgres=\# `show archive_command;`

archive_command                                 
\---------------------------------------------------------------------------------

 wal-g wal-push "%p" >> /var/lib/postgresql/15/main/log/archive_command.log 2>&1
(1 row)

--Подключаемся к тестовой БД bench

postgres=# \c bench

You are now connected to database "bench" as user "postgres".


bench=# \dt

              List of relations

 Schema |       Name       | Type  |  Owner   

--------+------------------+-------+----------

 public | pgbench_accounts | table | postgres

 public | pgbench_branches | table | postgres

 public | pgbench_history  | table | postgres

 public | pgbench_tellers  | table | postgres

(4 rows)

--Создаем таблицу заполненную значениями текущего времени

bench-# `CREATE TABLE t_table(created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW());`

bench-# do $$

declare 

   counter integer := 0;

begin

   while counter < 10 loop

 INSERT INTO t_table(created_at) VALUES (CURRENT_TIMESTAMP);

 perform pg_sleep(60);

 counter := counter + 1;

   end loop;

end$$;

bench=# select * from t_table;

          created_at          

------------------------------

 2023-06-12 22:10:46.17891+00

 2023-06-12 22:10:46.17891+00

 2023-06-12 22:10:46.17891+00

 2023-06-12 22:10:46.17891+00

 2023-06-12 22:10:46.17891+00

 2023-06-12 22:10:46.17891+00

 2023-06-12 22:10:46.17891+00

 2023-06-12 22:10:46.17891+00

 2023-06-12 22:10:46.17891+00

 2023-06-12 22:10:46.17891+00

(10 rows)

--Делаем бэкап

$ `wal-g backup-push /var/lib/postgresql/15/main `

![wal-g backup](/img/walg_1.png)
 
  -- проверка лога выполнения wal-g
  ![wal-g log](/img/walg_2.png)

--смотрим backup-list

postgres@vmex4:~$ wal-g backup-list

name                          modified             wal_segment_backup_start
base_000000010000000B00000076 2023-06-12T22:23:28Z 000000010000000B00000076

--Дозаплолним таблицу новыми данными
postgres@vmex4:~$bench=#  select * from t_table;

          created_at           

-------------------------------

 2023-06-12 22:10:46.17891+00

 2023-06-12 22:10:46.17891+00

 2023-06-12 22:10:46.17891+00

 2023-06-12 22:10:46.17891+00

 2023-06-12 22:10:46.17891+00

 2023-06-12 22:10:46.17891+00

 2023-06-12 22:10:46.17891+00

 2023-06-12 22:10:46.17891+00

 2023-06-12 22:10:46.17891+00

 2023-06-12 22:10:46.17891+00

 2023-06-13 15:31:35.769292+00

 2023-06-13 15:31:35.769292+00

 2023-06-13 15:31:35.769292+00

 2023-06-13 15:31:35.769292+00

 2023-06-13 15:31:35.769292+00

 2023-06-13 15:31:35.769292+00

 2023-06-13 15:31:35.769292+00

 2023-06-13 15:31:35.769292+00

 2023-06-13 15:31:35.769292+00

 2023-06-13 15:31:35.769292+00

(20 rows)

--Запустим процедуру бакапирования повторно, создадим "дельта" бэкап
![wal-g extra backap](/img/walg_3.png)

--Проверим наличие резервных копий

postgres@vmex4:~$ `wal-g backup-list`

postgres@vmex4:~$ `wal-g wal-show`
![wal-g backup-list](/img/walg_4.png)

--Копируем файлы бакапов и wal файлы на host vmex3, где будет провеедено восстановление

postgres@vmex4:~$ du -hs /backup/

191M    /backup/

postgres@vmex4:~$ `scp -r /backup/* postgres@10.164.0.3:/backup`

postgres@vmex3:~$ ip address|grep inet

    inet 127.0.0.1/8 scope host lo

    inet6 ::1/128 scope host 

    inet 10.164.0.3/32 scope global dynamic ens4

    inet6 fe80::4001:aff:fea4:3/64 scope link

postgres@vmex3:~$ `du -hs /backup/`

191M    /backup/

--Останавливаем существующий экземпляр Postgres, удаляем каталог с текущей БД

postgres@vmex3:~$ `pg_lsclusters`

Ver Cluster Port Status Owner    Data directory              Log file

15  main    5432 online postgres /var/lib/postgresql/15/main /var/log/postgresql/postgresql-15-main.log

postgres@vmex3:~$ `pg_ctlclaster 15 main stop`

--Удаляем существующую БД
postgres@vmex3:~$ `mv /var/lib/postgresql/15/main /var/lib/postgresql/15/main_w`
postgres@vmex3:~$ `mkdir /var/lib/postgresql/15/main`

--Устанавливаем ПО wal-g на хоста vmex3

![wal-g install](/img/walg_5.png)

--Cоздаем конфигурационные файлы wal-g

$ cat > /var/lib/postgresql/.walg.json << EOF

{

    "WALG_FILE_PREFIX": "/backup",
    "WALG_COMPRESSION_METHOD": "brotli",
    "WALG_DELTA_MAX_STEPS": "5",
    "PGDATA": "/var/lib/postgresql/15/main",
    "PGHOST": "/var/run/postgresql/.s.PGSQL.5432"
}
EOF

--Проверяем, что доступны скопированные с vmex4 файлы бэкапов
![wal-g install](/img/walg_5.png)


--Восстанавливаем файлы БД на последний в наличии 
$  `wal-g backup-fetch /var/lib/postgresql/15/main LATEST`

![wal-g backup-feetch](/img/walg_7.png)

--создаем файл для восстановления из архивов wal
$ touch "/var/lib/postgresql/15/main/recovery.signal"


--запкускаем БД
$ pg_ctlcluster 15 main start

-проверяем, что восстановление прошло успешно

![wal-g backup-feetch](/img/walg_8.png)

--создаем бакет в CGP 

Connecting Google Cloud Storage to SimpleBackups https://docs.simplebackups.com/storage-providers/ghsg5GE15AMwMo1qFjUCXn/google-cloud-storage/qYCmhYSxmquPEH3ARwRTnV


Best practices for managing service account keys https://cloud.google.com/iam/docs/best-practices-for-managing-service-account-keys?_ga=2.74477394.-1741265027.1686753580



bucket-otus-lab_5

gsutil URI gs://bucket-otus-lab_5

Default storage class: Standard
Location: europe-west4 (Netherlands)
Location type: Region
Public access prevention: On
Access control: Uniform
Protection tools: None
Data encryption: Google-managed key
Price europe-west4 (Netherlands)	
$0.020 per GB-month

-- Настроим использование CGP bucket для wal-g
postgres@vmex4:~$ cat .walg.json

{

    "WALG_FILE_PREFIX": "gs://bucket-otus-lab_5/walg_backup",

      "GOOGLE_APPLICATION_CREDENTIALS" : "/var/lib/postgresql/otus-385420-b7251068ebb8.json",

    "WALG_COMPRESSION_METHOD": "brotli",

    "WALG_DELTA_MAX_STEPS": "5",

    "PGDATA": "/var/lib/postgresql/15/main",

    "PGHOST": "/var/run/postgresql/.s.PGSQL.5432"

}

postgres@vmex4:~$ touch otus-385420-a7b558aa0f35.json
postgres@vmex4:~$ vi otus-385420-a7b558aa0f35.json
postgres@vmex4:~$ vi .walg.json
postgres@vmex4:~$ gsutil cp touchf.txt  gs://bucket-otus-lab_5/walg_backup/ 
Copying file://touchf.txt [Content-Type=text/plain]...
/ [1 files][    4.0 B/    4.0 B]                                                
Operation completed over 1 objects/4.0 B.                                        
postgres@vmex4:~$ wal-g backup-push /var/lib/postgresql/15/main
ERROR: 2023/06/18 18:45:13.529771 failed to configure folder: FS error : Folder not exists or is inaccessible: stat gs://bucket-otus-lab_5/walg_backup/: no such file or directory


В журнале :
![wal-g БД log](/img/walg_10.png)

--!!Бакет от CGP подключить не удалось.

По совету Евгениия  пеересмотрел видео, но там показывается готовый бакет, деталей по настройке нет.

--Выполнение дполнительной практики задание без использования бакета.

--Создаем реплику БД на хосте VMEX5 (Standby) для БД на хосте VMEX4(Primary).
/*
vmex5 vmex4 root gfhbvnTRY546 

vmex5 vmex4 postgres  cde3$RFVbgt5

vmex4.europe-west4-b.c.otus-385420.internal

 vmex5 10.164.0.6
 vmex4 10.164.0.5
 */

-- Для пользователем postgres выполняем проброс ssh ключей между primary на standby серверами:
На Primary

$ ssh-keygen -t rsa
$ touch ~/.ssh/authorized_keys
$ chmod 600 ~/.ssh/authorized_keys
$ ssh-copy-id -f postgres@10.164.0.6

На Standby
$ sudo -u root echo 'PasswordAuthentication yes' >> /etc/ssh/sshd_config

$ sudo systemctl restart sshd

$ ssh-keygen -t rsa
$ touch ~/.ssh/authorized_keys
$ chmod 600 ~/.ssh/authorized_keys
$ ssh-copy-id postgres@10.164.0.5


-- Добавляем слот репликации на Primary:
/*
select * from pg_replication_slots;
select pg_create_physical_replication_slot('slot_a');
*/

postgres=# select  pg_create_physical_replication_slot('slot_a');

 pg_create_physical_replication_slot 

-------------------------------------

 (slot_a,)


postgres=# `select * from pg_replication_slots\gx`

-[ RECORD 1 ]-------+---------

slot_name           | slot_a

plugin              | 

slot_type           | physical

datoid              | 

database            | 

temporary           | f

active              | f

active_pid          | 

xmin                | 

catalog_xmin        | 

restart_lsn         | 

confirmed_flush_lsn | 

wal_status          | 

safe_wal_size       | 

two_phase           | f


-- Создаем специального пользователя в БД для репликации и удаляем права на репликацию
(REPLICATION) у superuser'а postgres с целью обеспечения безопасности.


CREATE ROLE replicauser WITH REPLICATION LOGIN;

ALTER ROLE postgres NOREPLICATION;


-- Добавляем в файл pg_hba.conf на Primary и на Standby разрешение на доступ пользователя replicauser\
между хостами:\
env=postgres\

$ vi pg_hba.conf
#Replication beetween hosts\
host replication replicauser 10.164.0.6/32 trust\
host replication replicauser 10.164.0.5/32 trust\
host all replicauser 10.164.0.6/32 trust\
host all replicauser 10.164.0.5/32 trust

-- На Standby импортируем резервную копию БД с Primary сервера:
postgres@vmex5:~$ `mv /var/lib/postgresql/15/main /var/lib/postgresql/15/main.w `
postgres@vmex5:~$` mkdir /var/lib/postgresql/15/main`


postgres@vmex5:~$ `pg_basebackup -D /var/lib/postgresql/15/main -P -R -U replicauser -h 10.164.0.5 --slot=slot_a`

2760902/2760902 kB (100%), 1/1 tablespace

postgres@vmex5:~$ `du -hs /var/lib/postgresql/15/main `
2.7G    /var/lib/postgresql/15/main

--Проверяемя что слот репликациии на Primary активен
postgres=# `select * from pg_replication_slots \gx`

-[ RECORD 1 ]-------+-----------

slot_name           | slot_a

plugin              | 

slot_type           | physical

datoid              | 

database            | 

temporary           | f

active              | f

active_pid          | 

xmin                | 

catalog_xmin        | 

restart_lsn         | B/D1000000

confirmed_flush_lsn | 

wal_status          | reserved

safe_wal_size       | 

two_phase           | f


--Меняем конфиг файл  postgresql.conf на Standby сервере

$ echo "standby_mode = 'on'" >> /etc/postgresql/15/main/postgresql.conf

$ echo "primary_conninfo = 'user=replicauser host=10.164.0.5  port=5432 sslmode=prefer sslcompression=1 krbsrvname=postgres target_session_attrs=any'" >> /etc/postgresql/15/main/postgresql.conf

$ echo "primary_slot_name = 'slot_a'" >>/etc/postgresql/15/main/postgresql.conf

--Запустили БД Standby
postgres@vmex5:~$ `rm -rf /var/lib/postgresql/15/main/*`

postgres@vmex5:~$ `pg_basebackup -D /var/lib/postgresql/15/main -P -R -U 
replicauser -h 10.164.0.5 --slot=slot_a`

2760904/2760904 kB (100%), 1/1 tablespace

В журнале БД Standby экзепляр поднялся:

![wal-g БД log](/img/walg_16.png)



На Primary БД создали табличку t и заполнили
![wal-g Primary](/img/walg_17.png)

На Standby БД провеирили - репликация работает.

![wal-g standby](/img/walg_18.png)

-- Устанавливамем wal_g на реплике vmex5

root@vmex5:~# echo "postgres  ALL=(ALL) ALL" | sudo tee /etc/sudoers.d/postgres
postgres  ALL=(ALL) ALL

![wal-g standby](/img/walg_19.png)

--Настраиваем конфигурацию

$ `echo "wal_level=replica" >> /var/lib/postgresql/15/main/postgresql.auto.conf`

$`echo "archive_mode=on" >> /var/lib/postgresql/15/main/postgresql.auto.conf`

$`echo "archive_command='wal-g wal-push \"%p\" >> /var/lib/postgresql/15/main/log/archive_command.log 2>&1' " >> /var/lib/postgresql/15/main/postgresql.auto.conf `

$`echo "archive_timeout=60" >> /var/lib/postgresql/15/main/postgresql.auto.conf `

$ `echo "restore_command='wal-g wal-fetch \"%f\" \"%p\" >> /var/lib/postgresql/15/main/log/restore_command.log 2>&1' " >> /var/lib/postgresql/15/main/postgresql.auto.conf`

![wal-g standby](/img/walg_20.png)


-- Запускаем бакап на реплике:
![wal-g standby](/img/walg_21.png)

--Запускаем бакап Standby под нагрузкой Primary создание БД для pg_bench

![wal-g Primary](/img/walg_22.png)


Бакап wal-g отработал на Standby БД во время репликации данных с Primary, ошибок не выявлено.

В последующем, при попытке выполнить бакап завершается с ошибкой:  Finish LSN of backup base_000000010000000C000000A4_D_000000010000000C00000046 greater than current LSN, что говорит вероятно, что нет еще данных для дельта бакапа.
![wal-g Primary](/img/walg_23.png)


https://github.com/wal-g/wal-g/issues/1089