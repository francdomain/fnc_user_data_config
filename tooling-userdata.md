#!/bin/bash

sudo yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm
sudo yum install -y dnf-utils http://rpms.remirepo.net/enterprise/remi-release-9.rpm

sudo yum install wget vim python3 telnet htop git mysql net-tools chrony -y

sudo systemctl start chronyd
sudo systemctl enable chronyd

sudo setsebool -P httpd_can_network_connect=1
sudo setsebool -P httpd_can_network_connect_db=1
sudo setsebool -P httpd_execmem=1
sudo setsebool -P httpd_use_nfs 1

git clone https://github.com/aws/efs-utils
cd efs-utils

sudo yum install -y make
sudo yum install -y rpm-build
sudo yum install openssl-devel -y
sudo yum install cargo -y
sudo make rpm
sudo yum install -y  ./build/amazon-efs-utils*rpm

mkdir /var/www/
sudo mount -t efs -o tls,accesspoint=fsap-072dcbbf66d01c7f4 fs-00f95e7d5fce567ab:/ /var/www/
yum install -y httpd
systemctl start httpd
systemctl enable httpd
yum module reset php -y
yum module enable php:remi-7.4 -y
yum install -y php php-common php-mbstring php-opcache php-intl php-xml php-gd php-curl php-mysqlnd php-fpm php-json
systemctl start php-fpm
systemctl enable php-fpm
git clone https://github.com/francdomain/tooling.git
mkdir /var/www/html
cp -R /tooling/html/*  /var/www/html/
cd /tooling
# mysql -h fnc-database.clcmaymew814.us-east-1.rds.amazonaws.com -u francis -p toolingdb < tooling-db.sql
mysql -h ${rds_endpoint} -u francis -p toolingdb < tooling-db.sql
cd /var/www/html/
touch healthstatus
sed -i "s/$db = mysqli_connect('mysql.tooling.svc.cluster.local', 'admin', 'admin', 'tooling');/$db = mysqli_connect('${rds_endpoint}', 'francis', 'devopspbl', 'toolingdb');/g" functions.php
chcon -t httpd_sys_rw_content_t /var/www/html/ -R
mv /etc/httpd/conf.d/welcome.conf /etc/httpd/conf.d/welcome.conf_backup
systemctl restart httpd

# Create databases
mysql -h ${rds_endpoint} -P 3306 -u francis -p devopspbl -e "CREATE DATABASE IF NOT EXISTS toolingdb;"
