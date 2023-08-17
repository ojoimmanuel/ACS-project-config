#!/bin/bash
sudo su -
mkdir /var/www/

#obtain mount point from tooling access point in EFS when you click on attach 
sudo mount -t efs -o tls,accesspoint=fsap-03ad4a2f38824ad76 fs-09c2b46e4803c9924:/ /var/www/
yum install -y httpd 
systemctl start httpd
systemctl enable httpd
yum module reset php -y
yum module enable php:remi-7.4 -y
yum install -y php php-common php-mbstring php-opcache php-intl php-xml php-gd php-curl php-mysqlnd php-fpm php-json
systemctl start php-fpm
systemctl enable php-fpm
git clone https://github.com/Livingstone95/tooling-1.git
mkdir /var/www/html
cp -R tooling-1/html/*  /var/www/html/
cd tooling-1/

#edit rds endpoint
mysql -h proj15-db.cife53kj4bdy.us-east-1.rds.amazonaws.com -u admin -p toolingdb < tooling-db.sql
cd /var/www/html/
touch healthstatus

#edit rds endpoint and login details 
sed -i "s/$db = mysqli_connect('mysql.tooling.svc.cluster.local', 'admin', 'admin', 'tooling');/$db = mysqli_connect('proj15-db.cife53kj4bdy.us-east-1.rds.amazonaws.com', 'admin', '12345678', 'toolingdb');/g" functions.php
chcon -t httpd_sys_rw_content_t /var/www/html/ -R
systemctl restart httpd
