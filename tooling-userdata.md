#!/bin/bash

exec > /var/log/tooling.log 2>&1

# Create directory and mount EFS
mkdir -p /var/www/
sudo mount -t efs -o tls,accesspoint=fsap-01060d13e30957a51 fs-0a71d71a5e6a321e6:/ /var/www/

# Install and start Apache
sudo yum install -y httpd
sudo systemctl start httpd
sudo systemctl enable httpd

# Install PHP and necessary extensions
sudo yum module reset php -y
sudo yum module enable php:remi-7.4 -y
sudo yum install -y php php-common php-mbstring php-opcache php-intl php-xml php-gd php-curl php-mysqlnd php-fpm php-json
sudo systemctl start php-fpm
sudo systemctl enable php-fpm

# Clone the tooling repository
git clone https://github.com/francdomain/tooling.git
mkdir -p /var/www/html/
cp -R tooling/html/* /var/www/html/

# Setup MySQL database
cd /tooling
mysql -h fnc-database.clcmaymew814.us-east-1.rds.amazonaws.com -u francis -p toolingdb < tooling-db.sql

cd /var/www/html/
touch healthstatus

sed -i "s/$db = mysqli_connect('mysql.tooling.svc.cluster.local', 'admin', 'admin', 'tooling');/$db = mysqli_connect('fnc-database.clcmaymew814.us-east-1.rds.amazonaws.com', 'francis', 'devopspbl', 'toolingdb');/g" functions.php

chcon -t httpd_sys_rw_content_t /var/www/html/ -R

# Disable Apache welcome page and restart Apache
sudo mv /etc/httpd/conf.d/welcome.conf /etc/httpd/conf.d/welcome.conf_backup
sudo systemctl restart httpd
