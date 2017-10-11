# Ushahidi Installation

These instructions explain how to install Ushahidi on Ubuntu through Virtual Box.

### Requirements
- download Virtual Box: `https://www.virtualbox.org/wiki/Downloads`
- download Ubuntu Server 16.04: `http://releases.ubuntu.com/16.04.3/ubuntu-16.04.3-server-amd64.iso`

### Ubuntu Installation
- after both downloads have completed, install and run Virtual Box
- create a new VM and give it a name like "ushahidiserver" with type "Linux" and version "Ubuntu (64-bit)"
- select your RAM and storage settings (i chose the defaults), and then start the new machine
- when prompted for a disk, select the Ubuntu .iso image from your downloads
- upon boot, proceed to install Ubuntu with defaults (except select "Guided - use entire disk" for disk options)
- when installation is complete and the machine has rebooted, login with the username and password you just created

At this point, all we have left to do with the virtual machine is get ssh working. This way, we can install the Ushahidi platform from our native system and not have to work in the virtual machine.

### SSH Setup
- on your virtual machine, install ssh: `sudo apt-get install openssh-server`
- show your virtual device ip address info: `ifconfig -a`
- find the inet addr under the first group (mine is 10.0.2.15) and remember that for a later step
- the inet addr under Local Loopback should be 127.0.0.1

Now, we should no longer need this window, but we do need the machine to continue running.

- type `exit` to logout and then minimize the window 
- in the main virtual box window, click settings and go to the network tab
- at the bottom, click advanced, then port forwarding, and then the plus button on the right side
- the first rule we create will allow ssh access, so name it "ssh"
- Protocol is still TCP, Host IP is 127.0.0.1, Host Port is 2222, Guest IP is the first IP from above where mine was 10.0.2.15, and Guest Port is 22
- while we're here, we can add another rule to allow http traffic that we'll use later, so name a second rule "http"
- all other options will be the same for this rule except Host Port is 8080 and Guest Port is 80
- click "OK" and "OK"

Let's test SSH!

- you can minimize all Virtual Box windows and then open a terminal on your native system
- login to your virtual machine: `ssh username@127.0.0.1 -p2222` where username is what you originally logged into Ubuntu with
- it should ask if you want to continue connecting and then prompt you for your ubuntu password
- if you see your username@ubuntu just like it showed in Virtual Box, then you're all set! otherwise, check that your IP address and port settings match correctly

### Apache Web Server Setup
- an apt update never hurts: `sudo apt-get update` 
- install apache2: `sudo apt-get install apache2`

One time after I turned the VM off and on, I tried to use apt-get and got the errors:
```
E: Could not get lock /var/lib/dpkg/lock - open (11: Resource temporarily unavailable)
E: Unable to lock the administration directory (/var/lib/dpkg/), is another process using it?
```
If this ever happens, restarting Ubuntu in the VM with `sudo reboot` fixes the problem.

- since we've already set up port forwarding, we can verify that apache works by visiting `http://localhost:8080` in your browser
- you should see the default Apache page that says it works!

Now we will setup a more convenient public directory to clone Ushahidi in.

- in your home directory (you should already be here), create a new directory: `mkdir Ushahidi_Web`
- open an apache conf file: `sudo nano /etc/apache2/sites-available/000-default.conf`
- change the Document Root from `/var/www/html` to `/home/username/Ushahidi_Web` where username is your username
- "control-x", "y", and then "enter" to save your changes and exit
- open the main apache conf file: `sudo nano /etc/apache2/apache2.conf`
- give permissions to the new root: scroll down to `<Directory /var/www/>` and change it to `<Directory /home/username/Ushahidi_Web>`, again with your username
- while we are here, change `AllowOverride None` to `AllowOverride All` below the Directory line we just changed (recommended by Ushahidi)
- "control-x", "y", and then "enter" to save your changes and exit
- restart apache to initiate changes: `sudo service apache2 restart`

We will test apache again to see if this new directory works.

