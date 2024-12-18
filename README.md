# How to Install Pterodactyl Panel on Your Virtual Server
                                                    
<img src="https://raw.githubusercontent.com/JakeClapper/Writ-305/main/pterodactyl.png?raw=true" alt="Sublime's custom image"/>

<br>


 # __Requirements Before Installing__
 - Optional: A DNS provider like __Squarespace__.
 - Hosting with SSH Access: Example - __OVHCloud__.
 - Linux OS: Use Ubuntu 20.04 LTS or newer.
 - Optional: __MobaXTerm__ for SSH connections.

<br>

# __Accessing Your Virtual Server Via MobaXterm__

<br>

__Step 1:__ Getting Started with MobaXterm.
 - Open MobaXTerm.
 - Click on Sessions in the toolbar, then choose New Session.

<img src="https://raw.githubusercontent.com/JakeClapper/Writ-305/main/mobx.png?raw=true" alt="Sublime's custom image"/>

<br>

__Step 2:__ Accessing the Terminal.
 - In the session setup, select SSH.
 - Enter your virtual server's IP Address.
 - Click OK.
<img src="https://raw.githubusercontent.com/JakeClapper/Writ-305/main/sshpic.png?raw=true" alt="Sublime's custom image"/>
<br>

__Step 3:__ Loggin in to your Virtual Server.
 - When prompted for Login; Type: root.
 - When prompted for Password: Paste in your SSH private key.
<img src="https://raw.githubusercontent.com/JakeClapper/Writ-305/main/logins.png?raw=true" alt="Sublime's custom image"/>
 - Once successfully connected, you're ready to proceed with the installation guide.

<br>
<br>



# Part: 1
# Dependency Installation 
__Step 1:__ Add "add-apt-repository" command.
```shell
apt -y install software-properties-common curl apt-transport-https ca-certificates gnupg
```
<br>

 __Step 2:__ Add additional repositories for PHP (Ubuntu 20.04 and Ubuntu 22.04).

```shell
LC_ALL=C.UTF-8 add-apt-repository -y ppa:ondrej/php
```
<br>

__Step 3:__ Add Redis official APT repository.

```shell
curl -fsSL https://packages.redis.io/gpg | sudo gpg --dearmor -o /usr/share/keyrings/redis-archive-keyring.gpg
```

```shell
echo "deb [signed-by=/usr/share/keyrings/redis-archive-keyring.gpg] https://packages.redis.io/deb $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/redis.list
```
<br>

__Step 4:__ MariaDB repo setup script (Ubuntu 20.04).

```shell
curl -sS https://downloads.mariadb.com/MariaDB/mariadb_repo_setup | sudo bash
```
<br>

__Step 5:__ Update repositories list.
  
```shell
apt update
```
<br>

__Step 6:__ Update repositories list.
  
```shell
apt -y install php8.3 php8.3-{common,cli,gd,mysql,mbstring,bcmath,xml,fpm,curl,zip} mariadb-server nginx tar unzip git redis-server
```
<br>
<br>

# Part: 2
# Installing Composer
<br>

__Step 1:__ Downloads and installs Composer, a dependency manager for PHP.

```shell
curl -sS https://getcomposer.org/installer | sudo php -- --install-dir=/usr/local/bin --filename=composer
```

<br>
<br>

# Part: 3
# Download Files
<br>

__Step 1:__ Create a directory at /var/www/pterodactyl.

```shell
mkdir -p /var/www/pterodactyl
```
<br>

__Step 2:__ Change the current working directory to /var/www/pterodactyl.

```shell
cd /var/www/pterodactyl
```
<br>

__Step 3:__ Download the latest Pterodactyl Panel archive from GitHub and saves it.

```shell
curl -Lo panel.tar.gz https://github.com/pterodactyl/panel/releases/latest/download/panel.tar.gz
```
<br>

__Step 4:__ Extract the contents of the panel.tar.gz archive.
  
```shell
tar -xzvf panel.tar.gz
```
<br>

