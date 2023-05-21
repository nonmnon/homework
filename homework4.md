## Home work #4

- **Развернуть Постгрес на ВМ**
  ##### *Создаем VM*
  ![CreateVM](img/hw4_createVM.png)

#### *Устанавливаем PostgreSQL v15*

gmfcbkaccnt@vmex4:~$ `uname -a`

Linux vmex4 5.10.0-22-cloud-amd64 #1 SMP Debian 5.10.178-3 (2023-04-22) x86_64 GNU/Linux

gmfcbkaccnt@vmex4:~$ `sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list' `

gmfcbkaccnt@vmex4:~$ `wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add - 

Warning: apt-key is deprecated. Manage keyring files in trusted.gpg.d instead (see apt-key(8)).

OK

gmfcbkaccnt@vmex4:~$  ` sudo apt-get update`

Get:1 http://apt.postgresql.org/pub/repos/apt bullseye-pgdg InRelease [117 kB]

 
*Установка ПО PostgreSQL v15 из репозитария*

gmfcbkaccnt@vmex4:~$ `sudo apt install postgresql-15 postgresql-client-15 -y`

Reading package lists... Done

Building dependency tree... Done

Reading state information... Done

The following additional packages will be installed:

fixing permissions on existing directory /var/lib/postgresql/15/main ... ok

creating subdirectories ... ok

selecting dynamic shared memory implementation ... posix

selecting default max_connections ... 100

selecting default shared_buffers ... 128MB

selecting default time zone ... Etc/UTC

creating configuration files ... ok

running bootstrap script ... ok

performing post-bootstrap initialization ... ok

syncing data to disk ... ok

Setting up libjson-xs-perl (4.030-1+b1) ...

Processing triggers for man-db (2.9.4-2) ...

Processing triggers for libc-bin (2.31-13+deb11u6) ...

*Проверка что кластер запущен*

gmfcbkaccnt@vmex4:~$ `sudo -u postgres pg_lsclusters`

Ver Cluster Port Status Owner    Data directory              Log file

15  main    5432 online postgres /var/lib/postgresql/15/main /var/log/postgresql/postgresql-15-main.log

- **Протестировать pg_bench**

В этом тесте используем VM GCP c 4 ГБ RAM, 2 vCPU, 10 ГБ дискового пространства, управляемую базу данных (только ведущая нода). 

- **Выставить оптимальные настройки**
- **Проверить насколько выросла производительность**
- **Настроить кластер на оптимальную производительность не обращая внимания на стабильность БД**



