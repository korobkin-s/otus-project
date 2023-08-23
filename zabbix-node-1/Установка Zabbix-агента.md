Установить Zabbix-агент (IP сервера Zabbix заменить):

```
sudo apt update ;\
sudo apt install zabbix-agent -y &&
sudo sed -i 's/^\(Server=127.0.0.1\)/# \1/g' /etc/zabbix/zabbix_agentd.conf ;\
sudo sed -i 's/^\(ServerActive=127.0.0.1\)/# \1/g' /etc/zabbix/zabbix_agentd.conf ;\
echo 'Server=192.168.1.20' | sudo tee -a /etc/zabbix/zabbix_agentd.conf ;\
echo 'ServerActive=192.168.1.20' | sudo tee -a /etc/zabbix/zabbix_agentd.conf ;\
echo 'Hostname=system.hostname' | sudo tee -a /etc/zabbix/zabbix_agentd.conf ;\
sudo systemctl restart zabbix-agent
```
