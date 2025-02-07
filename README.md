# Secure-Apache-Web-Server-on-AWS
This is a writeup of a practical project that involves the following main concepts:
1. [AWS Account Setup and Launching an Ubuntu EC2 Instance]()
2. [Installing and Configuring Apache]()
3. [DNS Configuration with Cloudflare (or a Free DNS Provider)]()
4. [SSL Setup with Certbot and Let’s Encrypt]()
5. [Additional Security Best Practices]()



### AWS Account Setup and Launching an Ubuntu EC2 Instance
- In AWS, create a free tier account and complete the identity verification and billing information
- To launch an Ubuntu EC2 Instance, login to the AWS Management Console
- Go to the EC2 Dashboard and click Launch Instance
- Choose an Amazon Machine Image (AMI) and select Ubuntu LTS image (Ubuntu Server 20.04 LTS)
- Choose an Instance Type. Pick the t2.micro as it is eligible under Free Tier
- Configure the instance details. For storage, usually the default settings are sufficient
- Configure the Security Group. Create or select a security group that allows
  - SSH (port 22) – for remote management
  - HTTP (port 80) – for web traffic
  - HTTPS (port 443) – for secure traffic
- When prompted, select or create a key pair (download it and keep it secure. It will be used for SSH)
- To connect to the newly created instance, open a terminal on the local machine
- Set permissions for your key (if needed):
  ```
  chmod 400 /path/to/your-key.pem
  ```
- Connect via SSH:
  ```
  ssh -i /path/to/your-key.pem ubuntu@<EC2_PUBLIC_IP_ADDRESS>
  ```
- The local machine should now be connected to the Ubuntu instance



### Installing and Configuring Apache
- Update the system:
  ```
  sudo apt update && sudo apt upgrade -y
  ```
- Install Apache:
  ```
  sudo apt install apache2 -y
  ```
- Verify Apache is running, or simply visit `http://<EC2_PUBLIC_IP_ADDRESS>/` in your browser to see the default Apache page):
  ```
  sudo systemctl status apache2
  ```
- Adjust UFW Firewall Settings to allow Apache through UFW:
  ```
  sudo ufw allow 'Apache Full'
  sudo ufw enable
  sudo ufw status
  ```
- To configure Virtual Hosts for the website, create a directory for the website:
  ```
  sudo mkdir -p /var/www/yourdomain.com/public_html
  sudo chown -R $USER:$USER /var/www/yourdomain.com/public_html
  sudo chmod -R 755 /var/www
  ```
- Create a sample `index.html` page:
  ```
  nano /var/www/yourdomain.com/public_html/index.html
  ```
  and paste the following HTML, then save and exit:
  ```
  <!DOCTYPE html>
  <html>
  <head>
      <title>Welcome to YourDomain!</title>
  </head>
  <body>
      <h1>Success! Your Apache virtual host is working!</h1>
  </body>
  </html>
  ```
- Create a new Virtual Host configuration file:
  ```
  sudo nano /etc/apache2/sites-available/yourdomain.com.conf
  ```
  Paste the following configuration:
  ```
  <VirtualHost *:80>
    ServerAdmin admin@yourdomain.com
    ServerName yourdomain.com
    ServerAlias www.yourdomain.com
    DocumentRoot /var/www/yourdomain.com/public_html
    
    ErrorLog ${APACHE_LOG_DIR}/yourdomain_error.log
    CustomLog ${APACHE_LOG_DIR}/yourdomain_access.log combined
  </VirtualHost>
  ```
- Enable the new Virtual Host:
  ```
  sudo a2ensite yourdomain.com.conf
  ```
- Disable the Default Site (optional but recommended to avoid conflicts):
  ```
  sudo a2dissite 000-default.conf
  ```
- Test the Apache configuration and reload:
  ```
  sudo apache2ctl configtest
  sudo systemctl reload apache2
  ```
- Apache logs are typically stored in `/var/log/apache2/`
- You can adjust logging levels and rotation policies via the main Apache configuration files and logrotate (`/etc/logrotate.d/apache2`)
- For performance, consider tuning settings like `KeepAlive`, `MaxKeepAliveRequests`, etc., in `/etc/apache2/apache2.conf` if necessary





### DNS Configuration with Cloudflare (or a Free DNS Provider)
- 




### SSL Setup with Certbot and Let’s Encrypt
- Install Certbot and the Apache Plugin:
  ```
  sudo apt install certbot python3-certbot-apache -y
  ```
- Run Certbot for Apache:
  ```
  sudo certbot --apache -d yourdomain.com -d www.yourdomain.com
  ```
  Follow the interactive prompts:
  - Provide your email address
  - Agree to the Terms of Service
  - Certbot will ask if you want to redirect HTTP to HTTPS—choose Yes to enforce HTTPS
- Certbot sets up automatic renewal. You can test the renewal process with:
  ```
  sudo certbot renew --dry-run
  ```
- Visit `https://yourdomain.com/` in the browser to confirm the SSL certificate is active and that the site redirects from HTTP to HTTPS


### Additional Security Best Practices
- Keep the system updated:
  ```
  sudo apt update && sudo apt upgrade -y
  ```
- Regularly check `/var/log/apache2/` for any unusual activity
- To harden Apache, disable directory listings by ensuring Options -Indexes is set in your Apache configuration. Consider installing security modules like mod_security for added protection
- Change the default SSH port, and use key-based authentication and disable password login
- Ensure the SSL certificates are renewed by checking Certbot logs or setting up email notifications

