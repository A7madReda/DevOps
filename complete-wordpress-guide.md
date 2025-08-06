# Complete WordPress Setup Guide

A comprehensive reference for setting up WordPress with LEMP stack (Linux, Nginx, MySQL, PHP) using both traditional installation and Docker containerization.

---

## Table of Contents

1. [Method 1: Traditional Installation (Native)](#method-1-traditional-installation-native)
2. [Method 2: Docker Installation](#method-2-docker-installation)
3. [Comparison: Native vs Docker](#comparison-native-vs-docker)
4. [Common Issues & Troubleshooting](#common-issues--troubleshooting)
5. [Performance Optimization](#performance-optimization)
6. [Security Best Practices](#security-best-practices)

---

# Method 1: Traditional Installation (Native)

## Prerequisites
- Ubuntu/Debian Linux server
- Root or sudo access
- Internet connection

## Step 1: System Update
```bash
# Update package list and upgrade system
sudo apt update && sudo apt upgrade -y

# Install essential packages
sudo apt install curl wget git unzip -y
```

## Step 2: Install Nginx
```bash
# Install Nginx
sudo apt install nginx -y

# Start and enable Nginx
sudo systemctl start nginx
sudo systemctl enable nginx

# Check status
sudo systemctl status nginx

# Test Nginx (should show welcome page)
curl http://localhost
```

**Expected Output:**
```html
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
```

## Step 3: Install MySQL
```bash
# Install MySQL Server
sudo apt install mysql-server -y

# Start and enable MySQL
sudo systemctl start mysql
sudo systemctl enable mysql

# Secure MySQL installation
sudo mysql_secure_installation
```

**During mysql_secure_installation:**
- Set root password: `Yes`
- Remove anonymous users: `Yes`
- Disallow root login remotely: `Yes`
- Remove test database: `Yes`
- Reload privilege tables: `Yes`

### Create WordPress Database
```bash
# Login to MySQL
sudo mysql -u root -p

# Inside MySQL prompt:
CREATE DATABASE wordpress_db;
CREATE USER 'wp_user'@'localhost' IDENTIFIED BY 'strong_password_here';
GRANT ALL PRIVILEGES ON wordpress_db.* TO 'wp_user'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

## Step 4: Install PHP
```bash
# Install PHP and required extensions
sudo apt install php-fpm php-mysql php-curl php-gd php-xml php-zip php-mbstring -y

# Check PHP version
php --version

# Check if PHP-FPM is running
sudo systemctl status php8.3-fpm
```

**Note:** Replace `php8.3-fpm` with your PHP version (e.g., `php8.1-fpm`, `php8.2-fpm`)

## Step 5: Configure Nginx for PHP
```bash
# Edit default Nginx configuration
sudo nano /etc/nginx/sites-available/default
```

**Replace content with:**
```nginx
server {
    listen 80 default_server;
    listen [::]:80 default_server;
    
    root /var/www/html;
    index index.php index.html index.htm index.nginx-debian.html;
    
    server_name _;
    
    location / {
        try_files $uri $uri/ =404;
    }
    
    # PHP handling
    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php8.3-fpm.sock;
    }
    
    location ~ /\.ht {
        deny all;
    }
}
```

```bash
# Test Nginx configuration
sudo nginx -t

# Reload Nginx
sudo systemctl reload nginx

# Test PHP
echo "<?php phpinfo(); ?>" | sudo tee /var/www/html/test.php
curl http://localhost/test.php
```

## Step 6: Download and Install WordPress
```bash
# Navigate to web directory
cd /var/www

# Download WordPress
sudo wget https://wordpress.org/latest.tar.gz

# Extract WordPress
sudo tar -xzf latest.tar.gz

# Set proper permissions
sudo chown -R www-data:www-data /var/www/wordpress
sudo chmod -R 755 /var/www/wordpress

# Create wp-config.php
cd /var/www/wordpress
sudo cp wp-config-sample.php wp-config.php
sudo nano wp-config.php
```

**Edit wp-config.php:**
```php
define('DB_NAME', 'wordpress_db');
define('DB_USER', 'wp_user');
define('DB_PASSWORD', 'strong_password_here');
define('DB_HOST', 'localhost');
```

## Step 7: Configure Nginx for WordPress
```bash
# Create WordPress site configuration
sudo nano /etc/nginx/sites-available/wordpress
```

**WordPress Nginx Configuration:**
```nginx
server {
    listen 80;
    server_name your_domain.com www.your_domain.com;
    root /var/www/wordpress;
    index index.php index.html index.htm;
    
    location / {
        try_files $uri $uri/ /index.php?$args;
    }
    
    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php8.3-fpm.sock;
    }
    
    location ~ /\.ht {
        deny all;
    }
    
    # WordPress security headers
    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }
}
```

```bash
# Enable WordPress site
sudo ln -s /etc/nginx/sites-available/wordpress /etc/nginx/sites-enabled/

# Disable default site (optional)
sudo rm /etc/nginx/sites-enabled/default

# Test and reload Nginx
sudo nginx -t
sudo systemctl reload nginx
```

## Step 8: Complete WordPress Installation
```bash
# Access WordPress installation
curl http://localhost/

# Or open in browser: http://your_server_ip/
```

**WordPress Installation Steps:**
1. Select language
2. Enter database details
3. Create admin account
4. Install WordPress

## Step 9: Install WordPress CLI (Optional)
```bash
# Download WP-CLI
curl -O https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar

# Make executable
chmod +x wp-cli.phar

# Move to system path
sudo mv wp-cli.phar /usr/local/bin/wp

# Test WP-CLI
wp --info

# Install WordPress via CLI
cd /var/www/wordpress
sudo -u www-data wp core install \
  --url="http://your_domain.com" \
  --title="My WordPress Site" \
  --admin_user="admin" \
  --admin_password="secure_password" \
  --admin_email="admin@example.com"
```

---

# Method 2: Docker Installation

## Prerequisites
- Docker Engine 20.10+
- Docker Compose 2.0+
- Git

## Step 1: Install Docker
```bash
# Update package index
sudo apt update

# Install required packages
sudo apt install apt-transport-https ca-certificates curl gnupg lsb-release -y

# Add Docker's official GPG key
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# Add Docker repository
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Update package index
sudo apt update

# Install Docker
sudo apt install docker-ce docker-ce-cli containerd.io docker-compose-plugin -y

# Add user to docker group
sudo usermod -aG docker $USER

# Start and enable Docker
sudo systemctl start docker
sudo systemctl enable docker

# Test Docker installation
docker --version
docker compose version
```

## Step 2: Create Project Structure
```bash
# Create project directory
mkdir wordpress-docker
cd wordpress-docker

# Create directory structure
mkdir -p nginx php wordpress

# Create necessary files
touch docker-compose.yml .env nginx/default.conf php/Dockerfile
```

## Step 3: Configure Environment Variables
```bash
# Create .env file
cat > .env << 'EOF'
# MySQL Configuration
MYSQL_ROOT_PASSWORD=SecureRootPass123!
MYSQL_DATABASE=wordpress
MYSQL_USER=wpuser
MYSQL_PASSWORD=SecureUserPass123!

# WordPress Configuration
WORDPRESS_DB_HOST=db
WORDPRESS_DB_NAME=wordpress
WORDPRESS_DB_USER=wpuser
WORDPRESS_DB_PASSWORD=SecureUserPass123!
EOF
```

## Step 4: Create PHP Dockerfile
```bash
# Create PHP Dockerfile with MySQL extensions
cat > php/Dockerfile << 'EOF'
FROM php:8.3-fpm

# Install system dependencies
RUN apt-get update && apt-get install -y \
    git \
    curl \
    libpng-dev \
    libonig-dev \
    libxml2-dev \
    zip \
    unzip \
    && rm -rf /var/lib/apt/lists/*

# Install PHP extensions required for WordPress
RUN docker-php-ext-install mysqli pdo pdo_mysql mbstring exif pcntl bcmath gd

# Enable MySQL extensions
RUN docker-php-ext-enable mysqli

# Set working directory
WORKDIR /var/www/html

# Set proper permissions
RUN chown -R www-data:www-data /var/www/html
EOF
```

## Step 5: Configure Nginx
```bash
# Create Nginx configuration
cat > nginx/default.conf << 'EOF'
server {
    listen 80;
    server_name localhost;
    root /var/www/html;
    index index.php index.html index.htm;
    
    # Increase client max body size for file uploads
    client_max_body_size 100M;
    
    location / {
        try_files $uri $uri/ /index.php?$args;
    }
    
    location ~ \.php$ {
        include fastcgi_params;
        fastcgi_pass php:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_read_timeout 300;
    }
    
    location ~ /\.ht {
        deny all;
    }
    
    # Static files caching
    location ~* \.(css|gif|ico|jpeg|jpg|js|png|webp|svg)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
        access_log off;
    }
    
    # Security headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header X-Content-Type-Options "nosniff" always;
}
EOF
```

## Step 6: Create Docker Compose Configuration
```bash
# Create docker-compose.yml
cat > docker-compose.yml << 'EOF'
version: "3.9"

services:
  # PHP-FPM Service
  php:
    build: ./php
    container_name: wordpress_php
    restart: unless-stopped
    volumes:
      - ./wordpress:/var/www/html
    depends_on:
      - db
    environment:
      WORDPRESS_DB_HOST: ${WORDPRESS_DB_HOST}
      WORDPRESS_DB_NAME: ${WORDPRESS_DB_NAME}
      WORDPRESS_DB_USER: ${WORDPRESS_DB_USER}
      WORDPRESS_DB_PASSWORD: ${WORDPRESS_DB_PASSWORD}
    networks:
      - wordpress_network

  # Nginx Service
  nginx:
    image: nginx:alpine
    container_name: wordpress_nginx
    restart: unless-stopped
    ports:
      - "80:80"
    volumes:
      - ./wordpress:/var/www/html
      - ./nginx/default.conf:/etc/nginx/conf.d/default.conf
    depends_on:
      - php
    networks:
      - wordpress_network

  # MySQL Service
  db:
    image: mysql:5.7
    container_name: wordpress_db
    restart: unless-stopped
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
      MYSQL_DATABASE: ${MYSQL_DATABASE}
      MYSQL_USER: ${MYSQL_USER}
      MYSQL_PASSWORD: ${MYSQL_PASSWORD}
    volumes:
      - db_data:/var/lib/mysql
    networks:
      - wordpress_network

# Named Volumes
volumes:
  db_data:

# Custom Networks
networks:
  wordpress_network:
    driver: bridge
EOF
```

## Step 7: Download WordPress
```bash
# Download and extract WordPress
wget https://wordpress.org/latest.tar.gz
tar -xzf latest.tar.gz
mv wordpress/* ./wordpress/
rm -rf wordpress latest.tar.gz

# Or download directly to wordpress directory
curl -o wordpress.tar.gz https://wordpress.org/latest.tar.gz
tar -xzf wordpress.tar.gz --strip-components=1 -C ./wordpress/
rm wordpress.tar.gz
```

## Step 8: Start Docker Services
```bash
# Build and start services
docker-compose up --build -d

# Check if services are running
docker-compose ps

# View logs
docker-compose logs -f

# View specific service logs
docker-compose logs nginx
docker-compose logs php
docker-compose logs db
```

**Expected Output:**
```
     Name                   Command               State          Ports
------------------------------------------------------------------------
wordpress_db      docker-entrypoint.sh mysqld   Up      3306/tcp
wordpress_nginx   /docker-entrypoint.sh ngin ... Up      0.0.0.0:80->80/tcp
wordpress_php     docker-php-entrypoint php-fpm  Up      9000/tcp
```

## Step 9: Complete WordPress Installation
```bash
# Test WordPress installation
curl http://localhost/

# Or access via browser: http://localhost/
```

## Step 10: WordPress CLI in Docker
```bash
# Install WP-CLI in PHP container
docker-compose exec php bash

# Inside container:
curl -O https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar
chmod +x wp-cli.phar
mv wp-cli.phar /usr/local/bin/wp

# Install WordPress via CLI
wp core install \
  --url="http://localhost" \
  --title="My Docker WordPress" \
  --admin_user="admin" \
  --admin_password="secure_password" \
  --admin_email="admin@example.com" \
  --allow-root

# Exit container
exit
```

## Docker Management Commands
```bash
# Start services
docker-compose up -d

# Stop services
docker-compose down

# Rebuild services
docker-compose up --build -d

# View running containers
docker-compose ps

# View all logs
docker-compose logs

# Follow logs in real-time
docker-compose logs -f

# Execute commands in containers
docker-compose exec php bash
docker-compose exec db mysql -u root -p

# Remove everything (including volumes)
docker-compose down -v

# Remove only containers (keep volumes)
docker-compose down

# Scale services (if needed)
docker-compose up -d --scale php=2
```

---

# Comparison: Native vs Docker

| Aspect | Native Installation | Docker Installation |
|--------|-------------------|-------------------|
| **Setup Time** | Longer (multiple packages) | Faster (pre-built images) |
| **System Impact** | Installs packages on host | Isolated containers |
| **Resource Usage** | Lower overhead | Slight overhead |
| **Portability** | Environment-specific | Highly portable |
| **Updates** | Manual package updates | Container image updates |
| **Backup** | Files + database dump | Volumes + containers |
| **Development** | One environment | Multiple environments |
| **Production** | Traditional deployment | Container orchestration |

---

# Common Issues & Troubleshooting

## Native Installation Issues

### 1. Nginx 404 Error
```bash
# Check Nginx configuration
sudo nginx -t

# Check file permissions
ls -la /var/www/wordpress/

# Fix permissions
sudo chown -R www-data:www-data /var/www/wordpress/
sudo chmod -R 755 /var/www/wordpress/
```

### 2. PHP Not Working
```bash
# Check PHP-FPM status
sudo systemctl status php8.3-fpm

# Check socket file
ls -la /run/php/php8.3-fpm.sock

# Test PHP processing
echo "<?php phpinfo(); ?>" | sudo tee /var/www/html/test.php
curl http://localhost/test.php
```

### 3. Database Connection Error
```bash
# Check MySQL status
sudo systemctl status mysql

# Test database connection
mysql -u wp_user -p wordpress_db

# Verify wp-config.php settings
grep "DB_" /var/www/wordpress/wp-config.php
```

## Docker Installation Issues

### 1. Container Won't Start
```bash
# Check container logs
docker-compose logs [service_name]

# Check container status
docker-compose ps

# Recreate containers
docker-compose down
docker-compose up -d
```

### 2. Database Connection Error
```bash
# Check if database container is running
docker-compose ps db

# Check database logs
docker-compose logs db

# Test database connection
docker-compose exec php ping db
```

### 3. Permission Issues
```bash
# Fix WordPress file permissions
docker-compose exec php chown -R www-data:www-data /var/www/html
docker-compose exec php chmod -R 755 /var/www/html
```

### 4. Port Conflicts
```bash
# Check what's using port 80
sudo netstat -tlnp | grep :80

# Use different port in docker-compose.yml
ports:
  - "8080:80"
```

## General WordPress Issues

### 1. Memory Limit Errors
```bash
# For native installation - edit PHP configuration
sudo nano /etc/php/8.3/fpm/php.ini

# Increase memory limit
memory_limit = 256M

# For Docker - modify PHP Dockerfile
RUN echo "memory_limit = 256M" > /usr/local/etc/php/conf.d/memory-limit.ini
```

### 2. File Upload Size Limit
```bash
# PHP configuration
upload_max_filesize = 100M
post_max_size = 100M

# Nginx configuration
client_max_body_size 100M;
```

### 3. SSL Certificate Issues
```bash
# For native installation with Let's Encrypt
sudo apt install certbot python3-certbot-nginx
sudo certbot --nginx -d your_domain.com

# For Docker with SSL
# Add SSL volume and configuration to docker-compose.yml
```

---

# Performance Optimization

## Native Installation Optimization

### 1. PHP-FPM Tuning
```bash
# Edit PHP-FPM pool configuration
sudo nano /etc/php/8.3/fpm/pool.d/www.conf

# Optimize process management
pm = dynamic
pm.max_children = 50
pm.start_servers = 10
pm.min_spare_servers = 5
pm.max_spare_servers = 35
```

### 2. Nginx Optimization
```bash
# Edit Nginx main configuration
sudo nano /etc/nginx/nginx.conf

# Add optimizations
worker_processes auto;
worker_connections 1024;
keepalive_timeout 65;
gzip on;
gzip_types text/plain text/css application/json application/javascript;
```

### 3. MySQL Optimization
```bash
# Edit MySQL configuration
sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf

# Add optimizations
innodb_buffer_pool_size = 256M
query_cache_type = 1
query_cache_size = 64M
```

## Docker Optimization

### 1. Multi-stage Builds
```dockerfile
# Optimized PHP Dockerfile
FROM php:8.3-fpm as base
RUN docker-php-ext-install mysqli pdo pdo_mysql

FROM base as production
COPY --from=base /usr/local/lib/php/extensions/ /usr/local/lib/php/extensions/
COPY --from=base /usr/local/etc/php/conf.d/ /usr/local/etc/php/conf.d/
```

### 2. Resource Limits
```yaml
# Add resource limits to docker-compose.yml
services:
  php:
    deploy:
      resources:
        limits:
          memory: 512M
        reservations:
          memory: 256M
```

---

# Security Best Practices

## General Security

### 1. Strong Passwords
```bash
# Generate strong passwords
openssl rand -base64 32

# Use different passwords for:
# - MySQL root
# - WordPress database user
# - WordPress admin user
```

### 2. Regular Updates
```bash
# Native installation updates
sudo apt update && sudo apt upgrade
wp core update
wp plugin update --all
wp theme update --all

# Docker updates
docker-compose pull
docker-compose up -d
```

### 3. Firewall Configuration
```bash
# Enable UFW firewall
sudo ufw enable

# Allow SSH, HTTP, HTTPS
sudo ufw allow ssh
sudo ufw allow 80
sudo ufw allow 443

# Check firewall status
sudo ufw status
```

### 4. Backup Strategy
```bash
# Database backup
mysqldump -u root -p wordpress_db > backup_$(date +%Y%m%d).sql

# Docker database backup
docker-compose exec db mysqldump -u root -p wordpress > backup_$(date +%Y%m%d).sql

# Files backup
tar -czf wordpress_files_$(date +%Y%m%d).tar.gz /var/www/wordpress/

# Automated backup script
#!/bin/bash
DATE=$(date +%Y%m%d_%H%M%S)
mysqldump -u root -p$MYSQL_PASSWORD wordpress > backup_${DATE}.sql
tar -czf wordpress_${DATE}.tar.gz /var/www/wordpress/
```

---

## Quick Reference Commands

### Native Installation
```bash
# Service management
sudo systemctl start|stop|restart|status nginx
sudo systemctl start|stop|restart|status php8.3-fpm
sudo systemctl start|stop|restart|status mysql

# Configuration files
/etc/nginx/sites-available/
/etc/php/8.3/fpm/
/etc/mysql/

# WordPress files
/var/www/wordpress/
```

### Docker Installation
```bash
# Container management
docker-compose up -d
docker-compose down
docker-compose restart
docker-compose logs -f

# Execute commands
docker-compose exec php wp --info
docker-compose exec db mysql -u root -p

# Clean up
docker-compose down -v
docker system prune -a
```

This comprehensive guide covers both installation methods with detailed explanations, troubleshooting steps, and optimization techniques. Keep this as your reference for WordPress installations!