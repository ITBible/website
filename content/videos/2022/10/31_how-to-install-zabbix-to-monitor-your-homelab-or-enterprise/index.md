---
title: "How to install Zabbix to monitor your Homelab or Enterprise"
description: "A quick rundown of installing Zabbix to monitor your homelab or even enterprise environment."
summary: "A quick rundown of installing Zabbix to monitor your homelab or even enterprise environment."
category: Hypervisor
tags: [Tutorial, ProxmoxVE]
date: 2022-10-17
slug: how-to-install-zabbix-to-monitor-your-homelab-or-enterprise
authors:
    - "wizardtux"
---
{{< youtubeLite id="X3Tv4GMOF0U" label="How to install Zabbix to monitor your Homelab or Enterprise" >}}

# Install the repository
```bash
wget https://repo.zabbix.com/zabbix/6.2/ubuntu/pool/main/z/zabbix-release/zabbix-release_6.2-2%2Bubuntu22.04_all.deb
sudo dpkg -i zabbix-release_6.2-2+ubuntu22.04_all.deb
sudo apt update
```

# Install Zabbix server, front end and agent
I had to install mysql-server as zabbix-server-mysql didn't seem to install it at the time of this recording.

This is not a comprehensive install of MySQL and there is more to do to secure your MySQL installation.
```bash
sudo apt install mysql-server zabbix-server-mysql zabbix-frontend-php zabbix-nginx-conf zabbix-sql-scripts zabbix-agent
```
# Verify MySQL is running
```bash
ps ax | grep mysql
```
# Login to MySQL
```bash
sudo mysql -u root
```
# Create the Zabbix database
I would suggest changing the database and user name as well as the password but for this example we are going to use zabbix for the database and user and then password for the user's password.

SQL:
```sql
create database zabbix character set utf8mb4 collate utf8mb4_bin;
create user zabbix@localhost identified by 'password';
grant all privileges on zabbix.* to zabbix@localhost;
set global log_bin_trust_function_creators = 1;
quit;
```

# Import initial schema
This step will take a few minutes but will be what imports the Zabbix database structure into your database, this step does take a couple minutes once you hit enter.
```bash
zcat /usr/share/zabbix-sql-scripts/mysql/server.sql.gz | sudo mysql --database=zabbix --default-character-set=utf8mb4 -u zabbix -p
```
# Disable log_bin_trust_function_creators option after importing
Log back in to MySQL as root

```bash
sudo mysql -u root
Set the global variable back to 0
set global log_bin_trust_function_creators = 0;
quit;
```

Configure the database for Zabbix server
```bash
sudo nano /etc/zabbix/zabbix_server.conf
```
Press ctrl+w to search and search for DBPassword, you'll want to create this variable and set the password that you set when creating the database user.

Configure Nginx for the front end
In the video I only edited the listen line (because its going to be behind a reverse proxy) but if you are wanting to have it resolve a url you'll need to also un-comment the server_name line.

```bash
sudo nano /etc/zabbix/nginx.conf
```
Start zabbix server and agent
```bash
sudo systemctl restart zabbix-server zabbix-agent nginx php8.1-fpm
sudo systemctl enable zabbix-server zabbix-agent nginx php8.1-fpm
```
Navigate to your server in a web browser. In the video example I used 192.168.1.50 so my URL became http://192.168.1.50:8080 once the page loads you can login with the below information:
Username: Admin
Password: zabbix