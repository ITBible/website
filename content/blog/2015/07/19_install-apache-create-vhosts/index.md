---
title: 'Archive: Install Apache and Create Virtual Hosts in Ubuntu and CentOS'
description: 'This guide is going to cover installing Apache2 and creating multiple virtual hosts, for different domains and / or subdomains. I will show examples in Ubuntu and CentOS. I started with a fresh install of both operating systems for this tutorial.'
summary: 'This guide is going to cover installing Apache2 and creating multiple virtual hosts, for different domains and / or subdomains. I will show examples in Ubuntu and CentOS. I started with a fresh install of both operating systems for this tutorial.'
category: Linux
tags: [Archive, Apache, Linux]
date: 2015-07-19
slug: archive-install-apache-and-create-virtual-hosts-in-ubuntu-and-centos
authors:
    - "wizardtux"
---

{{< alert icon="fire" cardColor="#e63946" iconColor="#1d3557" textColor="#f1faee" >}}
This is a re-post from a previous version of The Computer Crowd.
{{< /alert >}}

If you are reading this chances are you are looking on how to host multiple websites on a single server.

This guide is going to cover installing Apache2 and creating multiple virtual hosts, for different domains and / or subdomains. I will show examples in Ubuntu and CentOS. I started with a fresh install of both operating systems for this tutorial.

Note: I never suggest logging in as root. I would also run the service as a non-root user.

# Ubuntu
## Step 1: Install Apache2
```bash
sudo apt-get install apache2
```
## Step 2: Create Directory Structure
Personally I like the directory structure organized so I would do something like below.
```bash
sudo mkdir /var/www/domain1.com
sudo mkdir /var/www/domain2.com
```

## Step 3: Create/Edit Your Virtual Hosts
For this example you can just copy and paste the code below for each of your domains.
```bash
sudo nano /etc/apache2/sites-available/domain1.com
```
Be sure to change the information to your specific info.
```text
<VirtualHost *:80>
        ServerAdmin webadmin@domain1.com
        ServerName domain1.com
        ServerAlias www.domain1.com domain1.com

        DocumentRoot /var/www/domain1.com

        <Directory /var/www/domain1.com/>
                Options Indexes FollowSymLinks MultiViews
                AllowOverride all
                Order allow,deny
                allow from all
        </Directory>

        ErrorLog ${APACHE_LOG_DIR}/domain1.com-error.log
        CustomLog ${APACHE_LOG_DIR}/domain1.com-access.log combined
</VirtualHost>
```
Then repeat for additional domains.

## Step 4: Enabling Your Virtual Hosts
For each file you created above run the following command.
```bash
sudo a2ensite domain1.com
```
Once you have enabled all of your virtual hosts you then run.
```bash
sudo service apache2 reload
```
As long as your domains are pointed to your network and port 80 is open your domains should be directed to the correct virtual host.

# CentOS
## Step 1: Install httpd
```bash
sudo yum install httpd
```
## Step 2: Have Apache Start at Boot
```bash
sudo chkconfig -levels 235 httpd on
sudo service httpd start
```
or with CentOS 7
```bash
sudo systemctl enable httpd
sudo systemctl start httpd
```

## Step 3: Create Your Directory Structure
```bash
sudo mkdir /var/www/domain1.com
sudo mkdir /var/www/domain2.com
```
Repeat for any additional sites.

## Step 4: Configure Your Virtual Hosts
Open the httpd.conf
```bash
sudo vi /etc/httpd/conf/httpd.conf
```
Add the following lines to the bottom of httpd.conf be sure to hit “i” to insert text.
```text
<VirtualHost *:80>
    ServerAdmin webmaster@domain1.com
    ServerName www.domain1.com
    ServerAlias www.domain1.com domain1.com

    Document Root /www/domain1.com

    ErrorLog logs/domain1.com-error.log
    CustomLog logs/domain1.com-access.log combined
</VirtualHost>
```
Repeat for each additional host you want to add.

Save the file by hitting Esc and typing:
```bash
:wq
```
and hitting enter.

## Step 5: Restart the Apache HTTPD
```bash
sudo service httpd restart
```
or in CentOS 7
```bash
sudo systemctl restart httpd
```