## Zabbix agent installation on a ubuntu machine

### Prepare the system
sudo apt update
sudo apt upgrade -y
sudo apt install -y wget gnupg2 lsb-release ca-certificates

### Add Zabbix Repository
wget https://repo.zabbix.com/zabbix/6.4/ubuntu/pool/main/z/zabbix-release/zabbix-release_6.4-1+ubuntu22.04_all.deb
sudo dpkg -i zabbix-release_6.4-1+ubuntu22.04_all.deb
sudo apt update

### Install Zabbix Components
sudo apt install -y \
zabbix-server-mysql \
zabbix-frontend-php \
zabbix-apache-conf \
zabbix-sql-scripts \
zabbix-agent

### Install and Secure MariaDB
sudo apt install -y mariadb-server
sudo systemctl enable --now mariadb
sudo mysql_secure_installation

### Create Zabbix Database
sudo mysql -u root -p

CREATE DATABASE zabbix CHARACTER SET utf8mb4 COLLATE utf8mb4_bin;
CREATE USER 'zabbix'@'localhost' IDENTIFIED BY 'StrongPasswordHere';
GRANT ALL PRIVILEGES ON zabbix.* TO 'zabbix'@'localhost';
FLUSH PRIVILEGES;
EXIT;

### Import Zabbix Schema
sudo nano /etc/zabbix/zabbix_server.conf

DBName=zabbix
DBUser=zabbix
DBPassword=StrongPasswordHere

### Configure PHP Timezone
sudo vi /etc/zabbix/apache.conf
php_value date.timezone America/Toronto

sudo systemctl restart apache2

### Start and Enable Services
sudo systemctl enable --now zabbix-server zabbix-agent apache2

systemctl status zabbix-server

### Complete Web Installation
http://<server-ip>/zabbix

User: zabbix
Pass: above password
