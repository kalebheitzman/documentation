# Moodle Install Instructions

## Create and ssh into the VM

```bash
multipass launch --name moodle --cpus 4 --disk 64G --memory 8G -vvvv
multipass sh moodle
```

## Initial update

```bash
sudo apt-get update && sudo apt-get install -y software-properties-common
```

## Install and configure PHP

Install PHP

```bash
sudo apt-get install -y php8.1-cli php8.1-curl php8.1-fpm php8.1-gd \
    php8.1-imagick php8.1-intl php8.1-mbstring php8.1-ldap php8.1-opcache \
    php8.1-pgsql php8.1-pspell php8.1-redis php8.1-soap php8.1-xml \
    php8.1-xmlrpc php8.1-zip
```

Customize the configuration at `/etc/php/8.1/fpm/conf.d/custom.ini`

```bash
sudo nano /etc/php/8.1/fpm/conf.d/custom.ini
```

```ini
[php]
file_uploads = On
allow_url_fopen = On
short_open_tag = On
memory_limit = 256M
cgi.fix_pathinfo = 0
post_max_size = 200M
upload_max_filesize = 200M
max_execution_time = 600
date.timezone = America/New_York
max_input_vars = 5000
```

Reload the configuration

```bash
sudo systemctl reload php8.1-fpm.service
sudo systemctl status php8.1-fpm
```

## Install additional libs

These are additional libraries used by Moodle

```bash
sudo apt-get intall -y git aspell clamav graphviz ghostscript
```

## Install nginx

```bash
sudo apt-get install -y nginx
```

## Redis Configuration

Redis is used for application caching

```bash
sudo apt-get install -y redis-server
sudo systemctl status redis
```

## PostgreSQL Configuration

Install postgres

```bash
sudo apt-get install -y postgresql postgresql-contrib
```

Create user and database. You will be prompted for a password.

```bash
sudo -u postgres createuser moodle --no-createdb \
   --no-superuser --no-createrole --pwprompt
sudo -u postgres createdb moodle_production --owner=moodle
```

## Install Nginx

Install Nginx. Configuring Nginx will come after installing Moodle.

```bash
sudo apt-get install -y nginx
```

## Install Moodle

Installing via git

```bash
sudo mkdir -p /opt/moodle
sudo chown ubuntu:ubuntu /opt/moodle
git clone git://git.moodle.org/moodle.git /opt/moodle
```

Checking out the current version

```bash
cd /opt/moodle
sudo git branch --track MOODLE_402_STABLE origin/MOODLE_402_STABLE
sudo git checkout MOODLE_402_STABLE
sudo chown -R www-data /opt/moodle
```

Create the moodledata folder

```bash
sudo mkdir -p /var/moodledata
sudo chown -R www-data /var/moodledata
```

## Configuring Nginx

```conf
server {
    listen          80;
    listen          [::]:80;
    server_name     _;
    return          301 https://$host$request_uri;
}

server {
    # server information
    listen          443         ssl http2;
    listen          [::]:443    ssl http2;
    server_name _;

    # SSL
    ssl_protocols               TLSv1 TLSv1.1 TLSv1.2;
    ssl_prefer_server_ciphers   on;
    ssl_ciphers                 "ECDH+AESGCM:ECDH+AES256:ECDH+AES128:!ADH:!AECDH:!MD5;";

    ssl_certificate             /etc/ssl/certs/moodle.local.pem;
    ssl_certificate_key         /etc/ssl/certs/moodle.local-key.pem;
    # End SSL

    # Logging
    access_log  /var/log/nginx/access.log;
    error_log   /var/log/nginx/error.log;
    # End Logging

    # Configuration
    root /opt/moodle;
    index index.php;
    client_max_body_size 500M;
    autoindex off;

    # This passes 404 pages to Moodle so they can be themed
    error_page 404 /error/index.php;
    error_page 403 =404 /error/index.php;

    # Hide all dot files but allow "Well-Known URIs" as per RFC 5785
    location ~ /\.(?!well-known).* {
        return 404;
    }

    # Moodle rules.
    location / {
        try_files $uri $uri/ =404;
    }

    # Dataroot
    location /dataroot/ {
        internal;
        alias /var/moodledata/;
    }

    # Pass all .php files onto a php-fpm/php-fcgi server.
    location ~ [^/].php(/|$) {
        fastcgi_split_path_info   ^(.+\.php)(/.+)$;
        fastcgi_index             index.php;
        fastcgi_pass              unix:/var/run/php/php8.1-fpm.sock;
        include                   fastcgi_params;
        fastcgi_param             PATH_INFO $fastcgi_path_info;
        fastcgi_param             SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }

    # This should be after the php fpm rule and very close to the last nginx ruleset.
    # Don't allow direct access to various internal files. See MDL-69333
    location ~ (/vendor/|/node_modules/|composer\.json|/readme|/README|readme\.txt|/upgrade\.txt|db/install\.xml|/fixtures/|/behat/|phpunit\.xml|\.lock|environment\.xml) {
        deny all;
        return 404;
    }
}
```
