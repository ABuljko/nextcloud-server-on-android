# 📱☁️ Nextcloud Server on Android

> Self-hosted Nextcloud running on a **Samsung Galaxy A22 5G** via Termux — fully LAN-accessible, with Apache, PHP-FPM, and MariaDB.

![Status](https://img.shields.io/badge/status-running-brightgreen) ![Platform](https://img.shields.io/badge/platform-Android-blue) ![Stack](https://img.shields.io/badge/stack-Termux%20%7C%20Apache%20%7C%20PHP%208.5%20%7C%20MariaDB-informational)

---

## 🎯 Goal

Turn a mid-range Android phone into a self-hosted cloud storage server accessible from any device on the same network. No root required.

---

## 🧰 Software Stack

| Component     | Version |
| ------------- | ------- |
| Termux        | F-Droid |
| Apache        | 2.4     |
| PHP / PHP-FPM | 8.5     |
| MariaDB       | 12.2    |
| Nextcloud     | latest  |

---

# 🚀 Installation

## 1. Update Termux

```bash
pkg update && pkg upgrade
```

---

## 2. Install packages

```bash
pkg install php php-apache mariadb wget unzip
```

---

## 3. Start MariaDB and create the database

### Start MariaDB

```bash
mariadbd-safe &
```

### Open MariaDB

```bash
mariadb
```

### Create database and user

```sql
CREATE DATABASE nextcloud;
CREATE USER 'ncuser'@'localhost' IDENTIFIED BY 'ncpass';
GRANT ALL PRIVILEGES ON nextcloud.* TO 'ncuser'@'localhost';
FLUSH PRIVILEGES;
```

---

## 4. Download Nextcloud

```bash
wget https://download.nextcloud.com/server/releases/latest.zip
unzip latest.zip -d $PREFIX/share/apache2/default-site/htdocs/
```

---

# 🔧 Critical Fixes

These are the issues encountered and how they were resolved.

---

## GD Extension

```bash
pkg reinstall php-gd
pkill php-fpm
php-fpm
```

---

## MariaDB Recovery (corrupt data directory)

### Go to MariaDB data directory

```bash
cd $PREFIX/var/lib/mysql
```

### Remove corrupted system database

```bash
rm -rf mysql
rm ibdata1 ib_logfile0
rm -f undo*
```

### Reinitialize MariaDB

```bash
mariadb-install-db --user=$(whoami) --datadir=$PREFIX/var/lib/mysql
```

---

## Database Connection

Use `127.0.0.1` instead of `localhost` in Nextcloud's installer.

### Correct database host

```text
127.0.0.1
```

---

## PHP Memory (`php.ini`)

### Open php.ini

```bash
nano /data/data/com.termux/files/usr/etc/php/php.ini
```

### Add or modify

```ini
memory_limit = 512M
```

### Restart PHP-FPM

```bash
pkill php-fpm
php-fpm
```

---

## Apache Binding (`httpd.conf`)

### Open Apache config

```bash
nano $PREFIX/etc/apache2/httpd.conf
```

### Change

```apache
Listen 0.0.0.0:8080
```

---

## Trusted Domains (`config/config.php`)

### Open Nextcloud config

```bash
cd /data/data/com.termux/files/usr/share/apache2/default-site/htdocs/nextcloud/config
nano config.php
```

### Find this section

```php
'trusted_domains' =>
  array (
    0 => 'localhost',
  ),
```

### Replace with YOUR PHONE'S WIFI IP

```php
'trusted_domains' =>
  array (
    0 => 'localhost',
    1 => '192.168.2.112',
  ),
```

> Replace `192.168.2.112` with the actual Wi-Fi IP of your phone.

---

## Find Your Phone's Wi-Fi IP

### Method 1 (recommended)

```bash
ifconfig wlan0
```

Look for:

```text
inet 192.168.X.X
```

---

### Method 2

```bash
hostname -I
```

---

### Method 3 (Android Settings)

Settings → Wi-Fi → Connected Network → IP Address

---

# ▶️ Running the Server

## Start everything

```bash
mariadbd-safe &
php-fpm &
httpd
```

---

## Stop everything

```bash
pkill mariadbd
pkill php-fpm
pkill httpd
```

---

# 🌐 Access

| Context             | URL                       |
| ------------------- | ------------------------- |
| On-device           | `http://127.0.0.1:8080`   |
| LAN (other devices) | `http://192.168.X.X:8080` |

---

# ⚡ Optimization

Keep the server alive when the screen is off and limit memory usage.

## Prevent Android from sleeping Termux

```bash
termux-wake-lock
```

---

## Reduce PHP-FPM memory usage

### Open PHP-FPM config

```bash
nano $PREFIX/etc/php-fpm.d/www.conf
```

### Change

```ini
pm.max_children = 2
```

---

# 📋 Summary

A fully working Nextcloud server running on Android with LAN access, MariaDB, and PHP-FPM correctly configured. No root, no cloud bills — just an old phone and Termux.

---
