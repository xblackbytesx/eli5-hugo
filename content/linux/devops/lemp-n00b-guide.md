+++
date = "2018-10-26T08:58:03+01:00"
title = "LEMP n00b guide"
Description = "Setting up a basic LEMP stack on Arch Linux"
categories = ["Linux","Devops"]
series = ["Install guides"]
draft = false

+++

### Nginx
```
aurman -S nginx
```
**Note:** the default path for your web content on Arch is `/usr/share/nginx/html/`.

##### Start Nginx on boot:
```
sudo systemctl enable nginx.service
```


### MariaDB
```
aurman -S mariadb
```

**Note:** Before starting the MariaDB server you need to issue the following command:
```
mysql_install_db --user=mysql --basedir=/usr --datadir=/var/lib/mysql
```

##### Importing a large database:
```
mysql -u {DB-USER-NAME} -p {DB-NAME} < {db.file.sql path}
```

##### Start MariaDB on boot:
```
sudo systemctl enable mariadb.service
```


### PHP
```
aurman -S php-fpm
```

In order for Nginx to handle PHP properly you need to tell it where to look for the php-fpm service.
In your `/etc/nginx/nginx.conf` add the following to the bottom of each location block.
```
include php.conf;
```

Since we pointed our nginx.conf to a `/etc/nginx/php.conf` configuration it's neccesary to create this file as well:
```
location ~ \.php$ {
    try_files $uri $document_root$fastcgi_script_name =404;
    fastcgi_pass unix:/run/php-fpm/php-fpm.sock;
    fastcgi_index index.php;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    include fastcgi.conf;

    # prevention for httpoxy vulnerability: https://httpoxy.org/
    fastcgi_param HTTP_PROXY "";
}
```

##### Start PHP-FPM on boot:
```
sudo systemctl enable php-fpm.service
```
