# 🚀 Nextcloud Installation on AWS EC2 (Ubuntu 22.04)

This repository contains **step-by-step notes and commands** to install **Nextcloud on AWS EC2** using **Apache, PHP 8.1, and MariaDB**.

Tested and verified setup for **future reuse**.

---

## 🧱 1. AWS EC2 INSTANCE SETUP

- **AMI**: Ubuntu Server 22.04 LTS  
- **Instance Type**:  
  - `t2.micro` (testing)  
  - `t3.small` (recommended)
- **Storage**: Minimum 20 GB
- **Security Group (Inbound Rules)**:
  - SSH – 22 – My IP
  - HTTP – 80 – Anywhere
  - HTTPS – 443 – Anywhere
- **Key Pair**: `.pem` file (keep safe)

---

## 🔐 2. CONNECT TO EC2

```bash
ssh -i key.pem ubuntu@PUBLIC_IP

🔄 3. SYSTEM UPDATE
sudo apt update && sudo apt upgrade -y

🌐 4. APACHE INSTALLATION
sudo apt install apache2 -y
sudo systemctl enable apache2
sudo systemctl start apache2


Test in browser:

http://PUBLIC_IP

🐘 5. PHP 8.1 INSTALLATION (Nextcloud Compatible)
sudo add-apt-repository ppa:ondrej/php -y
sudo apt update

sudo apt install php8.1 libapache2-mod-php8.1 \
php8.1-cli php8.1-common php8.1-mysql php8.1-xml \
php8.1-curl php8.1-zip php8.1-mbstring php8.1-gd \
php8.1-intl php8.1-bz2 -y


Disable other PHP versions (if installed):

sudo a2dismod php8.3
sudo a2enmod php8.1
sudo update-alternatives --set php /usr/bin/php8.1
sudo systemctl restart apache2


Verify:

php -v

🗄️ 6. MARIADB INSTALLATION
sudo apt install mariadb-server -y
sudo mysql_secure_installation


(Default options are fine)

🗃️ 7. DATABASE SETUP FOR NEXTCLOUD
sudo mysql

CREATE DATABASE nextcloud CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;
CREATE USER 'nextclouduser'@'localhost' IDENTIFIED BY 'Nc@12345';
GRANT ALL PRIVILEGES ON nextcloud.* TO 'nextclouduser'@'localhost';
FLUSH PRIVILEGES;
EXIT;


Test database login:

mysql -u nextclouduser -p

☁️ 8. DOWNLOAD & INSTALL NEXTCLOUD
cd /tmp
wget https://download.nextcloud.com/server/releases/latest.zip
sudo apt install unzip -y
unzip latest.zip
sudo mv nextcloud /var/www/html/


Set permissions:

sudo chown -R www-data:www-data /var/www/html/nextcloud
sudo chmod -R 755 /var/www/html/nextcloud

⚙️ 9. APACHE VIRTUAL HOST CONFIGURATION
sudo nano /etc/apache2/sites-available/nextcloud.conf

<VirtualHost *:80>
    ServerName PUBLIC_IP
    DocumentRoot /var/www/html/nextcloud

    <Directory /var/www/html/nextcloud>
        Require all granted
        AllowOverride All
        Options FollowSymLinks MultiViews
    </Directory>
</VirtualHost>


Enable configuration:

sudo a2dissite 000-default.conf
sudo a2ensite nextcloud.conf
sudo a2enmod rewrite headers env dir mime
sudo systemctl restart apache2

🌐 10. NEXTCLOUD WEB INSTALLATION

Open browser:

http://PUBLIC_IP

Admin Account

Username: admin

Password: Strong password

Data Folder
/var/www/html/nextcloud/data

Database Configuration

Database user: nextclouduser

Database password: Nc@12345

Database name: nextcloud

Database host: localhost

Click Install

⚡ 11. PHP PERFORMANCE TUNING (RECOMMENDED)
sudo nano /etc/php/8.1/apache2/php.ini


Update values:

memory_limit = 512M
upload_max_filesize = 2G
post_max_size = 2G
max_execution_time = 360


Restart Apache:

sudo systemctl restart apache2

⏱️ 12. NEXTCLOUD CRON JOB (IMPORTANT)
sudo crontab -u www-data -e


Add:

*/5 * * * * php -f /var/www/html/nextcloud/cron.php

📋 13. LOG FILES (TROUBLESHOOTING)
sudo tail -f /var/www/html/nextcloud/data/nextcloud.log

sudo tail -f /var/log/apache2/error.log

✅ FINAL STATUS

✔ AWS EC2 Ubuntu 22.04

✔ Apache Web Server

✔ PHP 8.1 (Nextcloud compatible)

✔ MariaDB

✔ Nextcloud (Latest)

✔ Ready for testing / demo use

📌 NEXT IMPROVEMENTS (Optional)

HTTPS using Let’s Encrypt

AWS Security Hardening

S3 / External Storage

Redis Caching

Nextcloud Performance Optimization
