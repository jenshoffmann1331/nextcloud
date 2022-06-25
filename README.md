# Nextcloud

## Manual Setup

Setup dependencies:

```
sudo apt-get update
sudo apt-get upgrade
sudo apt-get install apache2 libapache2-mod-php mariadb-server php-xml php-cli php-cgi php-mysql php-mbstring php-gd php-curl php-zip wget unzip
```

Edit /etc/php/7.4/apache2/php.ini:

```
memory_limt = 512M
upload_max_filesize = 2048M
post_max_size = 2048M
max_execution_time = 600
date.timezone = Europe/Berlin
```

Start services:

```
sudo systemctl start apache2
sudo systemctl start mariadb
sudo systemctl enable apache2
sudo systemctl enable mariadb
```

Create the database:

```
sudo mysql -u root -p
> CREATE DATABASE nextcloud;
> CREATE USER 'nextcloud'@'localhost' IDENTIFIED BY 'pass123';
> GRANT ALL ON nextcloud.* TO 'nextcloud'@'localhost';
> FLUSH PRIVILEGES;
> EXIT;
```

Download and install Nextcloud:

```
wget https://download.nextcloud.com/server/releases/latest.zip
unzip latest.zip
sudo mv nextcloud /var/www/html
sudo chown -R www-data:www-data /var/www/html/nextcloud
```

Create Apache site in /etc/apache2/sites-available/nextcloud.conf

```
<VirtualHost *:80>
  ServerAdmin jens@masterycloud.com
  DocumentRoot /var/www/html/nextcloud/
  ServerName  nextcloud.masterycloud.com
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

Enable new site:

```
sudo a2ensite nextcloud.conf
sudo systemctl restart apache2
```

Create data directory:

```
sudo mkdir /mnt/nextcloud
```

Install required php modules:

```
sudo apt-get install php-intl php-imagick php-gmp php-bcmath imagemagick
sudo service apache2 restart
```

Install memory cache:

```
sudo apt-get install redis-server php-redis
```

In /etc/redis/redis.conf:

```
port 0
unixsocket /var/run/redis/redis-server.sock
unixsocketperm 770
```

Add www-data to redis group

```
sudo usermod -a -G redis www-data
sudo service apache2 restart
sudo service redis-server start
sudo systemctl enable redis-server
```

Edit /var/www/html/nextcloud/config/config.php

```
...
  'memcache.local' => '\\OC\\Memcache\\Redis',
  'memcache.locking' => '\\OC\\Memcache\\Redis',
  'filelocking.enabled' => true,
  'redis' => array (
          'host' => '/var/run/redis/redis-server.sock',
          'port' => 0,
          'timeout' => 0.0,
  ),
...
```

```
sudo servic redis-server restart
sudo service apache2 restart
```