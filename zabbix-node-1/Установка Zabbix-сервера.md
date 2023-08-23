
Установить Apache и пакеты php:

```
sudo apt update && sudo apt upgrade -y ;\
sudo apt install -y apache2 php php-mysql php-mysqlnd php-ldap php-bcmath php-mbstring php-gd php-pdo php-xml libapache2-mod-php ;\
sudo systemctl enable apache2 && systemctl restart apache2
```

Установить PostgreSQL:

```
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list' && wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add - && sudo apt-get update && sudo apt install postgresql-15 postgresql-contrib-15 -y
```


Установить Zabbix:

```
wget -P /tmp https://repo.zabbix.com/zabbix/6.4/ubuntu/pool/main/z/zabbix-release/zabbix-release_6.4-1+ubuntu22.04_all.deb ;\
dpkg -i /tmp/zabbix-release_6.4-1+ubuntu22.04_all.deb ;\
sudo apt update ;\
sudo apt install zabbix-server-pgsql zabbix-frontend-php php8.1-pgsql zabbix-apache-conf zabbix-sql-scripts zabbix-agent -y
```

Создать исходную БД и импортировать схему

```
sudo -u postgres createuser --pwprompt zabbix
sudo -u postgres createdb -O zabbix zabbix

zcat /usr/share/zabbix-sql-scripts/postgresql/server.sql.gz | sudo -u zabbix psql zabbix
```


В файле /etc/zabbix/zabbix_server.conf задать пароль пользователя БД:

```
DBPassword=my_password
```


Запустить сервер Zabbix и процессы агента

```
sudo systemctl restart zabbix-server zabbix-agent ;\
sudo systemctl enable zabbix-server zabbix-agent ;\
sudo service apache2 restart
```



**Локализация интерфейса Zabbix для русского языка:**

В файле /etc/locale.gen раскомментировать:

```
ru_RU ISO-8859-5
ru_RU.CP1251 CP1251
ru_RU.KOI8-R KOI8-R
ru_RU.UTF-8 UTF-8
```

Далее выполнить:

```
locale-gen ;\
sudo service apache2 restart
```



