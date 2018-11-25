---
layout: post
title: Устанавливаем PostgreSQL 11 из sources
description: "PostgreSQL install"
modified: 2018-11-25
tags: [PostgreSQL, Install]
image:
  path: /images/matrix.jpg
  feature: matrix.jpg
---

Загрузка sources из [PostgreSQL.org](https://www.postgresql.org/download/) и распаковываем в папку ~/install
```css
mkdir install
cd install
wget https://ftp.postgresql.org/pub/source/v11.1/postgresql-11.1.tar.gz
tar xzf postgresql-11.1.tar.gz
cd postgresql-11.1
```

Приступаем к установке, в случае ошибки придется установить недостающие библиотеки и инструменты.
```css
./configure
make world
make install
```
По умолчанию система развернется в папке */usr/local/pgsql* проверьте наличие файлов
```css
ls -l /usr/local/pgsql
```

Для работы PostgreSQL сервера требуется создать пользователя *postgres*

```css
sudo useradd -M postgres
```

Создаем папку для данных $PGDATA
```css
mkdir /usr/local/pgsql/data
chown postgres /usr/local/pgsql/data
```
Проводим инициализацию базы под пользователем postgres
```css
su - postgres
/usr/local/pgsql/bin/initdb -D /usr/local/pgsql/data
```

Запускаем демон PostgreSQL
```css
/usr/local/pgsql/bin/postgres -D /usr/local/pgsql/data >logfile 2>&1 &
```

Создаем тестовую базу *test*
```css
/usr/local/pgsql/bin/createdb test
/usr/local/pgsql/bin/psql test
  type for exit q\
```

Создаем скрипт автоматического запуска сервиса PostgreSQL
```css
sudo touch /etc/systemd/system/postgresql.service
Инаполнение файла
[Unit]
Description=PostgreSQL database server
After=network.target

[Service]
Type=forking

User=postgres
Group=postgres

# Maximum number of seconds pg_ctl will wait for postgres to start.  Note that
# PGSTARTTIMEOUT should be less than TimeoutSec value.
Environment=PGSTARTTIMEOUT=270

Environment=PGDATA=/usr/local/pgsql/data


ExecStart=/usr/local/pgsql/bin/pg_ctl start -D ${PGDATA} -s -w -t ${PGSTARTTIMEOUT}
ExecStop=/usr/local/pgsql/bin/pg_ctl stop -D ${PGDATA} -s -m fast
ExecReload=/usr/local/pgsql/bin/pg_ctl reload -D ${PGDATA} -s

# Give a reasonable amount of time for the server to start up/shut down.
# Ideally, the timeout for starting PostgreSQL server should be handled more
# nicely by pg_ctl in ExecStart, so keep its timeout smaller than this value.
TimeoutSec=300

[Install]
WantedBy=multi-user.target
```

Останавливаем демон PostgreSQL и перезапустим конфигурацию демона systemd:
```css
su postgres -c "/usr/local/pgsql/bin/pg_ctl -D /usr/local/pgsql/data/ stop"
systemctl daemon-reload
systemctl start postgresql
systemctl enable postgresql
```
Основной файл настроек Postgresql это postgresql.conf, в директории с данными /usr/local/pgsql/data/

```css
cat pg_hba.conf

# TYPE  DATABASE        USER            ADDRESS                 METHOD

# "local" is for Unix domain socket connections only
local   all             all                                     trust
# IPv4 local connections:
host    all             all             127.0.0.1/32            trust
# IPv6 local connections:
host    all             all             ::1/128                 trust
# Allow replication connections from localhost, by a user with the
# replication privilege.
local   replication     all                                     trust
host    replication     all             127.0.0.1/32            trust
host    replication     all             ::1/128                 trust
```

Рассмотрим возможные значения столбцов:

Тип подключения:
- local - подключение через доменные сокеты Unix;
- host - подключение по TCP/IP;
- hostssl - подключение по TCP/IP с применением шифрования SSL;
- hostnossl - противоположен hostssl, подключение по TCP/IP без шифрования SSL.

База данных - определяет к какой базе осуществляется подключение:
- Имя конкретной баззы данных;
- all - все базы данных;
- sameuser - определяет, что данная запись соответствует только, если имя запрашиваемой базы данных совпадает с именем запрашиваемого пользователя.
- samerole - определяет, что запрашиваемый пользователь должен быть членом роли с таким же именем, как и у запрашиваемой базы данных.
- Пользователь - учетная запись под которой происходит подключение к базе:
- Конкретное имя пользователя (несколько пользователей через запятую);
- all - любой пользователь;
- знак + перед именем пользователя - подключаемый пользователь имеет отношение к указанной роли;
- знак @ далее идет имя файла содержащего список пользователей.
- Адрес - адрес клиента, с которого происходит подключение:
- ip-адрес/маска (172.20.1.0/24, 172.20.1.1/32);
- all - любой хост;
- samehost - любой адрес данного сервера;
- samenet - любой адрес любой сети, к которой сервер подключен напрямую.

Метод аутентификации:
- trust - разрешает безусловное подключение;
- reject - отклоняет подключение;
- md5 - требует от клиента предоставить пароль;
- password - требуется для аутентификации клиентом с вводом незашифрованного пароля;
- ident - получает имя пользователя операционной системы клиента, связываясь с сервером Ident, и проверяет, соответствует ли оно имени пользователя базы данных. Аутентификация ident может использоваться только для подключений по TCP/IP;
- peer - получает имя пользователя операционной системы клиента из операционной системы и проверяет, соответствует ли оно имени пользователя запрашиваемой базы данных. Доступно только для локальных подключений;
- ldap - аутентификация с помощью сервера LDAP;
- radius - аутентификация с помощью сервера Radius;
- cert - аутентификация с помощью сертификата;
- pam - аутентификация с помощью службы подключаемых модулей аутентификации (PAM).
