# Install and manage a complete PHP development environment (with https) on a Chromebook (or Debian) with just a few copy-paste

This guide explains how to install and manage a complete PHP development environment (PHP, Apache, MySQL, ) with HTTPS suppport for Virtual Hosts on a Chromebook (or Debian/Ubuntu) with just a few copy-paste.

## A. APPLICATIONS
### 1. INITIALISATION and TOOLS
⏱️ Copy the content of the cell below, paste it into the terminal (shift+ctrl+v), press the “enter” key and wait.
```shell
sudo apt update &&
sudo apt upgrade -y &&
sudo apt install curl -y &&
sudo apt install wget -y &&
sudo apt install nano -y &&
sudo apt install ufw -y &&
sudo apt install wget libnss3-tools -y &&
curl -JLO "https://dl.filippo.io/mkcert/latest?for=linux/amd64" &&
chmod +x mkcert-v*-linux-amd64 &&
sudo mv mkcert-v*-linux-amd64 /usr/local/bin/mkcert 
```

### 2. APACHE
⏱️ Copy the content of the cell below, paste it into the terminal (shift+ctrl+v), press the “enter” key and wait.
```shell
sudo apt install apache2 -y &&
sudo ufw allow OpenSSH &&
sudo ufw allow 'WWW Full' &&
sudo apt update &&
sudo apt upgrade -y &&
sudo a2dissite 000-default &&
sudo systemctl reload apache2 &&
sudo touch /etc/apache2/sites-available/localhost.conf &&
sudo chmod 755 /etc/apache2/sites-available/localhost.conf &&
sudo bash -c "cat <<EOT >> /etc/apache2/sites-available/localhost.conf
<VirtualHost *:80>
    ServerName localhost
    DocumentRoot /var/www/html
    <Directory /var/www/html>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
    ErrorLog localhost_error.log 
    CustomLog localhost_access.log combined 
</VirtualHost>
EOT" &&
sudo a2ensite localhost.conf &&
sudo systemctl reload apache2
```

### 3. PHP
⏱️ Copy the content of the cell below, paste it into the terminal (shift+ctrl+v), press the “enter” key and wait.
```shell
PHP_VERSION=8.4
sudo apt -y install lsb-release apt-transport-https ca-certificates &&
sudo apt-get -y install ca-certificates apt-transport-https software-properties-common &&
sudo add-apt-repository ppa:ondrej/php &&
sudo apt update &&
sudo apt install php$PHP_VERSION -y &&
sudo apt install php$PHP_VERSION-{bcmath,bz2,cgi,curl,fpm,xml,mysql,zip,imap,intl,imagick,ldap,gd,mbstring,mysql,pgsql,soap,xmlrpc} -y &&
sudo a2enmod proxy_fcgi setenvif &&
sudo a2enconf php$PHP_VERSION-fpm &&
sudo systemctl reload apache2
```

## B. VIRTUAL HOSTS

### 1. ADD A VIRTUAL HOST
Copy the content of the cell below, paste it into the terminal (shift+ctrl+v), change the name of the virtual host  and press the “enter” key.
```shell
VH_NAME="name_of_the_virtual_host_you_want_to_create"
```
Copy the content of the cell below, paste it into the terminal (shift+ctrl+v), press the “enter” key.
```shell
sudo touch /etc/apache2/sites-available/$VH_NAME.conf &&
sudo chmod 755 /etc/apache2/sites-available/$VH_NAME.conf &&
sudo bash -c "cat <<EOT >> /etc/apache2/sites-available/$VH_NAME.conf
<VirtualHost *:443>
    ServerName $VH_NAME
    DocumentRoot /var/www/html/$VH_NAME
    SSLEngine on
    SSLCertificateKeyFile /etc/ssl/private/$VH_NAME.key
    SSLCertificateFile /etc/ssl/certs/$VH_NAME.crt
    <Directory /var/www/html/$VH_NAME>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
    ErrorLog ${VH_NAME}_error.log 
    CustomLog ${VH_NAME}_access.log combined  
</VirtualHost>
<VirtualHost *:80>
    ServerName $VH_NAME
    DocumentRoot /var/www/html/$VH_NAME
    <Directory /var/www/html/$VH_NAME>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
    ErrorLog ${VH_NAME}_error.log 
    CustomLog ${VH_NAME}_access.log combined  
</VirtualHost>
EOT" &&
sudo a2ensite $VH_NAME.conf &&
sudo bash -c "cat <<EOT >> /etc/hosts
127.0.0.1 $VH_NAME
EOT" &&
sudo a2enmod ssl &&
mkcert -install &&
mkcert -key-file $VH_NAME.key -cert-file $VH_NAME.crt $VH_NAME &&
sudo mv $VH_NAME.key /etc/ssl/private/$VH_NAME.key &&
sudo mv $VH_NAME.crt /etc/ssl/certs/$VH_NAME.crt &&
sudo systemctl reload apache2
```

### 2. DELETE A VIRTUAL HOST
Copy the content of the cell below, paste it into the terminal (shift+ctrl+v), change the name of the virtual host  and press the “enter” key.
```shell
VH_NAME="name_of_the_virtual_host_you_want_to_delete"
```
Copy the content of the cell below, paste it into the terminal (shift+ctrl+v), press the “enter” key.
```shell
sudo a2dissite $VH_NAME &&
sudo rm /etc/apache2/sites-available/$VH_NAME.conf
sudo sed -i "/$VH_NAME/d" /etc/hosts &&
sudo systemctl reload apache2
```

