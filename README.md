# 📱☁️ Nextcloud Server on Android

> Self-hosted Nextcloud running on a **Samsung Galaxy A22 5G** via Termux — fully LAN-accessible, with Apache, PHP-FPM, and MariaDB.

![Status](https://img.shields.io/badge/status-running-brightgreen) ![Platform](https://img.shields.io/badge/platform-Android-blue) ![Stack](https://img.shields.io/badge/stack-Termux%20%7C%20Apache%20%7C%20PHP%208.5%20%7C%20MariaDB-informational)

---

## 🎯 Goal

Turn a mid-range Android phone into a self-hosted cloud storage server accessible from any device on the same network. No root required.

---

## 🧰 Software Stack

| Component | Version |
|-----------|---------|
| Termux | F-Droid |
| Apache | 2.4 |
| PHP / PHP-FPM | 8.5 |
| MariaDB | 12.2 |
| Nextcloud | latest |

---

## 🚀 Installation

### 1. Update Termux

```bash
pkg update && pkg upgrade
```

### 2. Install packages

```bash
pkg install php php-apache mariadb wget unzip
```

### 3. Start MariaDB and create the database

```bash
mariadbd-safe &
```

```sql
CREATE DATABASE nextcloud;
CREATE USER 'ncuser'@'localhost' IDENTIFIED BY 'ncpass';
GRANT ALL PRIVILEGES ON nextcloud.* TO 'ncuser'@'localhost';
FLUSH PRIVILEGES;
```

### 4. Download Nextcloud

```bash
wget https://download.nextcloud.com/server/releases/latest.zip
unzip latest.zip -d $PREFIX/share/apache2/default-site/htdocs/
```

---

## 🔧 Critical Fixes

These are the issues encountered and how they were resolved.

### GD Extension

```bash
pkg reinstall php-gd
pkill php-fpm
php-fpm
```

### MariaDB Recovery (corrupt data directory)

```bash
rm -rf mysql
rm ibdata1 ib_logfile0
rm -f undo*
mariadb-install-db --user=$(whoami) --datadir=$PREFIX/var/lib/mysql
```

### Database Connection

Use `127.0.0.1` instead of `localhost` in Nextcloud's config — Termux doesn't support Unix sockets the same way.

### PHP Memory (`php.ini`)

```ini
memory_limit = 512M
```

### Apache Binding (`httpd.conf`)

```apache
Listen 0.0.0.0:8080
```

### Trusted Domains (`config/config.php`)

```php
'trusted_domains' =>
  array (
    0 => 'localhost',
    1 => '192.168.2.*',
  ),
```

---

## ▶️ Running the Server

```bash
mariadbd-safe &
php-fpm &
httpd
```

---

## 🌐 Access

| Context | URL |
|---------|-----|
| On-device | `http://127.0.0.1:8080` |
| LAN (other devices) | `http://192.168.X.X:8080` |

---

## ⚡ Optimization

Keep the server alive when the screen is off and limit memory usage:

```bash
termux-wake-lock
```

```ini
pm.max_children = 2
```

---

## 📋 Summary

A fully working Nextcloud server running on Android with LAN access, MariaDB, and PHP-FPM correctly configured. No root, no cloud bills — just an old phone and Termux.

---

## 📄 Docs

Full setup documentation with visual formatting is available as a PDF in this repo: [`nextcloud_full_guide.pdf`](./setup_guide.pdf)
