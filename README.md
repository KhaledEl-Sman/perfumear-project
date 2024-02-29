# Perfumear-Project
Perfumear Project: A step-by-step guide for hosting WordPress themes on a Hetzner shared server. From server setup to domain attachment and SSL integration, this documentation provides a clear roadmap. Explore server optimization, connectivity issue resolution, and configuration of multiple WordPress instances for seamless web hosting. #Hetzner #WebHosting #WordPress #Nginx #MySQL

---

# [Perfumear](https://www.perfumear.com) Project Documentation

## 1. Buy Shared Server from Hetzner (CX21)

I Chose The CX21 Plan After Carefully Assessing Different Cloud Providers, Including DigitalOcean, Cloudways, Linode, Vultr, And AWS. Hetzner Emerged As The Optimal Solution, Offering A Combination Of Affordability, Simplicity, And Speed.

![PLAN_NAME](https://khaledel-sman.github.io/perfumear-project/plan.png)
![PLAN_SETUP](https://khaledel-sman.github.io/perfumear-project/Hetzner_setup.jpg)

## 2. Update And Upgrade
```bash
sudo apt update
sudo apt upgrade

```

## 3. Create Sudo User 
```bash
sudo adduser username && sudo usermod -aG sudo username && su - username
```

## 4. Install Nginx
```bash
sudo apt install nginx

```

## 5. Install MySQL
```bash
sudo apt install mysql-server

```

## 5. Install MySQL
```bash
sudo apt install mysql-server

```
Add Password To Root User Via MySQL-CLI
```bash
sudo mysql

```
```mysql
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'db_root_password';
EXIT;
```

## 6. Install PHP and Required Extensions
```bash
sudo apt install php-fpm php-mysql

```
### And Then Update Some Attributes Inside The *php.ini* File As Next:-
> upload_max_filesize = 200M
> post_max_size = 500M
> memory_limit = 512M
> cgi.fix_pathinfo = 0
> max_execution_time = 360
```bash
sudo nano /etc/php/<php_version>/fpm/php.ini
```
Then apply the changes
```bash
sudo nginx -t && sudo service nginx restart

```

## 7. Configure Nginx for WordPress
```bash
sudo nano /etc/nginx/sites-available/<domainName>
```
Then Add the following configuration, adjusting paths and server_name:
```sh
server {
    listen 80;
    server_name  **<domainName>**.com www.**<domainName>**.com;

    root /var/www/**wordpress**;
    index index.php index.html index.htm;

    location / {
        try_files $uri $uri/ /index.php?$args;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php**<php_version>**-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ /\.ht {
        deny all;
    }
}
```
Then Enable The Site And Restart Nginx:
```bash
sudo ln -s /etc/nginx/sites-available/domainName /etc/nginx/sites-enabled
sudo systemctl restart nginx
```

## 8. Create MySQL Database And User
```bash
sudo mysql -u root -p

```
```mysql
CREATE DATABASE db_name;
CREATE DATABASE blogs_db_name;
CREATE USER 'khaled'@'localhost' IDENTIFIED BY 'user_password';
GRANT ALL PRIVILEGES ON *.* TO 'khaled'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

## 9. Install WordPress And Set Correct Ownership
```bash
cd /var/www/ && sudo wget https://wordpress.org/latest.tar.gz && sudo tar -xvzf latest.tar.gz && sudo chown -R www-data:www-data /var/www/wordpress/ && sudo chmod -R 755 /var/www/wordpress/

```
Then Copy The Sample Configuration:
```bash
sudo cp /var/www/wordpress/wp-config-sample.php /var/www/wordpress/wp-config.php

```
And Edit On it By Providing Next Values
> define( 'DB_NAME', 'db_name' );
> define( 'DB_USER', 'db_user' );
> define( 'DB_PASSWORD', 'db_password' );
```bash
sudo nano /var/www/wordpress/wp-config.php

```
*Then Do The Same Thing With Blog Website Also*

## 10. Domain Setup and SSL
Attach The Domain To Cloudflare And Enable SSL certificate (HTTPS) By Changing The Nameserver DNS Records For The Domain Provider By Replacing Them With Cloudflare's Records.

## 11. Database Correction And Complete The Project
```bash
sudo mysql -u root -p

```
Then Edit Next Records
```mysql
USE <db_name>;
UPDATE wp_options SET option_value = '/' WHERE option_name = 'siteurl';
UPDATE wp_options SET option_value = '/' WHERE option_name = 'home';
USE <blogs_db_name>;
UPDATE wp_options SET option_value = '/blog' WHERE option_name = 'siteurl';
UPDATE wp_options SET option_value = '/blog' WHERE option_name = 'home';
EXIT;
```
