# Perfumear-Project
Perfumear Project: A step-by-step guide for hosting WordPress themes on a Hetzner shared server. From server setup to domain attachment and SSL integration, this documentation provides a clear roadmap. Explore server optimization, connectivity issue resolution, and configuration of multiple WordPress instances for seamless web hosting. #Hetzner #WebHosting #WordPress #Nginx #MySQL

---

# [Perfumear](https://www.perfumear.com) Project Documentation

## 1. Buy Shared Server from Hetzner (CX21)

I chose the CX21 plan after carefully assessing different cloud providers, including DigitalOcean, Cloudways, Linode, Vultr, and AWS. Hetzner emerged as the optimal solution, offering a combination of affordability, simplicity, and speed.

![CX21 plan details](https://cdn13674555.blazingcdn.net/Docs/CX21.PNG)

## 2. Server Setup

### Add SSH Key and Open Port 22

```bash
ssh-keygen -t rsa -b 2048 -f perfumear_key
ssh-copy-id -i ~/.ssh/perfumear_key.pub root@91.107.195.217
```
Configure Firewalls to enable HTTP/HTTPS requests
```bash
sudo ufw allow 80
sudo ufw allow 443
sudo ufw enable
```
Connect to Server
```bash
ssh -i ~/.ssh/sna root@91.107.195.217
```
Add Superadmin User
```bash
adduser khaled
usermod -aG sudo khaled
su khaled
```
## 3. Install Nginx, and PHP
```bash
sudo apt update
sudo apt upgrade
sudo apt install -y nginx php-dom php-simplexml php-ssh2 php-xml \
php-xmlreader php-curl php-exif php-ftp php-gd php-iconv \
php-imagick php-json php-mbstring php-posix php-sockets \
php-tokenizer php-fpm php-mysql php-gmp php-intl php-cli
```
## 4. Install MySQL
```bash
MySQL
```
## 5. Install WordPress
```bash
cd /var/www/
sudo wget https://wordpress.org/latest.tar.gz
sudo tar -xvzf latest.tar.gz
sudo chown -R www-data:www-data /var/www/wordpress/
sudo chmod -R 755 /var/www/wordpress/
```
## 6. Configure WordPress
Create wp-config.php
Create the wp-config.php file inside /var/www/wordpress/:
Note: replace every '-----------------------------------------' with its value, I removed theme due to security purposes. and you can create your own values for the authentication keys by generating new values using *wp-cli*.
```php
<?php
/**
 * The base configuration for WordPress
 *
 * The wp-config.php creation script uses this file during the installation.
 * You don't have to use the web site, you can copy this file to "wp-config.php"
 * and fill in the values.
 *
 * This file contains the following configurations:
 *
 * * Database settings
 * * Secret keys
 * * Database table prefix
 * * ABSPATH
 *
 * @link https://wordpress.org/documentation/article/editing-wp-config-php/
 *
 * @package WordPress
 */

// ** Database settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define( 'DB_NAME', ‘db_name’);

/** Database username */
define( 'DB_USER', ‘db_user’);

/** Database password */
define( 'DB_PASSWORD', ‘db_password’);

/** Database hostname */
define( 'DB_HOST', 'localhost' );

/** Database charset to use in creating database tables. */
define( 'DB_CHARSET', 'utf8mb4' );

/** The database collate type. Don't change this if in doubt. */
define( 'DB_COLLATE', '' );

/**#@+
 * Authentication unique keys and salts.
 *
 * Change these to different unique phrases! You can generate these using
 * the {@link https://api.wordpress.org/secret-key/1.1/salt/ WordPress.org secret-key service}.
 *
 * You can change these at any point in time to invalidate all existing cookies.
 * This will force all users to have to log in again.
 *
 * @since 2.6.0
 */
define( 'AUTH_KEY',         '-----------------------------------------' );
define( 'SECURE_AUTH_KEY',  '-----------------------------------------' );
define( 'LOGGED_IN_KEY',    '-----------------------------------------' );
define( 'NONCE_KEY',        '-----------------------------------------' );
define( 'AUTH_SALT',        '-----------------------------------------' );
define( 'SECURE_AUTH_SALT', '-----------------------------------------' );
define( 'LOGGED_IN_SALT',   '-----------------------------------------' );
define( 'NONCE_SALT',       '-----------------------------------------' );

/**#@-*/

/**
 * WordPress database table prefix.
 *
 * You can have multiple installations in one database if you give each
 * a unique prefix. Only numbers, letters, and underscores please!
 */
$table_prefix = 'wp_';

/**
 * For developers: WordPress debugging mode.
 *
 * Change this to true to enable the display of notices during development.
 * It is strongly recommended that plugin and theme developers use WP_DEBUG
 * in their development environments.
 *
 * For information on other constants that can be used for debugging,
 * visit the documentation.
 *
 * @link https://wordpress.org/documentation/article/debugging-in-wordpress/
 */
define( 'WP_DEBUG', false );

/* Add any custom values between this line and the "stop editing" line. */



/* That's all, stop editing! Happy publishing. */

/** Absolute path to the WordPress directory. */
if ( ! defined( 'ABSPATH' ) ) {
        define( 'ABSPATH', __DIR__ . '/' );
}

/** Sets up WordPress vars and included files. */
require_once ABSPATH . 'wp-settings.php';
?>
```

Access the Database and Create DB for WordPress
```bash
sudo mysql -u root -p
 > CREATE DATABASE dbname;
 > CREATE USER 'dbuser'@'localhost' IDENTIFIED WITH mysql_native_password BY 'dbpassword';
 > GRANT ALL ON dbname.* TO 'dbuser'@'localhost' WITH GRANT OPTION;
 > FLUSH PRIVILEGES;
 > EXIT
```
## 7. Nginx Configuration
Create config file:
```bash
sudo nano /etc/nginx/sites-enabled/perfumear
```
config file content:
```bash
server {
    listen 80;
    listen [::]:80;
    server_name  perfumear.com www.perfumear.com;
    root /var/www/wordpress;
    index  index.php index.html index.htm;
    access_log /var/log/nginx/wpress_access.log;
    error_log /var/log/nginx/wpress_error.log;

    client_max_body_size 100M;
    autoindex off;
    location / {
        try_files $uri $uri/ /index.php?$args;
    }

    location ~ \.php$ {
         include snippets/fastcgi-php.conf;
         fastcgi_pass unix:/var/run/php/php-fpm8.1.sock;
         fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
         include fastcgi_params;
    }
}
```
Then apply the changes:
```bash
sudo nginx -t
sudo service nginx restart
```
## 8. Domain Setup and SSL
Complete WordPress installation by providing DB configurations. Attach the domain to Cloudflare to add SSL certificate.
## 9. SSH Connection Issue
If facing SSH connection issues:
```bash
sudo ufw allow 22
sudo systemctl restart ssh
```
## 10. WordPress Themes and Plugins
Purchase and upload themes/plugins from WordPress admin panel. If facing file size limit issues, update PHP config file.
```bash
sudo nano /etc/php/8.1/fpm/php.ini
```
### Update some attributes inside the file as next:-
upload_max_filesize = 200M,
post_max_size = 500M,
memory_limit = 512M,
cgi.fix_pathinfo = 0,
max_execution_time = 360
Then apply the changes
```bash
sudo systemctl restart php8.1-fpm.service
```

## 11. SSH Connection Timeout Issue
If facing SSH connection timeout issue:
```bash
sudo ufw status
sudo ufw enable
sudo ufw allow 22
sudo systemctl restart ssh
```
## 12. Additional Blog Setup
Create a new directory for blogs:
```bash
sudo mkdir /var/www/wordpress/blog
```
Then install blog WordPress:
```bash
sudo apt update
sudo apt install curl
curl -O https://wordpress.org/latest.tar.gz
# ... (Follow the steps for WordPress installation)
```
## 13. Theme Activation and Database Correction
(If the site gives error (too many requests) but the admin panel works good, update next database records)
```bash
sudo mysql -u root -p
 > USE db_name;
 > UPDATE wp_options SET option_value = '/' WHERE option_name = 'siteurl';
 > UPDATE wp_options SET option_value = '/' WHERE option_name = 'home';
```
