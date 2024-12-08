# Writ-305
WP# 3
<br>
<br>



# Part: 1
# Dependency Installation 
- __Step: 1 # Add "add-apt-repository" command__
```shell
apt -y install software-properties-common curl apt-transport-https ca-certificates gnupg
```
<br>

- __Step: 2 # Add additional repositories for PHP (Ubuntu 20.04 and Ubuntu 22.04)__

```shell
LC_ALL=C.UTF-8 add-apt-repository -y ppa:ondrej/php
```
<br>

- __Step: 3 # Add Redis official APT repository__

```shell
curl -fsSL https://packages.redis.io/gpg | sudo gpg --dearmor -o /usr/share/keyrings/redis-archive-keyring.gpg
```

```shell
echo "deb [signed-by=/usr/share/keyrings/redis-archive-keyring.gpg] https://packages.redis.io/deb $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/redis.list
```
<br>

- __Step: 4 # MariaDB repo setup script (Ubuntu 20.04)y__

```shell
curl -sS https://downloads.mariadb.com/MariaDB/mariadb_repo_setup | sudo bash
```
<br>

- __Step: 5 # Update repositories list__
  
```shell
apt update
```
<br>

- __Step: 6 # Update repositories list__
  
```shell
apt -y install php8.3 php8.3-{common,cli,gd,mysql,mbstring,bcmath,xml,fpm,curl,zip} mariadb-server nginx tar unzip git redis-server
```
<br>
<br>

# Part: 2
# Installing Composer
<br>

- __Composer is a dependency manager for PHP that allows us to ship everything you'll need code wise to operate the Panel. You'll need composer installed before continuing in this process.__
- 
```shell
curl -sS https://getcomposer.org/installer | sudo php -- --install-dir=/usr/local/bin --filename=composer
```

<br>
<br>

# Part: 3
# Download Files
<br>

- __Step: 1 # This command creates a directory at /var/www/pterodactyl__

```shell
mkdir -p /var/www/pterodactyl
```
<br>

- __Step: 2 # This command changes the current working directory to /var/www/pterodactyl__

```shell
cd /var/www/pterodactyl
```
<br>

- __Step: 3 # This command downloads the latest Pterodactyl Panel archive from GitHub and saves it__

```shell
curl -Lo panel.tar.gz https://github.com/pterodactyl/panel/releases/latest/download/panel.tar.gz
```
<br>

- __Step: 4 # This command extracts the contents of the panel.tar.gz archive.__
  
```shell
tar -xzvf panel.tar.gz
```
<br>

- __Step: 5 # sets the file permissions for the storage/ and bootstrap/cache/ directories.__

```shell
chmod -R 755 storage/* bootstrap/cache/
```

<br>
<br>


# Part: 4
# Database Configuration


<br>

- __Step: 1 # opens the MySQL client as the root user.__
```shell
mysql -u root -p
```

<br>

- __Step: 2 # creates a new database user named pterodactyl. Replace yourPassword with your own password.__
```shell
CREATE USER 'pterodactyl'@'127.0.0.1' IDENTIFIED BY 'yourPassword';
```
<br>

- __Step: 3 # creates a new database named panel.__
```shell
CREATE DATABASE panel;
```
<br>

- __Step: 4 # grants the pterodactyl user full permissions to the panel database.__
```shell
GRANT ALL PRIVILEGES ON panel.* TO 'pterodactyl'@'127.0.0.1' WITH GRANT OPTION;
exit
```

<br>

- __Step: 5 # copies the example environment file .env.example to a new file__

```shell
cp .env.example .env
```
<br>

- __Step: 6 # installs all necessary PHP dependencies for the Pterodactyl Panel.__
```shell
COMPOSER_ALLOW_SUPERUSER=1 composer install --no-dev --optimize-autoloader
```
<br>

- __Step: 7 # generates a new encryption key for the application and saves it.__
```shell
php artisan key:generate --force
```

<br>
<br>

# Environment Configuration

```shell
php artisan p:environment:setup
php artisan p:environment:database

# To use PHP's internal mail sending (not recommended), select "mail". To use a
# custom SMTP server, select "smtp".
php artisan p:environment:mail
```
<br>
<br>

