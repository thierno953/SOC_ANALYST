```sh
agent01@agent01:~$ sudo apt update
agent01@agent01:~$ sudo apt install net-tools dnsutils mc htop

agent01@agent01:~$ sudo apt install -y software-properties-common
agent01@agent01:~$ sudo add-apt-repository ppa:ondrej/php -y
agent01@agent01:~$ sudo apt update
```

```sh
agent01@agent01:~$ sudo apt install -y \
    apache2 \
    libapache2-mod-fcgid \
    php8.2-fpm \
    php8.2-gd \
    php8.2-mysql \
    php8.2-curl \
    php8.2-mbstring \
    php8.2-intl \
    php8.2-gmp \
    php8.2-bcmath \
    php8.2-xml \
    php8.2-imagick \
    php8.2-zip \
    mariadb-server
```

```sh
agent01@agent01:~$ sudo a2enmod proxy_fcgi setenvif
agent01@agent01:~$ sudo a2enconf php8.2-fpm
agent01@agent01:~$ sudo systemctl restart apache2 php8.2-fpm
```

```sh
agent01@agent01:~$ sudo mysql_secure_installation

#Switch to unix_socket authentication [Y/n] n
#Change the root password? [Y/n] y
#Remove anonymous users? [Y/n] y
#Disallow root login remotely? [Y/n] y
#Remove test database and access to it? [Y/n] y
#Reload privilege tables now? [Y/n] y
```

```sh
agent01@agent01:~$ sudo mysql
```

```sh
MariaDB [(none)]> CREATE USER 'nextclouduser'@'localhost' IDENTIFIED BY 'poukal12@';
MariaDB [(none)]> CREATE DATABASE IF NOT EXISTS nextcloud CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;
MariaDB [(none)]> GRANT ALL PRIVILEGES ON nextcloud.* TO 'nextclouduser'@'localhost';
MariaDB [(none)]> FLUSH PRIVILEGES;
MariaDB [(none)]> quit;
```

```sh
agent01@agent01:~$ sudoedit /var/www/html/info.php
```

```sh
<?php phpinfo() ?>
```

```sh
agent01@agent01:~$ sudo systemctl restart apache2
```

https://nextcloud.com/install/#community-projects

```sh
agent01@agent01:~$ wget https://download.nextcloud.com/server/releases/latest.zip
agent01@agent01:~$ unzip latest.zip
agent01@agent01:~$ ls
agent01@agent01:~$ sudo cp -r --preserve=mode nextcloud/* nextcloud/.htaccess nextcloud/.user.ini /var/www/html/
agent01@agent01:~$ ls -al /var/www/html/
agent01@agent01:~$ sudo chown -R www-data:www-data /var/www/html/
agent01@agent01:~$ sudo mkdir /nextcloud_data
agent01@agent01:~$ sudo chown -R www-data:www-data /nextcloud_data
agent01@agent01:~$ sudo chmod -R 750 /nextcloud_data
```

```sh
http://192.168.129.73/index.php
```

```sh
agent01@agent01:~$ sudo nano /var/www/html/nextcloud/config/config.php
```

```sh
'dbuser' => 'nextclouduser',
'dbpassword' => 'poukal12@',
```
