Установить Grafana и и плагин взаимодействия с Zabbix:

```
sudo apt update && sudo apt upgrade -y && sudo apt-get install adduser libfontconfig1 gnupg2 -y ;\
wget -P /tmp https://379116.selcdn.ru/S3/grafana-enterprise_10.0.3_amd64.deb ;\
sudo dpkg -i /tmp/grafana-enterprise_10.0.3_amd64.deb ;\
systemctl enable grafana-server ;\
systemctl restart grafana-server ;\
grafana-cli plugins install alexanderzobnin-zabbix-app ;\
systemctl restart grafana-server
```

