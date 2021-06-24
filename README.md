# apache
Настройка аутентификации для веб-сервера Apache 

## 1. Установка пакетов и предварительная настройка системы

Установка пакетов apache2, libapache2-mod-authnz-pam:

```
$ sudo apt install apache2 
$ sudo apt install libapache2-mod-authnz-pam 
```

## 2. Конфигурация виртуальных хостов

В реализации будет два сайта: **group1.com** с таблицей по протоколам IPSec и **group2.com** с таблицей по протоколам TLS.

```
mkdir -p /var/www/group1.com/public_html
touch /var/www/group1.com/public_html/index.php
mkdir -p /var/www/group2.com/public_html
touch /var/www/group2.com/public_html/index.php
```

Установка прав доступа к каталогам:

```
chmod 755 /var/www
chmod -R 755 /var/www/group1.com/public_html/
chmod -R 755 /var/www/group1.com/public_html/
```

Выдача прав доступа к таблице сведений о мандатных метках пользователей:

```
$ sudo usermod -a -G shadow www-data
$ sudo setfacl -d -m u:www-data:r /etc/parsec/macdb
$ sudo setfacl -R -m u:www-data:r /etc/parsec/macdb
$ sudo setfacl -m u:www-data:rx /etc/parsec/macdb
```

Создание пользователей, для которых будем настраивать доступ к ресурсам:

```
$ sudo adduser user_1
$ sudo adduser user_2
$ usermac -c 0:0 user_1
$ usermac -c 0:0 user_2
```

Создание БД с таблицами по протоколам IPSec и TLS:

```
$ su postgres -c "psql" 
```

```
=# CREATE TABLE IPSec
```

### group1.com

*/var/www/group1.com/public_html/index.php*:
```
```


### group2.com


*/var/www/group2.com/public_html/index.php*:
```
```



## 3. Конфигурация apache

### 3.1. PAM аутентификация


*/etc/apache2/sites-available/group1.com.conf*:

```
<VirtualHost *:80>
        ServerName group1.com
        ServerAlias www.group1.com
        ServerAdmin admin@group1.com
        DocumentRoot /var/www/group1.com/public_html
        <Directory /var/www/group1.com/public_html>
                AuthType Basic
                AuthBasicProvider PAM
                AuthPAMService apache2
                Options Indexes FollowSymLinks MultiViews
                AuthName "group_1 PAM auth"
                AllowOverride All
                Require user user_1
        </Directory>
        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

### 3.2. Digest аутентификация


*/etc/apache2/sites-available/group2.com.conf*:

```
<VirtualHost *:80>
        ServerName group2.com
        ServerAlias www.group2.com
        ServerAdmin admin@group2.com
        DocumentRoot /var/www/group2.com/public_html
        <Directory /var/www/group2.com/public_html>
                AuthType Digest
                AuthBasicProvider PAM
                AuthPAMService apache2
                Options Indexes FollowSymLinks MultiViews
                AuthName "realm_2"
                AllowOverride All
                AuthDigestProvider file
                AuthUserFile /var/www/group2.com/.htpasswd
                Require user user_2
        </Directory>
        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```