__Step 5:__ Set the file permissions for the storage/ and bootstrap/cache/ directories.

```shell
chmod -R 755 storage/* bootstrap/cache/
```

<br>
<br>


# Part: 4
# Database Configuration


<br>

__Step 1:__ Open the MySQL client as the root user.
```shell
mysql -u root -p
```

<br>

__Step 2:__ Create a new database user named pterodactyl. Replace yourPassword with your own password.
```shell
CREATE USER 'pterodactyl'@'127.0.0.1' IDENTIFIED BY 'yourPassword';
```
<br>

__Step 3:__ Create a new database named panel.
```shell
CREATE DATABASE panel;
```
<br>

__Step 4:__ Grant the pterodactyl user full permissions to the panel database.
```shell
GRANT ALL PRIVILEGES ON panel.* TO 'pterodactyl'@'127.0.0.1' WITH GRANT OPTION;
exit
```

<br>

__Step 5:__ Copy the example environment file .env.example to a new file.

```shell
cp .env.example .env
```
<br>

__Step 6:__ Install all necessary PHP dependencies for the Pterodactyl Panel.
```shell
COMPOSER_ALLOW_SUPERUSER=1 composer install --no-dev --optimize-autoloader
```
<br>

__Step 7:__ Generate a new encryption key for the application and saves it.
```shell
php artisan key:generate --force
```

<br>
<br>

# Part: 5
# Environment Configuration
<br>

__Step 1:__ Set up the general environment for the Pterodactyl Panel.
```shell
php artisan p:environment:setup
```
<br>

__Step 2:__ Configure the database connection for the Pterodactyl Panel.
```shell
php artisan p:environment:database
```
<br>

__Step 3:__ Configure the mail settings for the Panel.
```shell
php artisan p:environment:mail
```
<br>
<br>

# Part: 6
# Database Setup

<br>

__Step 1:__ Run the database migrations to set up the necessary tables and structures.
```shell
php artisan migrate --seed --force
```


<br>

__Step 2:__ Create the first administrative user for the Pterodactyl Panel.
```shell
php artisan p:user:make
```


<br>
<br>

# Part: 8
# Set Permissions

<br>

__Step 1:__ Change the ownership of all files and directories.
```shell
chown -R www-data:www-data /var/www/pterodactyl/*
```
<br>
<br>


# Part: 9
# Crontab Configuration

<br>

__Step 1:__ Open the root user's crontab file for editing.
  
```shell
sudo crontab -e
```
<br>

__Step 2:__ Schedule a task to run every minute.
    
```shell
* * * * * php /var/www/pterodactyl/artisan schedule:run >> /dev/null 2>&1
```

<br>
<br>

# Part: 10
# Create Queue Worker

<br>

__Step 1:__ Create the pteroq.service file in the /etc/systemd/system/ directory.

```shell
sudo nano /etc/systemd/system/pteroq.service
```



<br>

__Step 2:__ # Inside the nano editor, paste the following contents.
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


<br>

__Step 3:__ Enable and start the Redis service.
```shell
sudo systemctl enable --now redis-server
```

<br>

__Step 4:__ Enable the service to start at boot and starts it.
```shell
sudo systemctl enable --now pteroq.service
```
<br>
<br>

# Part: 11
# Webserver Configuration


<br>

__Step 1:__ Remove the default NGINX configuration.
```shell
rm /etc/nginx/sites-enabled/default
```

<br>

__Step 2:__ Create a new configuration file in /etc/nginx/sites-available/ directory.
```shell
sudo nano /etc/nginx/conf.d/pterodactyl.conf
```


<br>

__Step 3:__ Inside the editor, paste the following configuration, replacing <domain> with your actual domain name.

