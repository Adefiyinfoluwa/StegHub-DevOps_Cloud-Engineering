## WEB STACK IMPLEMENTATION (LAMP STACK) IN AWS

### Introduction:

### What is a LAMP stack?

A LAMP stack comprises four distinct software components essential for developing websites and web applications. LAMP stands for Linux (the operating system), Apache (the web server), MySQL (the database server), and PHP (the programming language). Each of these elements is open source, indicating they're supported by a community and accessible for all to utilize at no cost.

The LAMP architecture is like a team of four friends working together to build a website. Each friend has a special job to do, and they all work together to make the website run smoothly.

Linux: Linux is like the leader of the group. Linux is an open-source operating system that you can install and configure to meet different application requirements.  

 Apache: an open-source web server, is like the storyteller. It's the second layer of the LAMP stack, responsible for sharing the website's files with people's web browsers. The Apache module stores website files and exchanges information with browsers using HTTP, an internet protocol for transferring website information in plain text.

MySQL: an open-source relational database management system, is like the librarian. It's the third layer of the LAMP stack, serving as a database where the website stores crucial information such as user accounts and product details. The LAMP model utilizes MySQL for storing, managing, and querying data in relational databases.

PHP: stands for PHP: Hypertext Preprocessor, is like the magician. It's the fourth and final layer of the LAMP stack, a scripting language that enables websites to run dynamic processes. Web developers embed the PHP programming language in HTML to display real-time or updated information on websites.

This documentation outlines the setup, configuration, and usage of the LAMP stack.

## Step 0: Prerequisites

### In order to complete this project, an AWS account and Ubuntu Server OS were utilized.

#### 1. Launch an EC2 Instance with t2.micro Ubuntu 24.04 LTS (HVM) in any region of your choose.

This document details the steps to launch an Amazon EC2 instance using the AWS console. The instance will be of type t2.micro, running Ubuntu 24.04 LTS (HVM), and deployed in my desired region.


![Lunch Instance](/LAMP%20STACK/img/LampServer%20.png)
![Lunch Instance](/LAMP%20STACK/img/EC2%20Running.png)

2. Created SSH key pair named "Lamp_server" to access the instance on port 22

![Lunch Instance](/LAMP%20STACK/img/keypair.png)

The security group was configured with the following inbound rules:

- Allow traffic on port 80 (HTTP) with source from anywhere on the internet.
- Allow traffic on port 443 (HTTPS) with source from anywhere on the internet.
- Allow traffic on port 22 (SSH) with source from any IP address. This is opened by default.

![Security Group Rules](/LAMP%20STACK/img/EC2%20Securitygroup.png)

__4.__ The default VPC and Subnet was used for the networking configuration.

![Network](/LAMP%20STACK/img/network.png)

__5.__ The private ssh key  downloaded was located, permission was changed for the private key file 
```
sudo chmod 0400 lamp_server.pem
```
- and then used to connect to the instance by running
```
ssh -i lamp_server.pem ubuntu@3.82.206.119
```

![Connect to instance](/LAMP%20STACK/img/ubuntu.png)


## Step 1 - Install Apache and Update the Firewall.

__1.__ __Update and upgrade list of packages in package manager.__
```
sudo apt update
sudo apt upgrade -y
```
![Update Packages](/LAMP%20STACK/img/upgrade.png)
![Upgrade Packages](/LAMP%20STACK/img/upgrade2.png)

__2.__ __Run apache2 package installation__
```
sudo apt install apache2 
```
![Install Apache](/LAMP%20STACK/img/installApache.png)

__3.__ __Enable and verify that apache is running on as a service on the OS.__
```
sudo systemctl enable apache2
sudo systemctl status apache2
```
If it green and running, then apache2 is correctly installed. And I had just launched my first Web Server in the Clouds.

![Apache Status](/LAMP%20STACK/img/verify&enableApache.png)

__4.__ __The server is running and can be accessed locally in the ubuntu shell by running the command below:__

```
curl http://localhost:80
OR
curl http://127.0.0.1:80
```

![Local URL](/LAMP%20STACK/img/default_page_url.png)

__5.__ __Test with the public IP address if the Apache HTTP server can respond to request from the internet using the url on a browser.__
```
http://3.82.206.119:80
```

![Apache Default Page](/LAMP%20STACK/img/Apache_page.png)
This indicates that the web server is properly installed and accessible through the firewall.

__6.__ __Another way to retrieve the public ip address,other than check the aws console__
```
curl -s http://169.254.169.254/latest/meta-data/public-ipv4
```
After running the command above, there was an error __401 - Unauthorized__ output.

