```sh
root@control:~# apt update
root@control:~# apt -y dist-upgrade
root@control:~# apt -y install openssh-server
root@control:~# nano /etc/ssh/sshd_config
```

```sh
PermitRootLogin yes
```

```sh
root@control:~# systemctl restart sshd
root@control:~# apt install apache2 -y
root@control:~# sudo apt install software-properties-common -y
root@control:~# sudo add-apt-repository ppa:ondrej/php -y
root@control:~# sudo apt update
```

```sh
root@control:~# sudo apt install -y \
    php8.2-cli \
    php8.2-fpm \
    php8.2-common \
    php8.2-mysql \
    php8.2-curl \
    php8.2-gd \
    php8.2-intl \
    php8.2-mbstring \
    php8.2-xml \
    php8.2-zip \
    php8.2-bcmath \
    php8.2-gmp \
    php8.2-opcache \
    php8.2-apcu
```

```sh
root@control:~# php8.2 -v
root@control:~# php8.2 -m
```

```sh
root@control:~# sudo apt update && sudo apt install -y \
    libapache2-mod-php8.2 \
    hunspell \
    certbot \
    imagemagick \
    unzip \
    php8.2-opcache
```

```sh
root@control:~# php -v
root@control:~# ls /etc/php/
```

```sh
root@control:~# nano /var/www/html/main.php
```

```sh
<?php phpinfo(); ?>
```

```sh
root@control:~# systemctl restart apache2
root@control:~# systemctl status apache2
```

`http://<IP_SERVER>/main.php`

```sh
root@control:~# apt -y install mariadb-server
root@control:~# systemctl restart mariadb
```

```sh
root@control:~# mysql_secure_installation
```

```sh
Switch to unix_socket authentication [Y/n] n
Change the root password? [Y/n] n
Remove anonymous users? [Y/n] y
Disallow root login remotely? [Y/n] y
Remove test database and access to it? [Y/n] y
Reload privilege tables now? [Y/n] y
```

```sh
root@control:~# systemctl restart mariadb
```

```sh
root@control:~# mysql -u root
```

```sh
MariaDB [(none)]> show databases;
MariaDB [(none)]> create database glpi;
MariaDB [(none)]> create user 'admin'@localhost identified by 'passer';
MariaDB [(none)]> grant all privileges on glpi.* to admin@localhost;
MariaDB [(none)]> flush privileges;
MariaDB [(none)]> exit
```

```sh
root@control:~# systemctl restart mariadb
```

https://glpi-project.org/downloads/

```sh
root@control:~# wget https://github.com/glpi-project/glpi/releases/download/10.0.18/glpi-10.0.18.tgz

root@control:~# tar -xvf glpi-10.0.18.tgz -C /var/www/html/
root@control:~# ls /var/www/html/
glpi  index.html  main.php
```

```sh
root@control:~# nano /etc/apache2/sites-available/000-default.conf
```

```sh
<VirtualHost *:80>
     DocumentRoot /var/www/html/glpi
    ServerName yourdomain.com

    <Directory /var/www/html/glpi>
        Options FollowSymlinks
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/glpi_error.log
    CustomLog ${APACHE_LOG_DIR}/glpi_access.log combined
</VirtualHost>
```

```sh
root@control:~# chown -R www-data:www-data /var/www/html/glpi
root@control:~# ls -l /var/www/html
```

```sh
root@control:~# sudo apt install selinux-basics selinux-policy-default auditd
root@control:~# sudo selinux-activate
```

```sh
root@control:~# nano /etc/selinux/config
```

```sh
SELINUX=Enforcing
```

```sh
root@control:~# sudo apt install php8.2-ldap php8.2-imap php8.2-xmlrpc
```

```sh
root@control:~# systemctl restart apache2
```

```
Serveur SQL (MariaDB ou MySQL): localhost
Utilisateur SQL: admin
Mot de passe SQL: passer
```

```sh
root@control:~# rm -fr /var/www/html/glpi/install/install.php
```

```sh
root@control:~# nano /var/www/html/glpi/src/System/Requirement/SafeDocumentRoot.php
```

```sh
 else {
        $this->validated = false;
        return;
}
```

```sh
root@control:~# nano /etc/php/8.2/apache2/php.ini
```

```sh
session.cookie_httponly = on
```

```sh
root@control:~# systemctl restart apache2
```

`http://<IP_SERVER>/install/install.php`
