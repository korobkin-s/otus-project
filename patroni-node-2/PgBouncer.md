Установить PgBouncer:

```
sudo apt update && sudo apt install pgbouncer -y ;\
sudo mv /etc/pgbouncer/pgbouncer.ini{,.original} ;\

echo '"postgres" "postgres"' | sudo tee /etc/pgbouncer/userlist.txt > /dev/null ;\

sudo sed -i '/\[Service\]/a LimitNOFILE=15000' /lib/systemd/system/pgbouncer.service ;\

sudo systemctl daemon-reload ;\
sudo systemctl restart pgbouncer
```

Создать конфигурационный файл `pgbouncer.ini` и перезапустить демон:
```
echo '
[databases]
postgres = host=127.0.0.1 port=5432 dbname=postgres
* = host=127.0.0.1 port=5432

[pgbouncer]
logfile = /var/log/postgresql/pgbouncer.log
pidfile = /var/run/postgresql/pgbouncer.pid
listen_addr = *
listen_port = 6432
unix_socket_dir = /var/run/postgresql
auth_type = md5
auth_file = /etc/pgbouncer/userlist.txt
admin_users = postgres
ignore_startup_parameters = extra_float_digits,geqo
pool_mode = session
server_reset_query = DISCARD ALL
max_client_conn = 10000
default_pool_size = 800
reserve_pool_size = 150
reserve_pool_timeout = 1
max_db_connections = 1000
pkt_buf = 8192
listen_backlog = 4096
log_connections = 0
log_disconnections = 0
' | sudo tee /etc/pgbouncer/pgbouncer.ini > /dev/null ;\

sudo systemctl restart pgbouncer
```
