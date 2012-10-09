How to Install GBS on EC2 and RDS
=================================

This document assumes you know how to manage linux via the commandline, how to setup an AWS account and how to setup a domain's DNS for EC2.

Launch EC2 Instance
----------------------

https://console.aws.amazon.com/ec2/home

* To create a new instance, access the AWS Management Console and click the EC2 tab:
* Choose an AMI in the classic instance wizard: I chose the Ubuntu Server 12.04.1 LTS AMI.
* Instance details: Select the Instance Type you want to use. I chose Small (m1.large). http://s-v.me/K0GY
* Skip steps until you can create a new key pair. Enter a name for your key pair (i.e. gbs) and download your key pair (i.e. gbs.pem).
* Select the quick start security group.
* Launch your instance.


Select the Type of EC2 instance you'd prefer. In order to get X conurrent users under $100/month for this walkthrough we used "m1.large". You will not want to use a Micro Instance, read more about why "here":http://gregsramblings.com/2011/02/07/amazon-ec2-micro-instance-cpu-steal/

Login
-----

SSH into the newly provisioned server.

Use the public DNS (found in the instance details) and the location of the downloaded security certificate (in previous steps) to connect.

e.g. 
> ssh ubuntu@ec2-204-236-213-81.compute-1.amazonaws.com -i /location/of/download/gbs.pem 

If you receive permission errors:

> Permissions 0644 for '/Users/dancameron/Desktop/gbs.pem' are too open.
> It is required that your private key files are NOT accessible by others.
> This private key will be ignored.

Modify the file permissions 

e.g.
> chmod 600 ~/christophe.pem

Switch to a super user
> sudo su

Install firewall
----------------

> ufw allow ssh
> ufw allow http
> ufw logging off
> ufw enable

Your server will now have a relatively secure firewall, although it may be work looking into fail2ban to prevent brute force password attacks.

Install and Configure MySQL
---------------------------

This step can be skipped if you plan on just using RDS.

> apt-get update
> apt-get install mysql-server

Set a mysql “root” user password when prompted.

Next setup the DB.

> mysql -u root -p

When prompted, enter the password set in the previous step.

At the mysql> prompt, run the following 4 commands, replacing ENTER_A_PASSWORD with a password of your own

mysql> CREATE DATABASE wordpress;
mysql> GRANT ALL PRIVILEGES ON wordpress.* TO "wp_user"@"localhost" IDENTIFIED BY "ENTER_A_PASSWORD";
mysql> FLUSH PRIVILEGES;
mysql> EXIT

Install and configure PHP
-------------------------

Install not just PHP, but the PHP FPM system, APC, and the MySQL module

> apt-get install php5-fpm php-pear php5-common php5-mysql php-apc

Edit /etc/php5/fpm/php.ini and add these lines at the bottom:

> [apc]
> apc.write_lock = 1
> apc.slam_defense = 0

Edit /etc/php5/fpm/pool.d/www.conf

Replace

> listen = 127.0.0.1:9000

with

> listen = /dev/shm/php-fpm-www.sock

Below that, insert these 3 lines

> listen.owner = nginx
> listen.group = nginx
> listen.mode = 0660

Then, further down in the same file, replace these 2 lines

> user = www-data
> group = www-data

with

> user = nginx
> group = nginx

Save the file, PHP FPM is now complete, but it won’t work until we install nginx.

Install Nginx
-------------

Instructions based on the Nginx website.

Download the nginx secure key to verify the package

> cd /tmp/
> wget http://nginx.org/keys/nginx_signing.key
> apt-key add /tmp/nginx_signing.key

Add the sources to the APT sources file by running these 2 commands (the >> is important!)

> echo "deb http://nginx.org/packages/ubuntu/ lucid nginx" >> /etc/apt/sources.list
> echo "deb-src http://nginx.org/packages/ubuntu/ lucid nginx" >> /etc/apt/sources.list

Download and install nginx by running

> apt-get update
> apt-get install nginx

When that completes, nginx will be installed, but needs configuring for WordPress.

nginx configuration files are in /etc/nginx

Configure Nginx
---------------

First, edit /etc/nginx/nginx.conf

Inside the http section, insert the following line so that when you later add varnish in front, things don’t break all over the place:

> port_in_redirect off;

Next, cd to /etc/nginx/conf.d and create these new files

> wget https://raw.github.com/GroupBuyingSite/Wordpress-Server-Configuration-Files/master/drop
> wget https://raw.github.com/GroupBuyingSite/Wordpress-Server-Configuration-Files/master/default.conf

Make a directory, /var/www/ and set the ownership:

> mkdir -p /var/www/
> chown nginx:nginx /var/www/
> chmod 775 /var/www

That’s nginx configured, restart it and the PHP FPM service by running:

> service nginx restart
> service php5-fpm restart


Install RDS Database Instance
-----------------------------

Use the DB wizard and launch an instance. e.g. http://s-v.me/K00U

This instance is using an m1.large class to get good enough performance for X concurrent users. Make it an Multi-A Deployment.

Add the EC2 Security Group so that the DB instance can be accesed by your EC2 Server

Install/Configure WordPress
---------------------------

http://codex.wordpress.org/Installing_WordPress


Optimizations
=============

Caching
-------

Go to the wordpress admin page, then plugins, and click install new plugin.

Search for “W3 Total Cache”, then click “Install Now” when the search results return. When installation is complete, click “Activate Plugin”.

Go to the new “Performance” section in the menu at the left side of the page.

Scroll through the cache options, selecting “PHP APC” at each opportunity and enabling the following 2 sections:

> Database Cache
> Object Cache

Hit “Save All Settings” then hit “Deploy”

