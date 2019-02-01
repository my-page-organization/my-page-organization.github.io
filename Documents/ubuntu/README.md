# Introduction

This document will list steps on how to install **Faveo Helpdesk** on a new **Ubuntu 16.04 LTS**.

We will install following dependencies in order to make Faveo Helpdesk work:
- Apache
- PHP 7.1
- PHP Extensions: listed in [server requirement](https://github.com/ladybirdweb/faveo-helpdesk/wiki/Server-Requirements)
- MySQL
- Composer
- Cron Job

Read the detailed list of [server requirement](https://github.com/ladybirdweb/faveo-helpdesk/wiki/Server-Requirements)

We are using vi editor throughout to open and edit file, you can use nano editor also

## Configure IP Tables
Please note that you have to make changes in the iptables configurations. This allows to open ports that are necessary in Faveo installation.

This is an **optional step**, If you are able to access your server remotely on Public IP. This step will not be required.Mainly on local network server this step is required. If you are purchasing/renting server in a data centre this step might not be required.

```
iptables -A INPUT -p tcp --dport 22 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT

iptables -A OUTPUT -p tcp --sport 22 -m conntrack --ctstate ESTABLISHED -j ACCEPT

iptables -A INPUT -p tcp --dport 80 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT

iptables -A OUTPUT -p tcp --sport 80 -m conntrack --ctstate ESTABLISHED -j ACCEPT

iptables -A INPUT -p tcp --dport 443 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT

iptables -A OUTPUT -p tcp --sport 443 -m conntrack --ctstate ESTABLISHED -j ACCEPT
```

**PS:**
* You have to reset the firewall and iptables to your specifications
* This step might vary for different data centres or cloud service providers, Please check with your hosting company on opening port number and correct settings

## Create a user for Faveo and Add Repos
```
useradd -r www-data

usermod -G www-data www-data

apt-get install -y software-properties-common 
  
sudo add-apt-repository ppa:ondrej/php

apt-get update
```

## Install Apache and PHP with Extensions

In this step we install following
* PHP
* Required PHP Extension
* Git
* MariaDb
* Curl
* OpenSSL
* Apache

```
apt-get install -y curl git apache2

apt-get install sl mlocate dos2unix bash-completion openssl  php7.1-xml php7.1-xsl php7.1-mbstring php7.1-readline php7.1-zip php7.1-mysql php7.1-phpdbg php7.1-interbase php7.1-sybase php7.1 php7.1-sqlite3 php7.1-tidy php7.1-opcache php7.1-pspell php7.1-json php7.1-xmlrpc php7.1-curl php7.1-ldap php7.1-bz2 php7.1-cgi php7.1-imap php7.1-cli php7.1-dba php7.1-dev php7.1-intl php7.1-fpm php7.1-recode php7.1-odbc php7.1-gmp php7.1-common php7.1-pgsql php7.1-bcmath php7.1-snmp php7.1-soap php7.1-mcrypt php7.1-gd php7.1-enchant libapache2-mod-php7.1 libphp7.1-embed && updatedb

a2enmod rewrite

service apache2 restart
```

## Configure a Database for Faveo in MySQL
```
apt-get install -y mysql-server

service mysql start

mysql_secure_installation

mysql -u root -p

CREATE DATABASE faveo;

GRANT ALL PRIVILEGES ON faveo.* TO 'faveouser'@'localhost' IDENTIFIED BY 'faveouserpass';

FLUSH PRIVILEGES;

quit
```

## Setting Up ionCube
```
wget http://downloads3.ioncube.com/loader_downloads/ioncube_loaders_lin_x86-64.tar.gz

tar xvfz ioncube_loaders_lin_x86-64.tar.gz

php -i | grep extension_dir (To find PHP extension Directory)
```
Copy ioncube loader to Directory.
```
sudo cp ioncube/ioncube_loader_lin_7.1.so /usr/lib/php/20160303 

Add below line to before Windows Extensions in vi /etc/php/7.1/apache2/php.ini and /etc/php/7.1/cli/php.ini configuration.

zend_extension = "/usr/lib/php/20160303/ioncube_loader_lin_7.1.so"
```
Save and exit.
```
systemctl restart apache2.service
 ```
 
## Copy Faveo Help Desk from Github
Create a folder for Faveo and clone Faveo Help Desk Community latest release from Github to it
```
mkdir -p /var/www/faveo/faveo-helpdesk

git clone https://github.com/ladybirdweb/faveo-helpdesk.git /var/www/faveo/faveo-helpdesk
```
## Copy Faveo Help Desk via SSH
Incase you want to upload the Faveo files from your local system to your server and not use Github, then follow this step
Login to directory
```
Scp filename.zip username@ip:/location/to/
```
## Give correct file permission to Faveo files

```
chown -R www-data:www-data /var/www/

chown -R www-data:www-data /var/www/faveo/

chown -R www-data:www-data /var/www/faveo/faveo-helpdesk/

chmod -R 755 /var/www/

chmod -R 755 /var/www/faveo/

chmod -R 755 /var/www/faveo/faveo-helpdesk/

chmod -R 755 /var/www/faveo/faveo-helpdesk/storage/

chmod -R 755 /var/www/faveo/faveo-helpdesk/bootstrap/
```

## Install Composer
```
curl -sS https://getcomposer.org/installer | php

mv composer.phar /usr/bin/composer

chmod +x /usr/bin/composer
```
## Create a Virtual Host Configuration

```
vim /etc/apache2/sites-available/faveo.conf
```

Copy the contents below to above file

```
<VirtualHost *:80> 

ServerName localhost

ServerAdmin webmaster@localhost

DocumentRoot /var/www/faveo/faveo-helpdesk/public

<Directory /var/www/faveo/faveo-helpdesk>

AllowOverride All

</Directory>

ErrorLog ${APACHE_LOG_DIR}/error.log

CustomLog ${APACHE_LOG_DIR}/access.log combined

</VirtualHost>

```

## Activate the new apache configuration and restart apache and mysql services
```
a2ensite faveo.conf

a2dissite 000-default.conf

service apache2 restart

service mysql restart

```
 
## Setup Cron Job
We are using default localhost URL where Faveo is installed, you can change the URL based on your system setting and IP address
```
crontab -u www-data -e

* * * * * /usr/bin/php7.1 /var/www/faveo/faveo-helpdesk/artisan schedule:run >> /dev/null 2>&1
```
## Start Installation

Now you can install Faveo via [GUI](https://github.com/ladybirdweb/faveo-helpdesk/wiki/GUI-Install-Wizard) Wizard or [CLI](https://github.com/ladybirdweb/faveo-helpdesk/wiki/Install-Faveo-via-CLI).

You can access Faveo url in the browser

**PS:** 
* You have to reset the firewall and iptables to your specifications
* You need to follow steps yourself to harden the security of your server, server security is not covered in this article
* Redis is recommended for messaging que and improving system performance
* Always use SSL/HTTPS URL for Faveo

## Redis Installation
Redis is an open source (BSD licensed), in-memory data structure store, used as a database, cache and message broker.

This is an optional step and will improve the system performance and is highly recommended.

[Install and configure Redis, Supervisor and Worker for Faveo on Ubuntu 16.04](https://github.com/ladybirdweb/faveo-helpdesk/wiki/Install-and-configure-Redis,-Supervisor-and-Worker-for-Faveo-on-Ubuntu-16.04)
