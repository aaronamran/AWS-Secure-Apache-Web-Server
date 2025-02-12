# Secure Apache Web Server on AWS Free Tier
This is a writeup of a practical project that involves the following main concepts in order:
1. [AWS Account Setup and Launching an Ubuntu EC2 Instance](https://github.com/aaronamran/Secure-Apache-Web-Server-on-AWS/blob/main/README.md#aws-account-setup-and-launching-an-ubuntu-ec2-instance)
2. [Installing and Configuring Apache](https://github.com/aaronamran/Secure-Apache-Web-Server-on-AWS/blob/main/README.md#installing-and-configuring-apache)
3. [DNS Configuration with Cloudflare (or a Free DNS Provider)](https://github.com/aaronamran/Secure-Apache-Web-Server-on-AWS/blob/main/README.md#dns-configuration-with-cloudflare-or-a-free-dns-provider)
4. [SSL Setup with Certbot and Let’s Encrypt](https://github.com/aaronamran/Secure-Apache-Web-Server-on-AWS/blob/main/README.md#ssl-setup-with-certbot-and-lets-encrypt)
5. [Additional Security Best Practices](https://github.com/aaronamran/Secure-Apache-Web-Server-on-AWS/blob/main/README.md#additional-security-best-practices)
<br>
References: [Tutorial: Install a LAMP server on AL2](https://docs.aws.amazon.com/linux/al2/ug/ec2-lamp-amazon-linux-2.html)



## AWS Account Setup and Launching an Ubuntu EC2 Instance
- In AWS, create a free tier account and complete the identity verification and billing information
- To launch an Ubuntu EC2 Instance, login to the AWS Management Console <br>
  ![image](https://github.com/user-attachments/assets/c7047eb5-d785-4784-a2b6-4b8a788d1219)

- Go to the EC2 Dashboard and click Launch Instance <br>
  ![image](https://github.com/user-attachments/assets/d77593d3-3dba-467e-b7dc-c3518e3dd774)

- Choose an Amazon Machine Image (AMI) and select Ubuntu LTS image (Ubuntu Server 20.04 LTS) <br>
  ![image](https://github.com/user-attachments/assets/2d286e83-4656-411e-aeef-e48a5bf43f7c)

- Choose an Instance Type. Pick the t2.micro as it is eligible under Free Tier <br>
  ![image](https://github.com/user-attachments/assets/3a568b77-4db3-45c9-8f70-a5d8bdffd890)

  
- Create a Key Pair for login usage by clicking `Create new key pair`. Choose a name such as `my-ec2-key`. Keep it secure as it will be used for SSH <br>
  ![image](https://github.com/user-attachments/assets/082e34dc-c725-484b-b70d-e64ebf896961) <br>
  ![image](https://github.com/user-attachments/assets/9b1eb762-b88f-4412-b7cd-888994ac0f93)

- Configure the Security Group in Network settings. Create or select a security group that allows
  - SSH (port 22) – for remote management
  - HTTP (port 80) – for web traffic
  - HTTPS (port 443) – for secure traffic <br>
![image](https://github.com/user-attachments/assets/90e90a2f-6cef-4cdc-a5e6-889eb0a1bbb0)

- For storage configuration, usually the default settings are sufficient <br>
  ![image](https://github.com/user-attachments/assets/1f129163-de0e-40f4-92db-90f24b9c2b84)
    
- To connect to the newly created instance, open a terminal on the local machine. In this case, we will be using WSL (Ubuntu 20.04.6 LTS) on Windows 11
- In WSL, the C drive in Windows can be found in `/mnt`. To check, enter the following
  ```
  ls -l /mnt
  ``` 
  ![image](https://github.com/user-attachments/assets/f13d23fa-ec7b-4608-a272-6f9512643f9b)

- Since the key (`my-ec2-key.pem`) is in the Downloads folder in Windows, we need to navigate to the Windows Downloads folder to find the key.
  ```
  cd /mnt/c/Users/<Windows_username>/Downloads
  ```
  ![image](https://github.com/user-attachments/assets/1972c1ad-5d67-4723-9bed-5a5a7334f31f)

- Now that the key has been located, copy it to the WSL home directory:
  ```
  cp my-ec2-key.pem ~/my-ec2-key.pem
  ```
  Then return to WSL home directory and confirm the key is there <br>
  ![image](https://github.com/user-attachments/assets/8bf1635a-4fae-4936-9325-c460050957f2)

- Set permissions for your key (if needed):
  ```
  chmod 400 ~/my-ec2-key.pem
  ```
- Connect via SSH:
  ```
  ssh -i "my-ec2-key.pem" ec2-user@ec2-13-229-127-129.ap-southeast-1.compute.amazonaws.com
  ```
- The local machine should now be connected to the Ubuntu instance <br>
  ![image](https://github.com/user-attachments/assets/9b232694-28df-4de8-8f56-4122b47774ea)


## Installing and Configuring Apache
- Before updating the system, identify which Linux Distribution is in use. Run the command
  ```
  cat /etc/os-release
  ```
  ![image](https://github.com/user-attachments/assets/8134b92d-0bfb-43af-b0c8-0f7dacfecc77) <br>
  Since Amazon Linux is in use, use `yum` instead of `apt` which is commonly used on Ubuntu or Debian distributions
  
- Update the system:
  ```
  sudo yum update && sudo yum upgrade -y
  ```
  ![image](https://github.com/user-attachments/assets/79e99c39-5f41-4e6d-9997-cead919cb207)

- Install Apache:
  ```
  sudo yum install httpd -y
  ```
  ![image](https://github.com/user-attachments/assets/8c429d74-f728-4d27-8a2a-5cbe13e9ac78)

- Verify Apache is running using each of the commands below, or simply visit `http://<EC2_PUBLIC_IP_ADDRESS>/` in your browser to see the default Apache page):
  ```
  sudo systemctl status httpd
  sudo systemctl enable httpd
  sudo systemctl start httpd
  sudo systemctl status httpd
  ```
  ![image](https://github.com/user-attachments/assets/76e07dae-5855-4e80-9fb3-ea6b3c6e8754)
  <br>
  ![image](https://github.com/user-attachments/assets/a6423e80-19d6-46be-8bc4-1f8061376743)
  <br>
  ![image](https://github.com/user-attachments/assets/8e518aa6-91e3-4f37-ab06-3e77e5d5e810)



- Note that in Amazon Linux, UFW firewall does not exist. It does not matter since Firewall rules were already configured earlier in Network settings' Security Group
  ```
  sudo ufw allow 'Apache Full'
  ```
  ![image](https://github.com/user-attachments/assets/d8073dea-047e-4f74-a18d-f2bcc4dda284)
  

- Now let's prepare the website in the EC2 instance. First, navigate to the `/var/www/html` folder
  ```
  cd /var/www/html
  ```
- Check if git is installed in the EC2 instance. If it is not installed, run the following command:
  ```
  sudo yum install git -y
  ```
  Then confirm it is installed by running
  ```
  git --version
  ```
- Now let's clone a [sample PHP web app repository](https://github.com/aaronamran/sample-php-crud) into the current directory
  ```
  sudo git clone https://github.com/aaronamran/sample-php-crud.git .
  ```
  ![image](https://github.com/user-attachments/assets/d180bd17-f78c-4324-966b-9fdd9ced3d71)

- Install MariaDB server for the website. Run the following command and look for `mariadb105-server`
  ```
  sudo yum search mariadb
  sudo yum install mariadb105-server
  ```
  ![image](https://github.com/user-attachments/assets/d97ba470-9ae5-42d0-a9cd-9cc9288770a2)
  
- Run each of the following commands after the installation is complete:
  ```
  sudo systemctl start mariadb
  sudo systemctl enable mariadb
  sudo systemctl status mariadb
  ```
  ![image](https://github.com/user-attachments/assets/b4df02f1-3c80-4895-a1e2-b70666af9d5b)


## DNS Configuration with Cloudflare (or a Free DNS Provider)
- For the sake of simplicity and free resource, No-IP will be used for a free hostname instead of Cloudflare. After signing up for a free account in No-IP, create a suitable hostname such as `todolist.zapto.org`. The public IP address of the hostname is the public IP address of the EC2 instance <br>
  ![image](https://github.com/user-attachments/assets/143dce9a-8cf6-4754-9554-0369b7a3d523)



## SSL Setup with Certbot and Let’s Encrypt
- Install Certbot and the Apache Plugin:
  ```
  sudo yum install certbot python3-certbot-apache -y
  ```
  ![image](https://github.com/user-attachments/assets/49ba9947-fc35-405d-b6fa-a4abe73c4fb3)


- Run Certbot for Apache:
  ```
  sudo certbot --apache -d todolist.zapto.org
  ```
  Follow the interactive prompts:
  - Provide your email address
  - Agree to the Terms of Service
  - Certbot will ask if you want to redirect HTTP to HTTPS—choose Yes to enforce HTTPS
- Certbot sets up automatic renewal. You can test the renewal process with:
  ```
  sudo certbot renew --dry-run
  ```
- Visit `https://todolist.zapto.org/` in the browser to confirm the SSL certificate is active and that the site redirects from HTTP to HTTPS


## Additional Security Best Practices
- Keep the system updated:
  ```
  sudo apt update && sudo apt upgrade -y
  ```
- Regularly check `/var/log/apache2/` for any unusual activity
- To harden Apache, disable directory listings by ensuring Options -Indexes is set in your Apache configuration. Consider installing security modules like mod_security for added protection
- Change the default SSH port, and use key-based authentication and disable password login
- Ensure the SSL certificates are renewed by checking Certbot logs or setting up email notifications

