# Welcome to Web Development
The beginning of this independent study shows users how they can quickly create a development environment for web application development. This independent study sheet will take a user through the steps to make a virtual environment emulate a server's environment. This is a very simple process that feels complex. Nevertheless, a few steps - and perhaps a few beers - later, and you'll have a perfect virtual environment to create a simple virtual environment that emulates a real server on AWS. Note if you want to turn this into a professional environment, I would take this guide with a grain of salt as it is just for quick development prototyping.

## What we're making
A LAMP server hosted by AWS EC2
A Vagrant virtualbox

Both of these environments will utilize the Lumen framework created by Laravel.

## Requirements
I'm developing with Ubuntu 16.04 LTS, feel free to use any OS to your liking, but this guide will have linux commands, etc.
Vagrant
AWS Account


# Server Configuration
Amazon AWS makes web development easy with providing users quick creation of EC2 instances (servers). For this guide, we'll create a new EC2 instance that is free tier *hallelujah*. We'll also be making this instance backed by Laravel's Lumen framework.


## AWS Instance Creation
If you have not already created an AWS account, please do so now. Otherwise, let's get started. Begin by going to the EC2 instance management console, and launch a new EC2 instance via the Launch Instance tab. Next, select Ubuntu 18.04 LTS (HVM), SSD Volume Type image. Make sure you have the t2.micro (the current free tier instance type) selected so this experiment remains free. After you have selected the instance type, press the blue Review and Launch button. Create a new key pair and store this ".pem" file in a safe location as we will need this to connect to our newly created instance. Ensure this ".pem" file has chmod permissions of 400, so that way, we can connect to our instance. Also, make sure you write down the public DNS IPv4. This is located in the details panel below the instance table on the  EC2 Instances Dashboard.

## Security Group Configuration
After the Instance launching process has been completed, navigate through AWS until you can view your EC2 Instances. Then, select the newly created instance and look at the details panel below the instance table. Scroll down until you see the Security groups section and click on launch-wizard-<n> - where n is some number. This will navigate you to the Network and Security section called Security Groups where one security row will populate the security group table. Click on the Actions dropdown button and go to Edit inbound rules. Select Add Rule and select the type HTTP and ensure the details are as follows: Protocol: TCP, Port Range: 80, Source: Custom - 0.0.0.0/0, ::/0. This will allow us to connect to the server via HTTP (through a web browser). We also need to open port 3306 so that we can remotely work with the mysql database. So create a new rule with the following details: Protocol: Custom TCP, Port Range: 3306, Source: Anywhere. Now go back and revisit the EC2 Instances Dashboard. 

## Connect to the EC2 Instance
Remember the location of that key pair we just created? I hope so because we'll be using that in this section. In order to easily ssh to this server, I find that creating an alias in our 	~/.bashrc file is the simplest manner of going about this. Here's mine so you can base yours off of it: 
	<pre><code>alias devServer="ssh -i  <path/to/.pem> ubuntu@<public DNS>"</code></pre>

Don't forget to run source ~/.bashrc if you're adding an alias!

## Install LAMP
From here on out, unless specified, you'll type these commands into the ssh'ed terminal. Now that you're connected to the EC2 server, we're going to loosely follow this fantastic guide to install exactly what we need for our LAMP server. The first thing to do, after SSH-ing into our server, is update and upgrade everything through:
	<pre><code>sudo apt-get update && sudo apt-get upgrade -y</code></pre>
	
Next, we'll install apache2:
	<pre><code> sudo apt-get install apache2 -y</code></pre>

Now that apache2 is installed, we can test our site by connecting to it via web browser. Type in your public DNS into some web browser and you should see the Apache2 Ubuntu Default Page. Now, we need to install PHP 7.2:
	<pre><code> sudo apt-get install php7.2 -y</code></pre>

PHP has been installed, and we just have to install mysql now.
	<pre><code> sudo apt-get install mysql-server php7.2-mysql -y</code></pre>


## Connect to mysql Remotely
We're going to create a very basic user to connect to mysql:
<pre><code>  
sudo su
mysql -u root
CREATE USER 'user'@'%' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON *.* TO 'user'@'%' WITH GRANT OPTION;
FLUSH PRIVILEGES;
quit
</code></pre>

