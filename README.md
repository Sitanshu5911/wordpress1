## Welcome to the documentation for hosting and deploying of WordPress website on VPS Server.

## This README.md file provides a step-by-step guide for setting up an automated deployment process for a WordPress website using the Nginx web server and the LEMP stack (Linux, Nginx, 
   MySQL, PHP). The deployment process outlined here adheres to security best practices and aims to ensure optimal performance for your WordPress site.


## Prerequisites
  * Free Tier AWS account for launching an EC2 Instance. 
  * A Github account.
	* Free Domain from "https://www.noip.com/"
 	* MobaXterm for connecting an AWS Instance through SSH securely.

## Let's Create a Wordpress Website with VPS Server.

## Launching an AWS EC2 Instance:

  1. Firstly, you need an AWS account to deploy your WordPress website.
  2. Sign in to the AWS Management Console, open the Amazon EC2 console, choose 'Launch Instance,' and type a name for your instance.
  3. Select the application and OS Images (Amazon Machine Image) choose Ubuntu 20.4 as the OS, and select the instance type, i.e., t2.micro (free tier). Create a key pair to connect to your 
     EC2 instance from your local machine. Download and securely store the key pair; it will be used for connecting to the EC2 instance later.
  4. Here you have to create a new "Security Group" in which you will specify which Ports will be Open for EC2 instance. You have to add HTTP, HTTPS, and SSH Port for your EC2 Instance. HTTP       and HTTPS to allow web request from anywhere in the world to your instance. And SSH to Connect to your EC2 Instance from Local Machine. Allocate storage for your EC2 instance, e.g., 
     20GB. (Free Tier is eligible for up to 30 GB of storage.)
  5. After all these steps, click on 'Launch Instance.' Then you have to open MobaXterm, choose 'SSH' as the session type, Enter your EC2 instance public IP address in the "Remote host"            field. Set the "username" field to "ubuntu". Under the "Advanced SSH settings" tab, go to the "Use private key" field and select your private key (.pem) file. You should now be logged 
     into your EC2 instance using SSH.

## Configuring Server and Deploying WordPress on Ubuntu 20.4:

  * Run these command first to update your server and installed packages.

     sudo apt update 
     sudo apt upgrade 
 
  * Now you need install NGINX. because we will use this nginx server to host our websites. Install the NGINX using following command.

     sudo apt install nginx

  * Once you have installed nginx, then you can confirm it by checking their status. whether its running or not.

     sudo systemctl status nginx

  * Enabling UFW wall and port:
    80 and 443 port should be enabled for nginx. by default, these will be disabled. But you can enable these by adding just one rule ‘Nginx Full’. Nginx full has rules for both the port.
    But before that you have to enable your UFW wall.

     sudo ufw enable
     sudo ufw allow 'Nginx Full'
     sudo ufw allow 22

  * Once you’ve enabled all the required ports then you can check it by using this command.

     sudo ufw status

  * Installing MySQL Database and Configuring:
    Now we will install mysql and configure one database for our wordpress site. You can install mysql by using following command.

     sudo apt install mysql-server

  * Once you’ve installed your mysql then you can check it by using following command, which will show you whether your database is running or not.

     sudo systemctl status mysql

  * Now it's time to create our database, but before doing so, log into your MySQL database using the following command.

     sudo mysql

  * And Once you have logged inside MySQL cli then your terminal will start with mysql >. Now type the following command to CREATE your new DATABASE.

      CREATE DATABASE server_db;

  * Now create new mysql user that we will use for our wordpress. And we will assign access of "server_db" to this user. To create new user type following command

     CREATE USER 'sitanshu'@'localhost' IDENTIFIED BY 'nexus.myftp.org@5911';

  * Now GRANT access of "server_db" to "sitanshu" by following command.

     GRANT ALL PRIVILEGES ON server_db . * TO 'sitanshu'@'localhost';

  * And then type following commands.

     FLUSH PRIVILEGES;

     exit;

  * That’s it. we will use this database later while we would configure our wordpress website.

## Installing PHP.

  * Now you need to install PHP. You can install PHP by using following command.

     sudo apt install php7.4-cli php7.4-fpm php7.4-mysql php7.4-json php7.4-opcache php7.4-mbstring php7.4-xml php7.4-gd php7.4-curl

  * Once you’ve installed PHP then you can confirm it by command:

     php -v