# Database Setup

```shell
php artisan migrate --seed --force
```

<br>
<br>
# Add The First User

```shell
php artisan p:user:make
```


<br>
<br>

# Set Permissions

```shell
# If using NGINX, Apache or Caddy (not on RHEL / Rocky Linux / AlmaLinux)
chown -R www-data:www-data /var/www/pterodactyl/*

# If using NGINX on RHEL / Rocky Linux / AlmaLinux
chown -R nginx:nginx /var/www/pterodactyl/*

# If using Apache on RHEL / Rocky Linux / AlmaLinux
chown -R apache:apache /var/www/pterodactyl/*
```
<br>
<br>

# Queue Listeners

<br>
<br>

# Crontab Configuration

```shell
* * * * * php /var/www/pterodactyl/artisan schedule:run >> /dev/null 2>&1
```

<br>
<br>

# Create Queue Worker

```shell
# Pterodactyl Queue Worker File
# ----------------------------------

[Unit]
Description=Pterodactyl Queue Worker
After=redis-server.service

[Service]
# On some systems the user and group might be different.
# Some systems use `apache` or `nginx` as the user and group.
User=www-data
Group=www-data
Restart=always
ExecStart=/usr/bin/php /var/www/pterodactyl/artisan queue:work --queue=high,standard,low --sleep=3 --tries=3
StartLimitInterval=180
StartLimitBurst=30
RestartSec=5s

[Install]
WantedBy=multi-user.target
```

If you are using redis for your system, you will want to make sure to enable that it will start on boot. You can do that by running the following command:

```shell
sudo systemctl enable --now redis-server
```
Finally, enable the service and set it to boot on machine start.

```shell
sudo systemctl enable --now pteroq.service
```
<br>
<br>

# Next Step: Webserver Configuration








```shell
rm /etc/nginx/sites-enabled/default
```

test


```shell
server {
    # Replace the example <domain> with your domain name or IP address![jest_logo](https://github.com/user-attachments/assets/e706b098-1d14-4aea-ac63-4cef5f237c4f)

    listen 80;
    server_name <domain>;
    return 301 https://$server_name$request_uri;
}

server {
    # Replace the example <domain> with your domain name or IP address
    listen 443 ssl http2;
    server_name <domain>;

    root /var/www/pterodactyl/public;
    index index.php;

    access_log /var/log/nginx/pterodactyl.app-access.log;
    error_log  /var/log/nginx/pterodactyl.app-error.log error;

    # allow larger file uploads and longer script runtimes
    client_max_body_size 100m;
    client_body_timeout 120s;

    sendfile off;

    # SSL Configuration - Replace the example <domain> with your domain
    ssl_certificate /etc/letsencrypt/live/<domain>/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/<domain>/privkey.pem;
    ssl_session_cache shared:SSL:10m;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers "ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384";
    ssl_prefer_server_ciphers on;

    # See https://hstspreload.org/ before uncommenting the line below.
    # add_header Strict-Transport-Security "max-age=15768000; preload;";
    add_header X-Content-Type-Options nosniff;
    add_header X-XSS-Protection "1; mode=block";
    add_header X-Robots-Tag none;
    add_header Content-Security-Policy "frame-ancestors 'self'";
    add_header X-Frame-Options DENY;
    add_header Referrer-Policy same-origin;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass unix:/run/php/php8.3-fpm.sock;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param PHP_VALUE "upload_max_filesize = 100M \n post_max_size=100M";
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param HTTP_PROXY "";
        fastcgi_intercept_errors off;
        fastcgi_buffer_size 16k;
        fastcgi_buffers 4 16k;
        fastcgi_connect_timeout 300;
        fastcgi_send_timeout 300;
        fastcgi_read_timeout 300;
        include /etc/nginx/fastcgi_params;
    }

    location ~ /\.ht {
        deny all;
    }
}
```

test

```shell
sudo ln -s /etc/nginx/sites-available/pterodactyl.conf /etc/nginx/sites-enabled/pterodactyl.conf
```

```shell
sudo systemctl restart nginx
```

test

Starting Wings

```shell

```

```diff
- red
  ```
