Загрузить и распаковать бинарник Consul:

```
sudo apt update && sudo apt upgrade -y && sudo apt install unzip -y ;\
wget -P /tmp https://379116.selcdn.ru/links/consul_1.16.1_linux_amd64.zip ;\
unzip /tmp/consul_1.16.1_linux_amd64.zip -d /usr/bin/ ;\
sudo chmod +x /usr/bin/consul ;\
sudo useradd -r -c 'Consul DCS service' consul ;\
sudo mkdir -p /var/lib/consul /etc/consul.d ;\
sudo chown consul:consul /var/lib/consul /etc/consul.d ;\
sudo chmod 775 /var/lib/consul /etc/consul.d
```

Сгенерировать ключ (`encrypt`) командой `consul keygen` (далее использовать его на остальных нодах).
Создать конфигурационный файл `config.json` и демон `consul.service`:

```
echo '
{
    "bind_addr": "0.0.0.0",
    "bootstrap_expect": 3,
    "client_addr": "0.0.0.0",
    "data_dir": "/var/lib/consul",
    "enable_script_checks": true,
    "dns_config": {
        "enable_truncate": true,
        "only_passing": true
    },
    "enable_syslog": true,
    "encrypt": "l98qB67tEGNArGNjc0tl2eIhtS7Q+o04qX6k8QIDs3E=",
    "leave_on_terminate": true,
    "log_level": "INFO",
    "rejoin_after_leave": true,
    "retry_join": [
        "patroni-node-1",
        "patroni-node-2",
        "patroni-node-3"
    ],
    "server": true,
    "start_join": [
        "patroni-node-1",
        "patroni-node-2",
        "patroni-node-3"
    ],
   "ui_config": { "enabled": true }
}
' | sudo tee /etc/consul.d/config.json > /dev/null ;\

echo '
[Unit]
Description=Consul Service Discovery Agent
Documentation=https://www.consul.io/
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=consul
Group=consul
ExecStart=/usr/bin/consul agent \
    -node=consul01.yc.local \
    -config-dir=/etc/consul.d
ExecReload=/bin/kill -HUP $MAINPID
KillSignal=SIGINT
TimeoutStopSec=5
Restart=on-failure
SyslogIdentifier=consul

[Install]
WantedBy=multi-user.target
' | sudo tee /etc/systemd/system/consul.service > /dev/null ;\

sudo systemctl daemon-reload ;\
sudo systemctl enable consul.service ;\
sudo systemctl start consul.service
```

Состояние нод: 

```
systemctl status consul
```

Детальная информация:

```
consul members
consul members -detailed
```
