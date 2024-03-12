# Project-LEMP-stack

Part 2: Setting Up LEMP Stack:

LEMP stands for: Linux, NGINX web server, MySQL Database, and PHP

What you will need?

AWS account
Local machine CLI
water and patience

1.Connect to EC2 Instance:

Step 1: Create an AWS EC2 instance

The first thing we are going to do is to log into in the AWS Management Console and create an EC2 instance
Choose a name you like and select the Ubuntu image for the instance.
Select the “Ubuntu Server 22.04 LTS HVM, Volume Type”.
You will see it under the free tier.
If you have a key pair already created select it from the drop down menu, 
if you don’t have one, then create one and download it to your local machine
Select the default EBS volume(8 gbs)

Under the “Network settings” we will create a new security group and select “allow SSH traffic from” and “Allow HTTP from the internet”.
This will allow our instance to be able to send and receive traffic, thus be able to connect to the internet.


Step 2: via CLI

i. ON YOUR COMMAND PROMPT Connect to Instance:
Establish SSH connection to the EC2 instance.--

cd "path to your key file" for eg- cd C:\Users\NAME\linux key  (remember to put the cd command before the path)
then, click enter and copy and paste your AWS CONSOLE SSH (its found when you click on your instance name and click on connect, 
under the 'connect to instance' page click on the SSH client and you will find your instance public DNS that
looks like this ssh -i /path/to/your/private-key.pem ec2-user@your-ec2-public-ip
                   
                                OR

ii. TO CONNECT to your instance VIA YOUR AWS LINUX (UBUNTU) - 
ON YOUR AWS CONSOLE click on your instance name and cick on 'connect',
on the connect to instance page under 'Connect using EC2 Instance Connect, get to the end and click on connect.


update to the latest version by running the command - 

sudo apt update


2.Install Components:

Install Nginx, MySQL, and PHP packages using package manager commands--

 
a. Installing Nginx --

sudo apt install nginx


check status of the NGINX web server:--

sudo systemctl status nginx

Check to see if we can receive traffic to our instance via the CLI.
If everything was setup properly you should see the default NGINX webpage

Run the following command:--
curl http://localhost:80

Check to see if we can receive traffic to our instance via the WEB BROWSER.
Copy the IP address of the instance and paste that into your browser window using the ,
and hit the ENTER/RETURN key on your keyboard.--

http://Public-IP-Address:80

 voila 

b. Installing MySQL
Our web server is up and running so now it is time to focus on the “M” part of the LEMP stack. The “M” stands for MySQL.
MySQL is a popular relational database management system used for PHP environments,
and we will deploying that as part of this project!

Run the following command to install MySQL:

When prompted enter “Y” and hit enter/return key--

sudo apt install mysql-server

After the successful installation of the MySQL database, we are going to log into the MySQL console.

Run the command:
sudo mysql

We will run a security script directly from the MySQL console, and this script will give us the permission 
to set a password for our example root user to be able to have access to the database.

Run the following script from the MySQL console: “mysql>” --

ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'PassWord.1';

Exit the MySQL console --

mysql> exit

Next, we are going to run a few scripts to validate the password configuration

Run the command, enter “Y” when prompted--

sudo mysql_secure_installation


When prompted with the validation policy, enter the value 1. The value 1 represents the medium value that is being applied to the root password.
You will be met with a few confirmation prompts, enter “Y” for all.

Now that our root user password has been SET. We will reconnect to MySQL to save the changes and exit.
MySQL is now officially installed and secured.


c. Installing PHP

So far our we have successfully deployed a linux instance, NGINX web server, and MySQL relational database. 
t is time to deploy the last part of the LEMP stack, and that is PHP.
For this project we will setup the PHP to be able to communicate with the NGINX web server so that it can host our custom website.

PHP- “A popular general-purpose scripting language that is especially suited to web development.
Fast, flexible and pragmatic, PHP powers everything from your blog to the most popular websites in the world”.- php.net

Run the following command to install PHP:

These two command will install php, and allow it to work with NGINX.
Enter “Y” when prompted and hit the ENTER/RETURN key on your keyboard.--

sudo apt install php-fpm php-mysql


3. Configuring NGINX to work with PHP

By default NGINX is setup to only run one server block.
Next, you'll need to configure Nginx to process PHP files.
By default, Nginx doesn't handle PHP files directly.
You'll need to configure it to pass PHP requests to the PHP FastCGI Process Manager (PHP-FPM).
We are going to change the default settings so that we can enable the server block to allow for multiple server configurations.
We will create a new directory to host our PHP based website.

Run the following command to create the root directory: We will name the root directory “/var/www/projectLEMP” --

sudo mkdir /var/www/projectLEMP


We will change ownership of the directory and assign it to our root user($USER) --

sudo chown -R $USER:$USER /var/www/projectLEMP


Using the CLI editor, open a new configuration file in Nginx’s sites-available directory. Use nano.--

sudo nano /etc/nginx/sites-available/projectLEMP



Copy and paste the script below and save and exit the file.--

#/etc/nginx/sites-available/projectLEMP

server {
    listen 80;
    server_name projectLEMP www.projectLEMP;
    root /var/www/projectLEMP;

    index index.html index.htm index.php;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
     }

    location ~ /\.ht {
        deny all;
    }

}
“listen”= Tells NGINX what port to listen on. Port 80 is the port for HTTP

“root”= Where the root document will be stored

“index”= refers to the index.html, index.htm, as well as the index.php files for the application.


