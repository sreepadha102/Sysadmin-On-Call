### Install Zabbix Agent on EC2 (Ubuntu 22.04)

```
wget https://repo.zabbix.com/zabbix/6.4/ubuntu/pool/main/z/zabbix-release/zabbix-release_6.4-1+ubuntu22.04_all.deb
sudo dpkg -i zabbix-release_6.4-1+ubuntu22.04_all.deb
sudo apt update


Install agent
-------------
sudo apt install zabbix-agent -y
```

### Configure Zabbix Agent
```
sudo vi /etc/zabbix/zabbix_agentd.conf

Server=<IP of zabbix-agent installed>
ServerActive=<IP of zabbix-agent installed>
Hostname=<vm name>
```

### Start and Enable Agent
```
sudo systemctl restart zabbix-agent
sudo systemctl enable zabbix-agent

sudo systemctl status zabbix-agent
```

### Validate Connectivity
```
zabbix_get -s <EC2_PRIVATE_IP> -k agent.ping

Expected output: 1
```

### logs can be checked at
sudo tail -f /var/log/zabbix/zabbix_agentd.log
