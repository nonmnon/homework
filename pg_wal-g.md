## PostgreSQL
### WAL-G 

You can use wal-g as a tool for making encrypted, compressed PostgreSQL backups (full and incremental) and push/fetch them to/from remote storages without saving it on your filesystem.

-- wal-g v1.1.1 - 2022

-- Current version wal-g v2.0.1 

-- [Installation](https://github.com/wal-g/wal-g#installation)

$  `sudo rm /usr/local/bin/wal-g`

$  `cd /tmp_dir`

$  `wget https://github.com/wal-g/wal-g/releases/download/v2.0.1/wal-g-pg-ubuntu-20.04-amd64.tar.gz && tar -zxvf wal-g-pg-ubuntu-20.04-amd64.tar.gz && sudo mv wal-g-pg-ubuntu-20.04-amd64 /usr/local/bin/wal-g`


-- [Configuration](https://github.com/wal-g/wal-g/blob/master/docs/PostgreSQL.md) 

There are two ways:

 - Using environment variables

- Using a config file

--config /path flag can be used to specify the path where the config file is located. 
 Config file support every format that the [viper](https://github.com/spf13/vipe) package supports: JSON, YAML, envfile and others.

PGHOST can connect over a UNIX socket. This mode is *preferred* for localhost connections, set PGHOST=/var/run/postgresql to use it. WAL-G will connect over TCP if PGHOST is an IP address.

 Set environment
 PGHOST, PGPORT, PGUSER, and PGPASSWORD/PGPASSFILE/~/.pgpass.

\#Note

1.[ Артём Картасов. Над пропастью WAL-G. Video](https://www.youtube.com/watch?v=oLhn85CDfXY)

2.[ Разгоняем бэкап – Андрей Бороди. Video](https://www.youtube.com/watch?v=bXuN4Na0cEo)

3.[ Advanced PostgreSQL backup & recovery methods. Video](https://cds.cern.ch/record/2706980)

4.[ Viper GitHub](https://github.com/spf13/viper)

5.[ Introducing WAL-G by Citus: Faster Disaster Recovery for Postgres](https://www.citusdata.com/blog/2017/08/18/introducing-wal-g-faster-restores-for-postgres/)

6.[]()