## Downloading and Configuring WordPress.

  * But before that create a folder with following command and mention your Domain name.

     sudo mkdir /var/www/html/nexus.myftp.org

  * Like my domain name is nexus.myftp.org then it would be like "mkdir /var/www/html/nexus.myftp.org". So, once you’ve created above folder then you can get inside this folder by using 
    following command.

     cd /var/www/html/nexus.myftp.org

  * Once you’ve inside "nexus.myftp.org" folder, then type following command to download wordpress.

     sudo wget https://wordpress.org/latest.tar.gz
  
  * Once you’ve downloaded wordpress then you have to unzip it by using following command.

     sudo tar xf latest.tar.gz
 
  * Above command will extract wordpress files inside wordpress folder. but we will move these wordpress file inisde our nexus.myftp.org folder.

     sudo mv wordpress/* ./
  
  * Now you can delete empty wordrpess folder and compressed file using following command, which we will not need now.

     sudo rm -r wordpress
    
     sudo rm -r latest.tar.gz

  * And now type following command to create "wp-config.php" with "wp-config-sample.php" file. Here we will give information of our database that we created.

     sudo mv wp-config-sample.php wp-config.php

  * And then type following command to edit "wp-config.php" file.

     sudo vim wp-config.php

  * Here you will have to update database information with your database like the below

     define( 'DB_NAME', 'test_db' );
     /** MySQL database username */
     define( 'DB_USER', 'sitanshu' );
     /** MySQL database password */
     define( 'DB_PASSWORD', 'nexus.myftp.org@5911' );

  * Once it’s update with your database credentials then you can save it Esc+wq-Enter.
    Now you have to give access of www-data to your wordpress directory by using following command.

     sudo chown -R www-data: /var/www/html/nexus.myftp.org

## Configuring NGINX for WordPress:
 
  * Now it’s time to configure NGINX for our WordPress website. but first, you have to go inside sites-available folder by using following command.

     cd /etc/nginx/sites-available/

  * Once you’re inside ‘sites-available’ directory. then you have to create configuration file for wordpress by using following command.

     sudo vim nexus.myftp.org.conf

  * Above command will open vim editor, you have to paste following configuration here.

     server {
     listen 80;
     listen [::]:80;
     root /var/www/html/nexus.myftp.org;
     index index.php index.html index.htm index.nginx-debian.html;
     server_name nexus.myftp.org www.nexus.myftp.org;
     location / {
          try_files $uri $uri/ /index.php$is_args$args;
     }
     location ~ \.php$ {
             include snippets/fastcgi-php.conf;
             fastcgi_pass unix:/run/php/php7.4-fpm.sock;
     }
} 


   * In above code, mention your domain name. Once it’s done then just save it using Esc+wq-Enter. And now you have to create symbolic link to sites-enabled folder using following command. 
    
      sudo ln -s /etc/nginx/sites-available/nexus.myftp.org.conf /etc/nginx/sites-enabled/

   * The above command will add same conf file inside sites-enabled folder. Once its done then you can check it by using command.

      sudo nginx -t

   * If its says test pass then you can restart your server by following command.

      sudo systemctl restart nginx

   * Above command will restart your server after that visit you website http://nexus.myftp.org. You’ve successfully setup and installed WordPress using NGINX.

## Installing Let's Encrypt to Secure the HTTP Communication.

   * The below command updates the list of available packages on your system, ensuring you have the latest information about available software.

      sudo apt-get update

   * After that the following command installs the "software-properties-common" package, which provides an abstraction of the used package management tools.

      sudo apt-get install software-properties-common

   * Then you have to add the Certbot repository to your system. Certbot is a tool for managing Let's Encrypt SSL certificates.

      sudo add-apt-repository ppa:certbot/certbot

   * The below command updates the package lists again, but now it includes the newly added Certbot repository.

      sudo apt-get update

   * Install Certbot for Nginx. For that you will need this command. The python-certbot-nginx package includes the Nginx plugin for Certbot, allowing it to automatically configure SSL 
     certificates for Nginx.

      sudo apt-get install python-certbot-nginx

   *  To run Certbot to Obtain and Install SSL Certificate type the following command. Certbot will automatically configure Nginx to use the obtained SSL certificate for the specified domain.

      sudo certbot --nginx -d nexus.myftp.org

   * After running these commands, Certbot will guide you through an interactive process to set up the SSL certificate for your domain. It will ask you to provide an email address for 
     renewal reminders and other important notifications. Additionally, it will prompt you to agree to the terms of service. Once the process is complete, your Nginx server should be 
     configured to use the Let's Encrypt SSL certificate, securing the communication between your server and clients.  


 

    

     
     
