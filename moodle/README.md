# Moodle Install Instructions

### Create and ssh into the VM

```bash
multipass launch --name moodle --cpus 4 --disk 64G --memory 8G -vvvv
multipass sh moodle
```

### Install and configure PHP

Install PHP

```bash
sudo apt-get update
sudo apt-get install -y php8.1-cli php8.1-curl php8.1-fpm php8.1-gd \
    php8.1-imagick php8.1-intl php8.1-mbstring php8.1-ldap php8.1-opcache \
    php8.1-pgsql php8.1-pspell php8.1-redis php8.1-soap php8.1-xml \
    php8.1-xmlrpc php8.1-zip
```

Customize the configuration at `/etc/php/8.1/fpm/conf.d/custom.ini`

```
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

### Install additional libs

```bash
sudo apt-get intall -y git aspell clamav graphviz ghostscript
```

### Redis Configuration

Redis is used for application caching

```bash


```
