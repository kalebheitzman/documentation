# Moodle Install Instructions

## Create and ssh into the VM

```bash
multipass launch --name moodle --cpus 4 --disk 64G --memory 8G -vvvv
multipass sh moodle
```

## Install and configure PHP

Install PHP

```bash
sudo apt-get update
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
