---
title: Zabbix Migration
author: Karl Webster
date: 2021-01-07
categories: [DevOps, Centos]
tags: [DevOps, Centos]
---


## My Spec
```
Old Server: Centos7
New Server: Centos7
DB:         RDS(SQL) on AWS
```
## Old Zabbix DB
So all my zabbix data is stored on a AWS RDS instance. Because of this I dont need to worry about creating a DB on the server as it *should* make migrating it a-lot easier.


## Zabbix

### Installation
https://www.zabbix.com/download?zabbix=5.0&os_distribution=centos&os_version=7&db=mysql&ws=apache


First make sure the system is up to date:
```bash
yum update -y
```
Install the repo:
```bash
rpm -Uvh https://repo.zabbix.com/zabbix/5.0/rhel/7/x86_64/zabbix-release-5.0-1.el7.noarch.rpm
yum clean all
```

Install the agent and server:
```bash
yum install zabbix-server-mysql zabbix-agent -y
```

Install the Zabbix Frontend
```bash
yum install centos-release-scl -y
```

Edit file `/etc/yum.repos.d/zabbix.repo` and enable zabbix-frontend repository.
```
[zabbix-frontend]
...
enabled=1
...
```

Install the front end packages:
```bash
yum install zabbix-web-mysql-scl zabbix-apache-conf-scl -y
```

Now you would create the mysql database if you need it, as mentioned I am already using an RDS instance so there is no need for me to do this.

### Config
Edit the `/etc/zabbix/zabbix_server.conf` and copy what the old server had for its config, the most important thing for me was to copy the Database settings.

Restart the services and enable them so they run on restart:
```bash
systemctl restart zabbix-server zabbix-agent httpd rh-php72-php-fpm
systemctl enable zabbix-server zabbix-agent httpd rh-php72-php-fpm
```

I always like to reboot to check it works as expected so I will do that:
```bash
reboot
```

Once its restarted go to the following URL to finish the setup:
```
http://<ServerIPAddress>/zabbix
```

You should see the welcome screen, Click next to check all the pre-requisites have been fufilled:


On the next screen you want to put the DB credentials in again.

I had to edit `/etc/selinux/config` and set SeLinux to `permissive` for it to connect to the database. (Dont forget you need to reboot for the SElinix change to take effect)

For the next screen the defaults are fine


Now you have sucessfully migrated Zabbix to a new server.