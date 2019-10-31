# RaspbPiConfig
Personal config for raspberry pi 3B+

# SSH at start
Create a file named `ssh` in root of the SD
```
cd a sdcard
type NUL >> ssh
```

# Wifi at start
create `wpa_supplicant.conf` at root of SD:
```
country=AR
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1

network={
    ssid="NETWORK-NAME"
    psk="NETWORK-PASSWORD"
}
```

# First boot
```
sudo raspi-config
```
change hsotname and expand filesystem

# Create user
```
sudo adduser USERHERE
for GROUP in adm dialout cdrom sudo audio video plugdev games users netdev input spi i2c gpio; do sudo adduser USERHERE $GROUP; done
```

# Autologin to CLI

Edit your /etc/systemd/logind.conf , change #NAutoVTs=6 to NAutoVTs=1

Create a /etc/systemd/system/getty@tty1.service.d/override.conf through ;

systemctl edit getty@tty1

Past the following lines
```
[Service]
ExecStart=
ExecStart=-/sbin/agetty --autologin USERHERE --noclear %I 38400 linux
```
enable the getty@tty1.service then reboot

systemctl enable getty@tty1.service

reboot

# Install ohmyZSH
`sudo apt install zsh`


`sh -c "$(wget https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"`



# Install Apache2 HTTP Server

```
sudo apt update
sudo apt install apache2
sudo systemctl stop apache2.service
sudo systemctl start apache2.service
sudo systemctl enable apache2.service

# Install MySQL and MariaDB Server
sudo apt-get install mariadb-server mariadb-client
sudo systemctl stop mariadb.service
sudo systemctl start mariadb.service
sudo systemctl enable mariadb.service
sudo mysql_secure_installation
```
```
    Enter current password for root (enter for none): Just press the Enter
    Set root password? [Y/n]: Y
    New password: Enter password
    Re-enter new password: Repeat password
    Remove anonymous users? [Y/n]: Y
    Disallow root login remotely? [Y/n]: Y
    Remove test database and access to it? [Y/n]:  Y
    Reload privilege tables now? [Y/n]:  Y
```

```
sudo systemctl stop mariadb.service
sudo systemctl start mariadb.service
sudo systemctl enable mariadb.service
```
```
sudo apt-get install software-properties-common
sudo add-apt-repository ppa:ondrej/php
sudo apt update
sudo apt install php7.2 libapache2-mod-php7.2 php7.2-common php7.2-gmp php7.2-curl php7.2-intl php7.2-mbstring php7.2-xmlrpc php7.2-mysql php7.2-gd php7.2-xml php7.2-cli php7.2-zip
```

# Config PHP
```
sudo nano /etc/php/7.2/apache2/php.ini
```
with:
```
file_uploads = On
allow_url_fopen = On
short_open_tag = On
memory_limit = 256M
upload_max_filesize = 100M
max_execution_time = 360
date.timezone = America/Argentina/Buenos_Aires
```

and restart PHP

```
sudo systemctl restart apache2.service
```
create a phpinfo.php
```
sudo nano /var/www/html/phpinfo.php
```
with
```
<?php phpinfo( ); ?>
```
You should see the info page in http://localhost/phpinfo.php

#  Create NextCloud Database

```
sudo mysql -u root -p
CREATE DATABASE nextcloud;
CREATE USER 'nextclouduser'@'localhost' IDENTIFIED BY 'new_password_here';
GRANT ALL ON nextcloud.* TO 'nextclouduser'@'localhost' IDENTIFIED BY 'user_password_here' WITH GRANT OPTION;
FLUSH PRIVILEGES;
EXIT;
```

# Download NextCloud Latest Release
```
sudo apt install curl git
curl -sS https://getcomposer.org/installer | sudo php -- --install-dir=/usr/local/bin --filename=composer
cd /var/www/html
sudo git clone --branch stable16 https://github.com/nextcloud/server.git nextcloud
cd /var/www/html/nextcloud
sudo composer install
sudo git submodule update --init
sudo chown -R www-data:www-data /var/www/html/nextcloud/
sudo chmod -R 755 /var/www/html/nextcloud/
```
# Configure apache2
```
sudo nano /etc/apache2/sites-available/nextcloud.conf
```
with
```
<VirtualHost *:80>
     ServerAdmin admin@example.com
     DocumentRoot /var/www/html/nextcloud/
     ServerName agustinso.serveo.net
     ServerAlias www.agustinso.serveo.net
  
     Alias /nextcloud "/var/www/html/nextcloud/"

     <Directory /var/www/html/nextcloud/>
        Options +FollowSymlinks
        AllowOverride All
        Require all granted
          <IfModule mod_dav.c>
            Dav off
          </IfModule>
        SetEnv HOME /var/www/html/nextcloud
        SetEnv HTTP_HOME /var/www/html/nextcloud
     </Directory>

     ErrorLog ${APACHE_LOG_DIR}/error.log
     CustomLog ${APACHE_LOG_DIR}/access.log combined

</VirtualHost>
```
# Enable the NextCloud and Rewrite Module
```
sudo a2ensite nextcloud.conf
sudo a2enmod rewrite
sudo a2enmod headers
sudo a2enmod env
sudo a2enmod dir
sudo a2enmod mime

sudo systemctl restart apache2.service
```
# Serveo Service and adding the trusted domain
```
sudo nano /var/www/html/nextcloud/config/config.php
```

```
sudo nano /etc/systemd/system/serveo.service
```
```
[Unit]
Description=Serveo connection to agustinso.serveo.net at port 80
After=network.target
StartLimitIntervalSec=0                                                                                                 [Service]                                                                                                               Type=simple
Restart=always
RestartSec=1
User=agustin
ExecStart=/usr/bin/env autossh -M 0 -o ServerAliveInterval=60 -R agustinso:80:localhost:80 serveo.net

[Install]
WantedBy=multi-user.target
```
```
sudo systemctl start serveo.service
sudo systemctl enable serveo.service
```
## Upgrading NextCloud
```
sudo mv /var/www/html/nextcloud /var/www/html/nextcloud_bak
cd /var/www/html
sudo git clone --branch stable14 https://github.com/nextcloud/server.git nextcloud
cd /var/www/html/nextcloud
sudo composer install
sudo git submodule update --init
sudo cp -rf /var/www/html/nextcloud_bak/data /var/www/html/nextcloud
sudo cp /var/www/html/nextcloud_bak/config/config.php /var/www/html/nextcloud/config/
sudo -u www-data php /var/www/html/nextcloud/occ maintenance:mode --on
sudo composer update /var/www/html/nextcloud --with-dependencies
sudo -u www-data php /var/www/html/nextcloud/occ upgrade
sudo -u www-data php /var/www/html/nextcloud/occ maintenance:mode --off
```
