Установить HAProxy:

```
sudo apt install --no-install-recommends software-properties-common ;\
sudo add-apt-repository ppa:vbernat/haproxy-2.8 -y ;\
sudo apt install haproxy=2.8.\* -y
```

Создать конфигурационный файл `haproxy.cfg` и перезапустить демон:

```
echo '
global
        log /dev/log    local0
        log /dev/log    local1 notice
        chroot /var/lib/haproxy
        stats socket /run/haproxy/admin.sock mode 660 level admin
        stats timeout 30s
        user haproxy
        group haproxy
        daemon



defaults
    log global
    mode http
	mode tcp
    option  httplog
    option  dontlognull
    timeout connect 5000
    timeout client  50000
    timeout server  50000
    errorfile 400 /etc/haproxy/errors/400.http
    errorfile 403 /etc/haproxy/errors/403.http
    errorfile 408 /etc/haproxy/errors/408.http
    errorfile 500 /etc/haproxy/errors/500.http
    errorfile 502 /etc/haproxy/errors/502.http
    errorfile 503 /etc/haproxy/errors/503.http
    errorfile 504 /etc/haproxy/errors/504.http

listen stats
    bind :10001
    mode http
    stats enable
    stats uri /haproxy_stats
    stats auth admin:admin

listen postgres
    bind *:5000
    mode tcp
    option httpchk
    http-check expect status 200
    default-server inter 3s fall 3 rise 2 on-marked-down shutdown-sessions
    server patroni-node-1 192.168.1.100:6432 maxconn 100 check port 8008
    server patroni-node-2 192.168.2.100:6432 maxconn 100 check port 8008
    server patroni-node-3 192.168.3.100:6432 maxconn 100 check port 8008
' | sudo tee /etc/haproxy/haproxy.cfg > /dev/null ;\

sudo systemctl restart haproxy.service
```
