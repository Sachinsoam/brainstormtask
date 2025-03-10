# **Automated Deployment of WordPress Using LEMP Stack and GitHub Actions**

## **Introduction**
This document outlines the step-by-step implementation of an automated deployment process for a WordPress website using **Nginx, MySQL, PHP (LEMP stack)**, and **GitHub Actions** as the CI/CD tool. The setup follows industry best practices for security and performance optimization.

## **1. Server Provisioning**
### **Server Provisioning on AWs**
- Provision a **Virtual Server ** with a major cloud provider (**AWS, Azure, GCP, or DigitalOcean**).
- Use **Ubuntu 22.04** as the operating system.
- Secure the server by performing the following actions:
  - **Update system packages:**
    ```bash
    sudo apt update && sudo apt upgrade -y
    ```
  - **Configure the firewall :**
  - Only Allow https traffic in Security Group of Ec2
  - Create a new user for deployment and disable root SSH login.

## **2. Installing and Configuring LEMP Stack**
### **Install Nginx**
```bash
sudo apt install nginx -y
```
- Verify installation:
  ```bash
  sudo systemctl status nginx
  ```

### **Install MySQL**
```bash
sudo apt install mysql-server -y
```
- Secure MySQL installation:
  ```bash
  sudo mysql_secure_installation
  ```
- Create a database for WordPress:
  ```sql
  CREATE DATABASE wordpress;
  CREATE USER 'wp_user'@'localhost' IDENTIFIED BY '<user_password>';
  GRANT ALL PRIVILEGES ON wordpress.* TO 'wp_user'@'localhost';
  FLUSH PRIVILEGES;
  ```

### **Install PHP**
```bash
sudo apt install php-fpm php-mysql php-cli php-curl php-mbstring php-xml php-zip -y
```
- Verify PHP installation:
  ```bash
  php -v
  ```

## **3. WordPress Installation & Nginx Configuration & Setup of Github with Wordpress Latest Code**
### **Download and Configure WordPress**
```bash
cd /var/www
sudo wget https://wordpress.org/latest.tar.gz
sudo tar -xvzf latest.tar.gz
sudo mv wordpress /var/www/html/wordpress
sudo chown -R www-data:www-data /var/www/html
sudo chmod -R 755 /var/www/html
```
### **Configure Nginx for WordPress**
- Create a new Nginx configuration file:
  ```bash
  sudo nano /etc/nginx/sites-available/wordpress
  ```
- Add the following configuration:
  ```nginx
  server {
      listen 80;
      server_name <your_domain>;
      root /var/www/html/wordpress;
      index index.php index.html index.htm;

      location / {
          try_files $uri $uri/ /index.php?$args;
      }

      location ~ \.php$ {
          include snippets/fastcgi-php.conf;
          fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
          fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
          include fastcgi_params;
      }
  }
  ```
- Enable the configuration:
  ```bash
  sudo ln -s /etc/nginx/sites-available/wordpress /etc/nginx/sites-enabled/
  sudo systemctl restart nginx
  ```

## **4. Securing the Website with SSL (Let's Encrypt)**
```bash
sudo apt install certbot python3-certbot-nginx -y
sudo certbot --nginx -d <your_domain>
```
- Set up auto-renewal:
  ```bash
  sudo certbot renew --dry-run
  ```

## **5. GitHub Actions Setup for CI/CD**
### **Create a GitHub Repository**
- Push WordPress files (excluding `wp-config.php`) to a **GitHub repository**.

### **Set Up GitHub Actions Workflow**
- Create `.github/workflows/deploy.yml` in the repository:
  ```yaml
  name: Deploy WordPress to VPS

  on:
    push:
      branches:
        - main

  jobs:
    deploy:
      runs-on: ubuntu-latest

      steps:
        - name: Checkout repository
          uses: actions/checkout@v3

        - name: Deploy to Server
          uses: appleboy/ssh-action@v0.1.8
          with:
            host: ${{ secrets.SERVER_IP }}
            username: ${{ secrets.SERVER_USER }}
            key: ${{ secrets.SSH_PRIVATE_KEY }}
            script: |
              cd /var/www/html/wordpress
              git pull origin main
              sudo systemctl restart nginx  # optional
  ```

### **Set Up GitHub Secrets**
Ensure the following **Secrets** are added in GitHub Actions:
- `SERVER_IP`: The public IP of your EC2.
- `SERVER_USER`: The username to SSH into the server.
- `SSH_PRIVATE_KEY`: The private SSH key for authentication.

---

## **Conclusion**
This setup ensures a secure, automated, and scalable deployment of a WordPress website using **Nginx, MySQL, and PHP (LEMP stack)** with **GitHub Actions** for CI/CD. The implementation follows security best practices and performance optimizations for a production-ready environment.

