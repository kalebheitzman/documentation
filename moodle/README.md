# Moodle Install Instructions

### Create and ssh into the VM

```bash
multipass launch --name moodle --cpus 4 --disk 64G --memory 8G -vvvv
multipass sh moodle
```

### Install PHP

```bash
sudo apt-get update
sudo apt-get install -y php8.1-cli php8.1-curl php8.1-fpm php8.1-gd \
    php8.1-imagick php8.1-intl php8.1-mbstring php8.1-ldap php8.1-opcache \
    php8.1-pgsql php8.1-pspell php8.1-redis php8.1-soap php8.1-xml \
    php8.1-xmlrpc php8.1-zip
```

### Install additional libs

```bash
sudo apt-get intall git aspell clamav graphviz ghostscript
```