- from your home directory, `cd Ushahidi_Web` to navigate into it
- create a test page: `nano index.html` and then type something in the file like "it works!"
- "control-x", "y", and then "enter" to save your changes and exit
- again browse to `http://localhost:8080` and you should see just a simple "it works!" in plain black text
- once working, delete the test index.html you just created: `rm index.html`

### Ushahidi Requirements
- an apt update never hurts: `sudo apt-get update`
- install PHP and the Apache PHP module: `sudo apt-get install php libapache2-mod-php`
- install PCRE: `sudo apt-get install libpcre3 libpcre3-dev`
- install mcrypt: `sudo apt-get install php7.0-mcrypt`
- install SPL: `sudo apt-get install spl`
- install mbstring: `sudo apt-get install php-mbstring php7.0-mbstring php-gettext`
- install MySQL: `sudo apt-get install mysql-server php-mysql`
- install cURL extension: `sudo apt-get install php7.0-curl`
- install IMAP: `sudo apt-get install php7.0-imap`
- install GD: `sudo apt-get install php7.0-gd`
- enable Ushahidi's "Clean URLS" feature: `sudo a2enmod rewrite`
- restart apache to initiate all changes: `sudo service apache2 restart`

### Ushahidi Installation
- return to your home directory: `cd`
- make sure you see your public directory "Ushahidi_Web" with `ls`
- remove the directory (we are about to get a new one with an identical name): `rm -r Ushahidi_Web`
- clone the Ushahidi project: `git clone --recursive git://github.com/ushahidi/Ushahidi_Web.git`
- ensure directories are writable:
```
cd Ushahidi_Web
chmod -R 777 application/config
chmod -R 777 application/cache
chmod -R 777 application/logs
chmod -R 777 media/uploads
chmod 777 .htaccess
```
- create a MySQL database "ushahidi": `mysqladmin -u root -p create ushahidi`
- enter the password you created when setting up MySQL
- login to MySQL: `mysql -u root -p`
- enter MySQL command `GRANT SELECT, INSERT, DELETE, UPDATE, CREATE, DROP, ALTER, INDEX, LOCK TABLES on ushahidi.* TO 'root'@'localhost' IDENTIFIED BY 'password';` where password is your MySQL password (include single quotes)
- you should get a "Query OK" reponse if entered correctly
- quit MySQL: `exit`
- check out the Ushahidi install page at `http://localhost:8080`
- click "Proceed with basic"
- click "Let's get started!"

Now we're in the place where any extension or requirement issues are addressed. Hopefully, all but one remains in your list. And that is `The mysql extension is disabled`. Apparently, the mysql extension for php is deprecated and has been removed from php7 entirely. See `http://php.net/manual/en/mysql.php`. The mysqli extension, its replacement, is already installed on our server, but the Ushahidi platform has not been updated to check for it. Someone has brought up this issue on the Ushahidi github page, but as far as I can tell, their code fix has not been approved to be merged. See `https://github.com/ushahidi/Ushahidi_Web/pull/1463`. So, I took their code fixes and made a new wizard.php that we can use to correct the problem locally. You can download it from this respository.

- back in the terminal logged into your virtual server, remove the current wizard.php:
- ```rm ~/Ushahidi_Web/installer/wizard.php```
- once the new wizard.php is in your downloads, copy it to the server from a terminal that's on your native system, not logged into the virtual server:
- ```scp -P2222 ~/Downloads/wizard.php username@127.0.0.1:~/Ushahidi_Web/installer/```
- where username is your virtual server username, like usual
- back in the terminal logged into your virtual server, check to make sure the new file is in the right place: `ls ~/Ushahidi_Web/installer/`

The rest of the installation will take place in your browser.

- refresh the page, and you should see a prompt for database info
- Database Name: ushahidi
- User Name: root
- Password: yourmysqlpassword
- Database Host: localhost
- leave Table Prefix blank
- click "Continue"
- enter name info (I themed everything after the Food Rescue)
- enter password info

### Installation Successful!

It breaks right away when trying to login, but I've already found the fix here: `https://github.com/ushahidi/Ushahidi_Web/pull/1466/files`
We can fix this when we have a universal version in github for everyone, or feel free to make the changes locally now.















