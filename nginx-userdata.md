#!/bin/bash

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

yum install -y nginx
systemctl start nginx
systemctl enable nginx
git clone https://github.com/francdomain/fnc_user_data_config.git
mv /fnc_user_data_config/reverse.conf /etc/nginx/
mv /etc/nginx/nginx.conf /etc/nginx/nginx.conf-distro
cd /etc/nginx/
touch nginx.conf
sed -n 'w nginx.conf' reverse.conf
systemctl restart nginx
rm -rf reverse.conf
rm -rf /fnc_user_data_config

