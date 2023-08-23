Установить Keepalived:

```
sudo apt update && sudo apt upgrade -y && sudo apt install keepalived -y ;\
sudo groupadd -r keepalived_script ;\
sudo useradd -r -s /sbin/nologin -g keepalived_script -M keepalived_script ;\
sudo sed -i 's/#net.ipv4.ip_forward=1/net.ipv4.ip_forward=1/' /etc/sysctl.conf ;\
sudo echo "net.ipv4.ip_nonlocal_bind = 1" >> /etc/sysctl.conf ;\
sudo sysctl -p
```

Создать конфигурационный файл `keepalived.conf` и перезапустить `keepalived.service`:

```
echo '
# Define the script used to check if haproxy is still working
vrrp_track_process haproxy {
    process haproxy
    quorum 1
    delay 2
}

# Configuration for Virtual Interface
vrrp_instance LB_VIP {
    interface eth0
    state MASTER        # set to BACKUP on the peer machine
    priority 101        # set to  99 on the peer machine
    virtual_router_id 51

    authentication {
        auth_type PASS
        auth_pass 1234  # Password for accessing vrrpd. Same on all devices
    }
    unicast_src_ip 192.168.1.50 # Private IP address of master
    unicast_peer {
        192.168.2.50             # Private IP address of the backup haproxy
   }

    # The virtual ip address shared between the two loadbalancers
    virtual_ipaddress {
        192.168.1.55 #dev eth0
    }

    # Use the Defined Script to Check whether to initiate a fail over
#    track_process {
    track_process {
        haproxy
    }
}
' | sudo tee /etc/keepalived/keepalived.conf > /dev/null ;\

sudo systemctl restart keepalived.service
```
