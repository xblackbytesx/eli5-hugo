+++
Description = ""
Tags = []
Categories = []
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


##### Start PHP-FPM on boot:
```
sudo systemctl enable php-fpm.service
```
