#!/bin/bash

exec > /var/log/wordpress.log 2>&1

sudo yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
sudo yum install -y dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm

yum install wget -y
yum install vim -y
yum install python3 -y
yum install telnet -y
yum install htop -y
yum install git -y
yum install mysql -y
yum install net-tools -y
yum install chrony -y

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
sudo mount -t efs -o tls,accesspoint=fsap-0ff4678d3b2bd11f5 fs-0da34877912ec9501:/ /var/www/
yum install -y httpd
systemctl start httpd
systemctl enable httpd
yum module reset php -y
yum module enable php:remi-7.4 -y
yum install -y php php-common php-mbstring php-opcache php-intl php-xml php-gd php-curl php-mysqlnd php-fpm php-json
systemctl start php-fpm
systemctl enable php-fpm
wget http://wordpress.org/latest.tar.gz
tar xzvf latest.tar.gz
rm -rf latest.tar.gz
cp wordpress/wp-config-sample.php wordpress/wp-config.php
mkdir /var/www/html/
cp -R /wordpress/* /var/www/html/
cd /var/www/html/
touch healthstatus
# sed -i "s/localhost/fnc-database.clcmaymew814.us-east-1.rds.amazonaws.com/g" wp-config.php
sed -i "s/localhost/${rds_endpoint}/g" wp-config.php
sed -i "s/username_here/francis/g" wp-config.php
sed -i "s/password_here/devopspbl/g" wp-config.php
sed -i "s/database_name_here/wordpressdb/g" wp-config.php
chcon -t httpd_sys_rw_content_t /var/www/html/ -R
systemctl restart httpd

# Create databases
mysql -h ${rds_endpoint} -P 3306 -u francis -p devopspbl -e "CREATE DATABASE IF NOT EXISTS wordpressdb;"