```shell
server {
    

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
<br>

__Step 4:__ Press Ctrl + X, then press Y, and hit Enter to save the file.

<br>
<br>

# Part: 12
# Enabling Configuration

<br>

__Step 1:__ Create a symbolic link for the NGINX configuration.
```shell
sudo ln -s /etc/nginx/sites-available/pterodactyl.conf /etc/nginx/sites-enabled/pterodactyl.conf
```

<br>

__Step 2:__ Restart NGINX to apply the new configuration.
```shell
sudo systemctl restart nginx
```
<br>
<br>

# Part: 13
# Installing Docker

<br>

__Step 1:__ Install Docker on your system.
```shell
curl -sSL https://get.docker.com/ | CHANNEL=stable bash
```

<br>
<br>

# Part: 14
# Start Docker on Boot
<br>

__Step 1:__ Enable the Docker service to start automatically.
```shell
sudo systemctl enable --now docker
```



<br>
<br>

# Part: 15
# Enabling Swap

<br>

__Step 1:__ Open the /etc/default/grub file as a root user.
```shell
sudo nano /etc/default/grub
```

<br>

__Step 2:__ Find the line starting with GRUB_CMDLINE_LINUX_DEFAULT and add swapaccount=1.
```shell
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash swapaccount=1"
```

<br>

__Step 3:__ Update the bootloader with the new settings.
```shell
sudo update-grub
```

<br>

__Step 4:__  Restart the system.
```shell
sudo reboot
```

<br>
<br>

# Part: 16
# Installing Wings

<br>

__Step 1:__ Create the directory /etc/pterodactyl.
```shell
sudo mkdir -p /etc/pterodactyl
```

<br>

__Step 2:__ Download the Wings executable.
```shell
curl -L -o /usr/local/bin/wings "https://github.com/pterodactyl/wings/releases/latest/download/wings_linux_$([[ "$(uname -m)" == "x86_64" ]] && echo "amd64" || echo "arm64")"
```

<br>

__Step 3:__ Make the downloaded Wings executable runnable.
```shell
sudo chmod u+x /usr/local/bin/wings
```

<br>
<br>

# Part: 17
# Configure
<br>

- __Step 1:__ Create a Node on Your Panel.
  - Go to your Panel's administrative view
  - Select Nodes from the sidebar.
  - On the right side, click the Create New button.

![Wings Configuration Example](https://raw.githubusercontent.com/JakeClapper/Writ-305/main/wings_configuration_example.9f3fdd0b.png)
<br>

__Step 2:__ Access the Configuration Tab.
  - After creating a node, click on it to open its details.
  - Navigate to the Configuration tab.

<br>

__Step 2:__ Copy and Save the Configuration.
  - Copy the content of the code block provided in the Configuration tab.
  - Paste it into a new file called config.yml in /etc/pterodactyl.
  - Save the file.

<br>
<br>

# Part: 18
# Starting Wings
<br>

__Step 1:__ Start Wings in debug mode and confirm it's running without errors.
```shell
sudo wings --debug
```

<br>
<br>

# Part: 19
# Daemonizing
<br>

__Step 1:__ Place the following contents into a file named wings.service located in the /etc/systemd/system directory.
```shell
[Unit]
Description=Pterodactyl Wings Daemon
After=docker.service
Requires=docker.service
PartOf=docker.service

[Service]
User=root
WorkingDirectory=/etc/pterodactyl
LimitNOFILE=4096
PIDFile=/var/run/wings/daemon.pid
ExecStart=/usr/local/bin/wings
Restart=on-failure
StartLimitInterval=180
StartLimitBurst=30
RestartSec=5s

[Install]
WantedBy=multi-user.target
```

<br>

__Step 2:__ Reload systemd and start Wings.
```shell
sudo systemctl enable --now wings
```

<br>
<br>

# Part: 20
# Node Allocations

<br>

__Step 1:__ Create Allocations.
  - Go to the Panel.
  - Navigate to Nodes > [Your Node] > Allocation.
  - Add a new allocation by specifying the IP and port.


![Node Allocations](https://raw.githubusercontent.com/JakeClapper/Writ-305/main/node_allocations.323d67f2.png)

-__Important Notes__
- Do not use 127.0.0.1 (localhost) for allocations.
- Use the correct IP address based on your server's network configuration.


# Your Pterodactyl environment is now ready to deploy and manage servers! 🚀