The following script will activate the configuration file from the NGINX’s sites-enabled directory:--

sudo ln -s /etc/nginx/sites-available/projectLEMP /etc/nginx/sites-enabled/

We want NGINX web server to serve the configuration file as opposed to the default NGINX webpage when the page is reloaded.
The following script will check the file for any errors..--

sudo nginx -t

If there are any syntax errors make sure to go back and check the configuration.

We must disable the default NGINX host that is listening on port 80.
Afterwards, we will reload NGINX to save the changes.

Run the following command to disable it.--

sudo unlink /etc/nginx/sites-enabled/default
( removes nginx as the default host )

sudo systemctl reload nginx
( reloads nginx )


4.Test LEMP stack - Create and access a PHP info file to confirm successful stack setup.

Our custom website is now up and running, but we have to create an index.html 
so that we can test out the server block and make sure things are working properly.--

sudo echo 'Hello LEMP from hostname' $(curl -s http://169.254.169.254/latest/meta-data/public-hostname) 'with public IP'
$(curl -s http://169.254.169.254/latest/meta-data/public-ipv4) > /var/www/projectLEMP/index.html

The last leg of the LEMP stack is now complete! In next step we will create a PHP script to test with NGINX!


Testing PHP with NGINX

We are going to test and validate that NGINX can handle PHP files and serve them on our website.

Before we can run a test, a test PHP file will need to be created in our root directory using the text editor.
Run the following command.--

sudo nano /var/www/projectLEMP/info.php

Paste the following text in the new file.
This PHP code will give us information about our server.
Save and exit the file.--

<?php
phpinfo();

With the PHP file setup we can now access the webpage in the web browser.
You will use the public IP of the instance or the domain name you setup in the NGINX configuration file,
followed by/info.php.

In your browser type the following and hit ENTER/RETURN key.--

http://`server_domain_or_IP`/info.php

http://44.203.190.129/info.php
( example)

You should see the webpage, Detailed information about your server will be shown. Scroll down and take a look!

Great! Good job the PHP file was able to be served up by the PHP server. Everything looks good! Now that you have taken a good look at the server information, it is time to remove the file. The PHP contains sensitive information about your PHP environment and Ubuntu server(instance).

Run the following command to remove the PHP file:
sudo rm /var/www/your_domain/info.php



NOTES

sudo apt install nginx 

sudo apt install mysql-server

sudo apt install php-fpm php-mysql

sudo mkdir /var/www/projectLEMP

sudo chown -R $USER:$USER /var/www/projectLEMP

sudo nano /etc/nginx/sites-available/projectLEMP

sudo ln -s /etc/nginx/sites-available/projectLEMP /etc/nginx/sites-enabled/

sudo nginx -t

sudo unlink /etc/nginx/sites-enabled/default

sudo systemctl reload nginx

sudo echo 'Hello LEMP from hostname' $(curl -s http://169.254.169.254/latest/meta-data/public-hostname) 'with public IP'
$(curl -s http://169.254.169.254/latest/meta-data/public-ipv4) > /var/www/projectLEMP/index.html


TROUBLESHOOTING ERROR

a. How to Uninstall MYSQL From Ubuntu?
i encountered a password error (i forgot to add a password)
so i got locked out from using mysql so i had to uninstall mysql from my ubuntu.(i reinstalled it after the uninstall to correct the password part)

Before uninstallation, you should back up your files and configurations i.e if you have any that is important.
You can do it manually if you are familiar with MYSQL or use the mysqldump utility, which will secure your data.

Step 1: Stop MYSQL Services
Before removing any application with background services, it is necessary to stop all its services before moving forward with the uninstallation process. To stop MySQL and its services, open your terminal window and run this command:

sudo systemctl stop mysql

stop mysql

This command will stop the services of MYSQL, and now you can safely remove the MYSQL database from your system.


Step 2: Remove the MySQL Essentials
Run this command to remove the MYSQL server, client, common, server-core, and client-core packages from your system:

sudo apt purge mysql-server mysql-client mysql-common mysql-server-core-* mysql-client-core-*
uninstall mysql

The MYSQL files are now removed from your system.

Step 3: Remove the MySQL Associated Files
Now the next step is to remove the remaining files from the folders:

sudo rm -rf /etc/mysql /var/lib/mysql /var/log/mysql
remove files

Step 4: Remove Additional Packages
This command will automatically remove the files from the folders. After this run, these commands:

sudo apt autoremove
autoremove

It is recommended to clean the disk space on Ubuntu as well, using the command:

sudo apt autoclean
autoclean

All the dependencies and additional files will be removed by auto-remove and auto-clean commands.

Step 5: Remove the Configurations
After this run, this command to remove the configuration, as in some cases configuration is not removed or some files are remaining on the system:

sudo apt remove dbconfig-mysql
remove configurations

Once all the above steps are completed, MySQL will be removed from your system completely.


b. sudo ss -tuln | grep :80

Stop the conflicting process: Once you've identified the process using port 80, you can stop it using the appropriate command. For example, if it's Apache, you can use:

sudo systemctl stop apache2
Replace apache2 with the actual service name if it's different.

Start Nginx: After stopping the conflicting process, you can start Nginx again:

sudo systemctl start nginx
Check Nginx status: Verify that Nginx is running without any errors:

sudo systemctl status nginx
If Nginx is running without errors, you should see an Active: active (running) status.

By following these steps, you should be able to resolve the issue.