Now that we have created a user that has remote access, we need to edit the mysqld.conf file or the my.conf file (whichever file has bind-address in it). Most recently, the file you need to edit is /etc/mysql/mysql.conf.d/mysqld.conf You can find the paths to these files in /etc/mysql/my.conf if it isn't populated with settings. 
<pre><code>	
nano /etc/mysql/mysql.conf.d/mysqld.cnf
(in nano)
Ctrl+w bind-address
(Comment out the bind-address line with a #.)
</code></pre>

We need to restart our mysql server with these new changes
<pre><code>service mysql restart</code></pre>

Now test the login by opening a new terminal and connecting remotely:
<pre><code>mysql -h <EC2-Instance-IP> -P 3306 -u user -ppassword</code></pre>

When you're ready to leave mysql, just type in quit. If you get error 2003, make sure you have commented out #bind-address and have the TCP port open as described in the section Security Group Configuration.

## Prepare for Lumen
### Install Composer
Thanks to a fantastic github post, we can use the following string of commands to install composer.
<pre><code>
exit (to exit root account)
cd ~
sudo curl -sS https://getcomposer.org/installer | sudo php
sudo mv composer.phar /usr/local/bin/composer
sudo ln -s /usr/local/bin/composer /usr/bin/composer
</code></pre>

### Install Lumen
Like any framework, we have a set of requirements that we need to fulfill to properly utilize it. We will be following the lumen installation guide. The following command will give us those requirements:	
<pre><code>sudo apt-get install php7.2-dom php7.2-zip php7.2-mbstring -y</code></pre>

After these required packages have been installed, we need to navigate to our html directory.
<pre><code>cd /var/www/html </code></pre>

Now that we're in our html directory, we can create a lumen project.
<pre><code>sudo composer global require "laravel/lumen-installer"</code></pre>

In order to use our newly created lumen executable, we can follow steps similar to the composer installation steps.
<pre><code>sudo ln -s /home/ubuntu/.composer/vendor/bin/lumen /usr/bin/lumen</code></pre>
If this step doesn't work, try the path ~/.config/composer/vendor/bin path instead of ~/.composer/vendor/bin

Now we need to grant permissions to the html directory so that we can create a new lumen installation.
<pre><code>sudo chmod 777 /var/www/html</code></pre>

Everything is set up so now you can create a new installation, change out <appname> to whatever you want to call your application!
<pre><code>lumen new <appname></code></pre>

Now, in your favorite web browser, connect to your public DNS and append /<appname>/public to the end of the URL. This will take you to a page that says "Lumen (5.7.6) (Laravel Components 5.7.*)"

## Enable Source Control 
This is exciting! Lumen is fully installed and now we just want to add this to github or some other source control service. I'm just going to use github for this. So create a new, empty repository and then switch back to your terminal for the remainder of this portion
<pre><code>	
git init
git remote add origin <link-to-your-newly-created-repo>.git
git add .
git commit -m "first commit"
git push origin master
</code></pre>

## Extra configuration
Run the following commands to get rid of the need to specify the appname.
<pre><code>	
sudo mv <appname> ..
sudo rm -rf html
sudo mv <appname> html
</code></pre>

With these last commands run, we are now fully prepared to create our local environment!

## Allow routes
There's one more catch with Lumen. Unfortunately, as much as I wish this were just plug-and-play, we have a few more things to rewrite. First we need to update Apache's sites-available to allow other routes to exist. 
<pre><code>	
sudo nano /etc/apache2/sites-available/000-default.conf
(in nano)
[update DocumentRoot to /var/www/html/public instead of /var/www/html]
(go to the end of the document)
(type in the following)
&lt;Directory /var/www/html/public&gt;
	Options Indexes FollowSymLinks
	AllowOverride All
  Require all granted
&lt;/Directory&gt;
</code></pre>

The final step we need to do is enable a2enmod rewrite witch allows apache to utilize lumen's .htaccess file.
<pre><code>sudo a2enmod rewrite</code></pre>

Now, just restart the apache service!
<pre><code>sudo service apache2 restart</code></pre>




# Vagrant Configuration
Vagrant makes local development easy. With many guides, you'll quickly develop in the environment you so desire. Luckily, Ubuntu 18.04 exists as a vagrant box so that we can emulate our server exactly on our local machine. 

## Vagrant box creation
Clone your created project into some directory and make sure you're in that directory on your local machine.
<pre><code>vagrant init ubuntu/bionic64</code></pre>

### Vagrantfile Modification
We have just created a "Vagrantfile" that we will edit to contain all of our specifications to emulate our server. There's a few modifications we want to make to this file, and we'll use nano to make these changes.
<pre><code>nano Vagrantfile</code></pre>
(in nano)

Allow mysql access to the private network.
<pre><code>Ctrl+W config.vm.network "forwarded_port"</code></pre>
(delete the #)
(change both guest and host values to 3306)

Allow access to this machine via private IP.
<pre><code>Ctrl+W config.vm.network "private_network"</code></pre>
(delete the #)

Set the synced folder (so that we can have the project file in the same location as the server)
<pre><code>Ctrl+W config.vm.synced_folder</code></pre>
(replace everything after config.vm.synced_folder with ".", "/var/www/html")

### Running Vagrant
With the changes made, we can start our newly created vagrant box.
<pre><code>vagrant up</code></pre>

## Install LAMP
This command will create and start an instance of Ubuntu 18.04. We then want to update and upgrade everything inside this instance.
<pre><code>sudo apt-get update && sudo apt-get upgrade -y</code></pre>

## Install Apache
Once again, we will install apache2, but this time we're going to edit the configuration file.
<pre><code>
sudo apt-get install apache2 -y
sudo nano /etc/apache2/apache2.conf
</code></pre>

We just have one minor thing to insert at the bottom of the file: ServerName localhost. Once you have saved and exited the file, restart the apache server.
<pre><code>sudo service apache2 restart</code></pre>

### Test Apache Server
We added some private networked IP, but we should make a URL that will be easily recognizable for testing. Edit your hosts file and add the IP followed by a domain.
<pre><code>sudo nano /etc/hosts</code></pre>

Add 192.168.33.10 <domain name (i'm using testing.dev)> to the hosts file. Then restart your vagrant instance 
<pre><code>vagrant reload</code></pre>

Once the  vagrant reload process has completed, open your web browser and the domain name in. This should take you to the very familiar Apache2 Ubuntu Default Page.

### Install PHP
Just like our EC2 Instance, we need to install php7.2 to fulfill the P in LAMP.
<pre><code>sudo apt-get install php7.2 -y</code></pre>

### Install mysql
PHP has been installed, and we just have to install mysql now.
<pre><code>	sudo apt-get install mysql-server php7.2-mysql -y</code></pre>

Just like on EC2, we need to modify the mysqld.conf
<pre><code>sudo nano /etc/mysql/mysql.conf.d/mysqld.conf
(in nano)
Ctrl+W skip-external-locking
(add # to skip-external-locking [if it exists])
Ctrl+W bind-address
(add # to the beginning of the bind-address line)
</code></pre>

For safety's sake we should run the secure installation for mysql.
<pre><code>	sudo mysql_secure_installation</code></pre>

Now we should restart Apache and MySQL
<pre><code>
sudo service apache2 restart
sudo service mysql restart
</code></pre>

### Testing MySQL Remotely
We need to make a basic account that will allow for remote access at any location.
<pre><code>
sudo su
mysql -u root
(in mysql)
CREATE USER 'user'@'%' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON *.* TO 'user'@'%' WITH GRANT OPTION;
FLUSH PRIVILEGES;
quit
</code></pre>

Now we should restart Apache and MySQL
<pre><code>
exit (to leave root)
sudo service apache2 restart
sudo service mysql restart
</code></pre>

Using some Database editor, you need to add the mysql details:
<pre><code>
HOST: 192.168.33.10
USERNAME: user
PASSWORD: password
PORT: 3306
</code></pre>

The SSH tunnel has the following details:
<pre><code>
HOST: 127.0.0.1
USERNAME: vagrant
KEY: <path/to/project/folder/>/.vagrant/machines/default/virtualbox/private_key
Port: 2222
</code></pre>

After connection, you have successfully set up the LAMP server.

## Prepare for Lumen
Yes, this is copied from above, and that's because it's exactly the same!
### Install Composer
Thanks to a fantastic github post, we can use the following string of commands to install composer.
<pre><code>
cd ~
sudo curl -sS https://getcomposer.org/installer | sudo php
sudo mv composer.phar /usr/local/bin/composer
sudo ln -s /usr/local/bin/composer /usr/bin/composer
</code></pre>

### Setup Lumen
Like any framework, we have a set of requirements that we need to fulfill to properly utilize it. We will be following the lumen installation guide. The following command will give us those requirements:	
<pre><code>	sudo apt-get install php7.2-dom php7.2-zip php7.2-mbstring -y</code></pre>

After these required packages have been installed, we need to navigate to our html directory.
<pre><code>	cd /var/www/html </code></pre>

Now that we're in our html directory, we can install the previously made project.
<pre><code>	composer install</code></pre>

Now we need to grant permissions to the html directory so that we can create a new lumen installation.
<pre><code>	sudo chmod 777 /var/www/html</code></pre>


Now, in your favorite web browser, connect to your host file domain name and append /public to the end of the URL. This will take you to a page that says "Lumen (5.7.6) (Laravel Components 5.7.*)"

## Allow routes
Yes, just like EC2, we need to update Apache's sites-available files to allow other routes to exist in our local environment. 
<pre><code>	sudo nano /etc/apache2/sites-available/000-default.conf</code></pre>
(in nano)
[update DocumentRoot to /var/www/html/public instead of /var/www/html]
(go to the end of the document)
(type in the following)
&lt;Directory /var/www/html/public&gt;
  Options Indexes FollowSymLinks
  AllowOverride All
	Require all granted
&lt;/Directory&gt;
</code></pre>

 The final step we need to do is enable a2enmod rewrite witch allows apache to utilize lumen's .htaccess file.
<pre><code>sudo a2enmod rewrite</code></pre>

Now, just restart the apache service!
<pre><code>sudo service apache2 restart</code></pre>


Save your vagrant box
If you want to save/backup your current box so you don't have to configure anymore just run vagrant package and this will create a .box file that you can recover from. You can also upload this .box file to vagrant cloud. Also, you can set it as the config.vm.box value and it'll always pull from it. Make sure to add *.box to your .gitignore file as .box files are too big for github. 
