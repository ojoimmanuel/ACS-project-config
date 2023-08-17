#!/bin/bash
sudo su -
mkdir /var/www/

#obtain mount point from wp access point from EFS by clicking on attach
sudo mount -t efs -o tls,accesspoint=fsap-0aeb7d37af73fce1c fs-09c2b46e4803c9924:/ /var/www/
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
cp -R wordpress/* /var/www/html/
cd /var/www/html/
touch healthstatus

#edit for appropraite rds endpoint, and login details
sed -i "s/localhost/proj15-db.cife53kj4bdy.us-east-1.rds.amazonaws.com/g" wp-config.php 
sed -i "s/username_here/admin/g" wp-config.php 
sed -i "s/password_here/12345678/g" wp-config.php 
sed -i "s/database_name_here/wordpressdb/g" wp-config.php 
chcon -t httpd_sys_rw_content_t /var/www/html/ -R
systemctl restart httpd