![Unauthorized Error-401](/LAMP%20STACK/img/unauthorized%20error_curl.png)

When troubleshooting this error, the following navigation was performed from the EC2 instance page on the AWS console:

- Actions > Instance Settings > Modify instance metadata options.
- Then change the __IMDSv2__ from __Required__ to __Optional__.

![imds option](/LAMP%20STACK/img/imd_option.png)

The command was run again, this time there was no error with the public IP address displayed.

```
curl -s http://169.254.169.254/latest/meta-data/public-ipv4
```

![public_ip_curl](/LAMP%20STACK/img/public_ip_curl.png)

## Step 2 - Install MySQL

__1.__ __Install a relational database (RDB)__

MySQL was installed in this project. It is a popular relational database management system used within PHP environments.
```
sudo apt install mysql-server
```

![Install Msql](/LAMP%20STACK/img/mysql_server.png)

When prompted, install was confirmed by typing y and then Enter.

__2.__ __Enable and verify that mysql is running with the commands below__
```
sudo systemctl enable --now mysql
sudo systemctl status mysql
```

![MySQL Status](/LAMP%20STACK/img/mysql_status.png)

__3.__ __Log in to mysql console__
```
sudo mysql
```

![MySQLlogin](/LAMP%20STACK/img/MySQL_login.png)

This connects to the MySQL server as the administrative database user __root__ infered by the use of __sudo__ when running the command.

__4.__ __Set a password for root user using mysql_native_password as default authentication method.__

Here, the user's password was defined as "Admin123$"
```
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'Admin123$';
```

![alteruser](/LAMP%20STACK/img/alteruser.png)

Exit the MySQL shell
```
exit
```

__5.__ __Run an Interactive script to secure MySQL__

The security script comes pre-installed with mysql. This script removes some insecure settings and lock down access to the database system.
```
sudo mysql_secure_installation
```

![mysql_secure_installation](/LAMP%20STACK/img/mysql_secure_installation.png)

Regardless of whether the VALIDATION PASSWORD PLUGIN is set up, the server will ask to select and confirm a password for MySQL root user.

__6.__ __Following the password change for the root user, log in to the MySQL console..__

A command prompt for password was noticed after running the command below.
```
sudo mysql -p
```
![mysql_2login](/LAMP%20STACK/img/mysql_2login.png)

Exit MySQL shell
```
exit
```

## Step 3 - Install PHP

__1.__ __Install php__

Apache is installed to serve the content and MySQL is installed to store and manage data.
PHP is the component of the set up that processes code to display dynamic content to the end user.

The following were installed:
- php package
- php-mysql, a PHP module that allows PHP to communicate with MySQL-based databases.
- libapache2-mod-php, to enable Apache to handle PHP files.
```
sudo apt install php libapache2-mod-php php-mysql
```

![mysql_2login](/LAMP%20STACK/img/installApache.png)

Confirm the PHP version
```
php -v
```

![Confirm php version](/LAMP%20STACK/img/phpconfirmation.png)

At this juncture, the LAMP stack is fully installed and operational.

To validate the setup with a PHP script, it's advisable to establish a well-configured Apache Virtual Host to manage the website's files and directories. A Virtual Host enables hosting multiple websites on a single machine seamlessly, without impacting website users.

## Step 4 - Create a virtual host for the website using Apache

__1.__ __The default directory serving the apache default page is /var/www/html. Create your document directory next to the default one.__

Created the directory for projectlamp using "mkdir" command
```
sudo mkdir /var/www/projectlamp
```

__Assign the directory ownership with $USER environment variable which references the current system user.__
```
sudo chown -R $USER:$USER /var/www/projectlamp
```

![Project Root Directory](/LAMP%20STACK/img/create_root_directory.png)

__2.__ __Create and open a new configuration file in apache’s “sites-available” directory using vim.__
```
sudo vim /etc/apache2/sites-available/projectlamp.conf
```

