Установить PostgreSQL
```
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list' && wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add - && sudo apt-get update && sudo apt install postgresql-15 postgresql-contrib-15 -y ;\
sudo systemctl disable postgresql --now ;\
sudo rm -rf /var/lib/postgresql/15/main/*
```

Установить Patroni
```
sudo apt install -y python3 python3-pip python3-psycopg2 && \
pip3 install patroni[consul] && \
sudo mkdir /etc/patroni
```

Создать конфигурационный файл `patroni.yml`

```
echo '
name: patroni-node-2
scope: postgres

watchdog:
  mode: off

consul:
  host: "localhost:8500"
  register_service: true
  #token: <consul-acl-token>

restapi:
  listen: 0.0.0.0:8008
  connect_address: "patroni-node-2:8008"
  auth: 'patroni:patroni'

bootstrap:
  dcs:
    ttl: 30
    loop_wait: 10
    maximum_lag_on_failover: 1048576
    postgresql:
      use_pg_rewind: true
      use_slots: true
      parameters:
        archive_mode: "on"
        wal_level: hot_standby
        max_wal_senders: 10
        wal_keep_segments: 8
        archive_timeout: 1800s
        max_replication_slots: 5
        hot_standby: "on"
        wal_log_hints: "on"
      pg_hba:
        - local all all trust
        - host replication replicator 192.168.1.100/32 trust
        - host replication replicator 192.168.2.100/32 trust
        - host replication replicator 192.168.3.100/32 trust
        - host replication replicator 127.0.0.1/32 trust
        - host all all 0.0.0.0/0 scram-sha-256
    

initdb:
  - encoding: UTF8
  - data-checksums

postgresql:
  pgpass: /var/lib/postgresql/15/.pgpass
  listen: 0.0.0.0:5432
  connect_address: "patroni-node-2:5432"
  data_dir: /var/lib/postgresql/15/main/
  bin_dir: /usr/lib/postgresql/15/bin/
  pg_rewind:
    username: postgres
    password: password
  replication:
    username: replicator
    password: replicator
  superuser:
    username: postgres
    password: postgres
' | sudo tee /etc/patroni/patroni.yml > /dev/null
```


Создать/запустить демон `patroni.service`
```
echo '
[Unit]
Description=Patroni service
After=syslog.target network.target

[Service]
Type=simple
User=postgres
Group=postgres
ExecStart=/usr/local/bin/patroni /etc/patroni/patroni.yml
ExecReload=/bin/kill -s HUP $MAINPID
KillMode=process
TimeoutSec=30
Restart=no

[Install]
WantedBy=multi-user.target
' | sudo tee /etc/systemd/system/patroni.service > /dev/null ;\


sudo systemctl daemon-reload ;\
sudo systemctl enable patroni --now ;\
sudo systemctl start patroni.service
```

Статус службы
```
systemctl status patroni
```

Перечень нод
```
patronictl -c /etc/patroni/patroni.yml list
```

