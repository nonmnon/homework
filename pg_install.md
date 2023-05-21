#PostgreSqL
#Installation PostgreSqL

## Ubuntu Enable PostgreSQL Package Repository
-------------------
Ссылка на страничку загрузки ПО Postgresql

#Note https://www.postgresql.org/download/linux/ubuntu/ Lnux downloads (Ubuntu)


To use the apt repository, follow these steps:

### Create the file repository configuration:

`sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'`

### Import the repository signing key:

`wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -`

### Update the package lists:

`sudo apt-get update`

Example:

$ `sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list`

$ `wget -qO- https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo tee /etc/apt/trusted.gpg.d/pgdg.asc &>/dev/null`


## Install the latest version of PostgreSQL.
------------------------------------------------

If you want a specific version, use 'postgresql-12' or similar instead of 'postgresql':

$ `sudo apt-get -y install postgresql`      -- поставит версию Postgres "по усмолчанию" для текущей версии ОС

$ `sudo apt install postgresql-15 postgresql-client-15 -y`

 проверить что кластер Postgres  запущен

 $ `sudo -u postgres pg_lsclusters`