Past in the bare-bones configuration below:
```
<VirtualHost *:80>
  ServerName projectlamp
  ServerAlias www.projectlamp
  ServerAdmin webmaster@localhost
  DocumentRoot /var/www/projectlamp
  ErrorLog ${APACHE_LOG_DIR}/error.log
  CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
![Virtual Host](/LAMP%20STACK/img/virtualhost.png)

__3.__ __Show the new file in sites-available__
```
sudo ls /etc/apache2/sites-available
```
```
Output:
000-default.conf default-ssl.conf projectlamp.conf
```

![Projectlamp.conf](/LAMP%20STACK/img/projectlamp.png)

With the VirtualHost configuration, Apache will serve projectlamp using /var/www/projectlamp as its web root directory.

__4.__ __Enable the new virtual host__
```
sudo a2ensite projectlamp
```
![enablerootdir](/LAMP%20STACK/img/enable_root_directory.png)

__5.__ __Disable apache’s default website.__

This is because Apache’s default configuration will overwrite the virtual host if not disabled. This is required if a custom domain is not being used.
```
sudo a2dissite 000-default
```
![disenablerootdir](/LAMP%20STACK/img/disable_root_directory.png)

__6.__ __Ensure the configuration does not contain syntax error__

The command below was used:
```
sudo apache2ctl configtest
```
![Check syntax error](/LAMP%20STACK/img/root_dir_syntax.png)

__7.__ __Reload apache for changes to take effect.__
```
sudo systemctl reload apache2
```
![Relaod Apache](/LAMP%20STACK/img/reload_aoache.png)

__8.__ __The new website is live now, but the web root at `/var/www/projectlamp` remains empty. To verify that the virtual host functions correctly, create an `index.html` file in this directory and test accordingly.__
```
sudo echo 'Hello LAMP from hostname' $(curl -s http://169.254.169.254/latest/meta-data/public-hostname) 'with public IP' $(curl -s http://169.254.169.254/latest/meta-data/public-ipv4) > /var/www/projectlamp/index.html
```
![Root dir content](/LAMP%20STACK/img/fill_root_dir.png)

__9.__ __Open the website on a browser using the public IP address.__
```
http://3.82.206.119:80
```
![publicIP](/LAMP%20STACK/img/public%20IP.png)

_10.__ Open the website with public dns name (port is optional)
```
http://<public-DNS-name>:80
```
You can keep this file as a temporary landing page for the application until an `index.php` file is configured to replace it. Once the `index.php` file is in place, consider renaming or removing the `index.html` file from the document root, as it will take precedence over the `index.php` file by default.

## Step 5 - Enable PHP on the website

Under Apache's default DirectoryIndex setting, the `index.html` file will consistently take precedence over the `index.php` file. This feature proves handy for setting up maintenance pages in PHP applications. By crafting a temporary `index.html` file with informative content for visitors, you can seamlessly redirect them to the maintenance page. Following the maintenance period, renaming or removing the `index.html` file from the document root will restore the regular application page.

To alter this behavior, one should edit the `/etc/apache2/mods-enabled/dir.conf` file. Adjusting the order in which the `index.php` file appears within the DirectoryIndex directive will facilitate the desired changes.

__1.__ __Open the dir.conf file with vim to change the behaviour__
```
sudo vim /etc/apache2/mods-enabled/dir.conf
```
```
<IfModule mod_dir.c>
  # Change this:
  # DirectoryIndex index.html index.cgi index.pl index.php index.xhtml index.htm
  # To this:
  DirectoryIndex index.php index.html index.cgi index.pl index.xhtml index.htm
</IfModule>
```
![dir_index](/LAMP%20STACK/img/dir-index.png)

![index_php_config](/LAMP%20STACK/img/index_php_config.png)

_2.__ __Reload Apache__

Apache is reloaded so the changes takes effect.
```
sudo systemctl reload apache2
```
![reload_apache2](/LAMP%20STACK/img/reload_apache2.png)

__3.__ __Create a php test script to confirm that Apache is able to handle and process requests for PHP files.__

A new index.php file was created inside the custom web root folder.

```
vim /var/www/projectlamp/index.php
```

__Add the text below in the index.php file__
```
<?php
phpinfo();
```
![Php index](/LAMP%20STACK/img/index_php.png)

__4.__ __Now refresh the page__

![Php_page](/LAMP%20STACK/img/php_page.png)

This page offers insights into the server's PHP perspective, aiding in debugging and verifying the correct application of settings. However, due to the sensitive nature of the PHP environment and Ubuntu server details contained within, it's advisable to remove the file after accessing the pertinent server information. If necessary, the file can be recreated at a later time to retrieve the information required.
```
sudo rm /var/www/projectlamp/index.php
```

__Conclusion:__

The LAMP stack offers a resilient and adaptable platform for both developing and deploying web applications. By adhering to the guidelines provided in this documentation, it becomes feasible to establish, configure, and sustain a LAMP environment efficiently. This facilitates the creation of potent and expandable web solutions.