### order of creation

- create security group first
- create certificate
- create ALB
- create Launch template
- create Austocaling



# Bastion ami installation
-------------------------------------
yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm
yum install -y dnf-utils http://rpms.remirepo.net/enterprise/remi-release-9.rpm
yum install wget vim python3 telnet htop git mysql net-tools chrony -y
systemctl start chronyd
systemctl enable chronyd

# Nginx ami installation
-----------------------------------------
```
yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm

yum install -y dnf-utils http://rpms.remirepo.net/enterprise/remi-release-9.rpm

yum install wget vim python3 telnet htop git mysql net-tools chrony -y

systemctl start chronyd

systemctl enable chronyd
```
## configure selinux policies for the webservers and nginx servers
```
setsebool -P httpd_can_network_connect=1
setsebool -P httpd_can_network_connect_db=1
setsebool -P httpd_execmem=1
setsebool -P httpd_use_nfs 1
```
## this section will instll amazon efs utils for mounting the target on the Elastic file system
```
git clone https://github.com/aws/efs-utils

cd efs-utils

yum install -y make

yum install -y rpm-build

# openssl-devel is needed by amazon-efs-utils-2.0.4-1.el9.x86_64
sudo yum install openssl-devel -y

# Cargo command needs to be installed as it is necessary for building the Rust project included in the source.
sudo yum install cargo -y

make rpm

yum install -y  ./build/amazon-efs-utils*rpm
```
## seting up self-signed certificate for the nginx instance
```
sudo mkdir /etc/ssl/private

sudo chmod 700 /etc/ssl/private

openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/fnc.key -out /etc/ssl/certs/fnc.crt

sudo openssl dhparam -out /etc/ssl/certs/dhparam.pem 2048
```


# webserver ami installation
```
yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm

yum install -y dnf-utils http://rpms.remirepo.net/enterprise/remi-release-9.rpm

yum install wget vim python3 telnet htop git mysql net-tools chrony -y

systemctl start chronyd

systemctl enable chronyd
```
## configure selinux policies for the webservers and nginx servers
```
setsebool -P httpd_can_network_connect=1
setsebool -P httpd_can_network_connect_db=1
setsebool -P httpd_execmem=1
setsebool -P httpd_use_nfs 1
```
## this section will instll amazon efs utils for mounting the target on the Elastic file system
```
git clone https://github.com/aws/efs-utils

cd efs-utils

yum install -y make

yum install -y rpm-build

make rpm

yum install -y  ./build/amazon-efs-utils*rpm
```

## seting up self-signed certificate for the apache  webserver instance
```
yum install -y mod_ssl

openssl req -newkey rsa:2048 -nodes -keyout /etc/pki/tls/private/fnc.key -x509 -days 365 -out /etc/pki/tls/certs/fnc.crt

vi /etc/httpd/conf.d/ssl.conf
```




# Login into the RDS instnace  and create  database for wordpress and tooling wordpress and tooling database
mysql -h fnc-database.cdqpbjkethv0.us-east-1.rds.amazonaws.com -u adminuser -p

CREATE DATABASE toolingdb;
CREATE DATABASE wordpressdb;


# References
[IP ranges](https://ipinfo.io/ips)

[ssh-gent forwarding on mobaxterm](http://docs.gcc.rug.nl/hyperchicken/ssh-agent-forwarding-mobaxterm/)

[Nginx reverse proxy server](https://www.nginx.com/resources/glossary/reverse-proxy-server/)

[Underdstanding ec2 user data](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/user-data.html)

[Manually installing the Amazon EFS client](https://docs.aws.amazon.com/efs/latest/ug/installing-amazon-efs-utils.html#installing-other-distro)

[creating target groups for AWS Loadbalancer](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-target-groups.html)

[Self-Signed SSL Certificate for Apache](https://www.digitalocean.com/community/tutorials/how-to-create-a-self-signed-ssl-certificate-for-apache-on-centos-8)

[Create a Self-Signed SSL Certificate for Nginx](https://www.digitalocean.com/community/tutorials/how-to-create-a-self-signed-ssl-certificate-for-nginx-on-centos-7)
