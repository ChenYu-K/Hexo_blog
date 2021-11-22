---
title: Centos7 update php5 to php7.2
date: 2021-11-15 13:48:29
tags: linux
---

In order to update to the recommended php version for wordpress,i update the original php 5.4 to php 7.2.

- check the list of installable php versions in yum.
```bash
# yum provides php
```

- To start upgrading the PHP update source, and remove the old php.
```bash
# rpm -Uvh https://mirror.webtatic.com/yum/el7/epel-release.rpm 
# rpm -Uvh https://mirror.webtatic.com/yum/el7/webtatic-release.rpm
# yum remove php-common -y
# yum install -y php72w php72w-opcache php72w-xml php72w-mcrypt php72w-gd php72w-devel php72w-mysql php72w-intl php72w-mbstring php72w-pdo php72w-pgsql
```
<!--more-->

- check the php version.
```bash
# php -v
PHP 7.2.34 (cli) (built: Oct  1 2020 13:37:37) ( NTS )
Copyright (c) 1997-2018 The PHP Group
Zend Engine v3.2.0, Copyright (c) 1998-2018 Zend Technologies
    with Zend OPcache v7.2.34, Copyright (c) 1999-2018, by Zend Technologies
```
- install the php-fem
```bash
# yum install php72w-fpm
# systemctl start php-fpm.service
# systemctl enable php-fpm.service
```