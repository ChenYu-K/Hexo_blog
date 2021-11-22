---
title: Centos 7.X use yum to install-php5.6.X
date: 2021-11-12 20:55:07
tags: linux
---

1. Check now php package  
```
yum list installed | grep php
```

2. if there have any php, deleate it first.  

```
yum remove php.x86_64 php-cli.x86_64 php-common.x86_64 php-gd.x86_64 php-ldap.x86_64 php-mbstring.x86_64 php-mcrypt.x86_64 php-mysql.x86_64 php-pdo.x86_64 
```
<!--more-->

3. Configure EPEL source  
```
yum install -y epel-release
wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
```
4. Configure remi source
```
rpm -ivh http://rpms.famillecollet.com/enterprise/remi-release-7.rpm
```

5. Install php5.6.x
```
yum install --enablerepo=remi --enablerepo=remi-php56 php php-opcache php-devel php-mbstring php-mcrypt php-mysqlnd php-phpunit-PHPUnit php-pecl-xdebug php-pecl-xhprof
```

6. install php-fpm
```
yum install --enablerepo=remi --enablerepo=remi-php56 php-fpm 
```

7. Configure boot service
```
systemctl restart php-fpm
systemctl enable php-frm
```

8. Check if the installation is successful
```
ps -ef | grep php
netstat -anp | grep 9000
```

9. restar service
```
sudo systemctl restart httpd
sudo systemctl restart php-fpm
service httpd restart
```