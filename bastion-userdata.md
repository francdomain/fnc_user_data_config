#!/bin/bash

exec > /var/log/bastion.log 2>&1

yum install -y mysql
yum install -y git tmux
yum install -y ansible
