# Ushahidi Installation

These instructions explain how to install the Ushahidi v3 platform for development

## Requirements

- PHP
- Composer
- VirtualBox
- Vagrant

I will explain how I installed all of these on a Mac. The original instructions (that are a little vague) can be found at `https://www.ushahidi.com/support/install-ushahidi#installing-for-development`

First, clone the Ushahidi platform from github (the right one this time!) and navigate into the directory:
```
git clone https://github.com/ushahidi/platform.git
cd platform
```

### Install PHP

Homebrew seems like the easiest way to install PHP on a Mac, so if you don't already have it, you can download and install with
```
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```
Their page and instructions can be found here if you need: `https://brew.sh/`

- then install PHP:
```
brew tap homebrew/homebrew-php
brew install php71
```

### Install Composer
instructions from: `https://getcomposer.org/doc/00-intro.md`

- download composer:
```
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php -r "if (hash_file('SHA384', 'composer-setup.php') === '544e09ee996cdf60ece3804abc52599c22b1f40f4323403c44d44fdfdd586475ca9813a858088ffbc1f233e9b180f061') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
php composer-setup.php
php -r "unlink('composer-setup.php');"
```
- make composer global on your system and then install:
```
mv composer.phar /usr/local/bin/composer
composer install --ignore-platform-reqs
```

### Install Vagrant

- download here: `https://www.vagrantup.com/downloads.html`
- run the installer after downloading

### Install VirtualBox

- download here: `https://www.virtualbox.org/wiki/Downloads`
- run the installer after downloading

### Server Setup

- setup vagrant:
```
vagrant up && vagrant provision
```
- configure the database and run migrations:
```
cp .env.example .env
composer migrate
```

Now visit `http://192.168.33.110` in your browser to make sure the API is running. You should see a JSON doc with API info.

## Ushahidi Client Installation

- install node.js
```
brew install node
```

- navigate out of the platform directory, clone the platform client, then navigate into it:
```
cd ../
git clone https://github.com/ushahidi/platform-client.git
cd platform-client
```

- install build requirements and packages:
```
npm install -g gulp
npm install
```

- create a .env file:
```
nano .env
```
add the following lines:
```
NODE_SERVER=true
BACKEND_URL=http://192.168.33.110
```
- "control-x", "y", and then "enter" to save your changes and exit

- run gulp to start the local development server:
```
gulp
```

It will say the localhost address in the terminal output to view the client, but it should be viewable at `http://localhost:3000`









