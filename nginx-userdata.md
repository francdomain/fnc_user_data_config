#!/bin/bash

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

