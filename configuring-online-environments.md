# Configuring Online Environments

This document explains the steps of configuring an Ubuntu Linux 18.04 Server into a php enabled web server & getting your web application online. 

I will be deploying 2 separate projects : 
- **Frontend:** React JS Website
- **Backend:** Laravel API

In most cases, what you will have after following this guide should work for most other solutions / frameworks. If not, you will have a good understanding to enable you to do whatever small change is needed to get it working.

## Introduction

Usually, my job finishes as soon as I push my final git commit ***(if you are working on a real life project without version control, please consider using git to avoid unnecessary headaches :)***. I never had to deploy projects myself. You might be just like me in that regard. At some point, you figured out that the web application you made in a month was sold for four times your salary. You got pissed, started looking for freelancing oppurtunities to get paid decently for once, you got a project, congratulations ! you worked a bit on it and want to show your client what you have so far, well fuck... you have no idea how to get it online do you? Devops has always been a mystery to you and the fact your application is in PHP is not making it any easier. You sure as hell are not paying for those quick deploy apps like [**Laravel Forge**](https://forge.laravel.com/).

Here you will find everything you need to know about how applications work on the internet and how you can get your application online yourself.

## Why You Need a Cloud Platform

First thing's first, so far you have been seeing your application on your computer. you have some kind of web server installed locally like wamp, xamp, mamp or maybe even apache2 or nginx. Your computer is acting as the host machine with a web server that makes your computer listen on port 80 so when you type ```localhost``` or you local ip ```127.0.0.01``` in the browser, that web server will serve the files in your server's root html directory. (/var/www/html, C://wamp/www/, C://xamp/htdocs/, etc..). 

Great, so all you need is to hook up your computer to the internet right ? well you can do that, but that won't really workout for you. 

### Why you can't rely on your computer to host your website

I am sure you have internet at home / your office. for that to work your internet provider gave your router a public IP address so it can communicate with everyone else. you can see how the rest of the world sees you by checking your network's public IP [here](https://whatismyipaddress.com/). so what's the problem ? 

- notice how I said *your **network's** public ip*, **NOT** you computer's. yes, all computers / phones on your network will share that same public IP address. So that IP address is not really mapped to your host machine specifically which means the rest of the world wont be able to reach your computer by that IP. (you can configure the router depending on how advanced it is to forward messages sent to port 80 to your computer but because of the other issues I will list below it's not really gonna help).

- Remember that IP address you just saw [here](https://whatismyipaddress.com/), It actually changes. Visit that website in another time and you will see another public IP address. The one your provider gives your router is temporary and variable. At some point, you want to map your website's domain (somesite.com) to an IP address where it can find your web server, but if that IP keeps changing, your domain - IP mapping will break the second your public IP changes. That is why you need a **static ip** which is a public IP address your provider gives you that will never change. Why don't you just ask him to give you one ? well because not all Internet providers offer this and it costs a lot of money and you just cheaped out on using quick deploy apps so I'm sure you can't afford it.

- Miraculously, none of the issues above apply to you, what if your computer turns off ? your web application will be down, can you guarantee your computer will never turn off or break ? 

- It's not scalable. Say you have a really good computer, but your web application is gaining publicity and you are getting millions of users in traffic at once, your computer will be physically unable to handle that kind of load, and it will take you way too long to but a new CPU, bigger RAMs, more storage, assemble them ... 

So I think we established your computer won't do it. What you need is a server provided by a good cloud platform like Google Cloud, AWS, Digital Ocean, etc.

### Benefits of using a cloud platform

- Assign a static ip, exclusively for your server for next to no cost.
- Google / AWS / Digital Ocean server machines don't simply get turned off unexpectedly.
- If you need to scale it up and add more specs, more network devices like load balancers, and its done with the press of a button with very little down time.
- Renting such a server will cost you very little *(cost varies across different cloud platforms, usually Google Cloud is expensive, Digital ocean is more affordable, starting $5 / month)*
## Setting Up a Cloud Platform

I will be using Digital Ocean in this guide. Setup an account or login [here](https://cloud.digitalocean.com/)

### Setting up your virtual machine

You know you need a server by now, but you don't actually get to buy or rent entire server machines as they are usually too large in specs for normal developers to use. So what happens is that cloud platforms allow you to setup virtual machines as big as you need from their servers. Therefore, what you will do to get a server is create a virtual machine that has a server operating system installed on it. Thankfully, cloud providers have a simple interface for creating those where you can choose what specs you want and what kind of OS you want installed on it.

- Go to your digital ocean dashboard, and create a new droplet

  *A droplet is digital ocean's weird term for virtual machine.*

- Choose whatever specs you think are good and a server operating system. I will choose Ubuntu Server 18.04 for this guide. 

- After your droplet is done, to access your virtual machine's terminal, go to your dashboard > droplets and you will see the one you just made. Click on **more** > **access console**. A new popup window will appear which will serve as the terminal interface of your machine.

### Installing Basic Packages

For those of you are used to download software from websites by clicking buttons you might be wondering how you are supposed to download software from your server's terminal. Well, Linux has a command base tool that downloads software on your machine. For Ubuntu, that tool is called ```apt```. We will be using ```apt``` to download some basic software we need on our machine.

- wget

  This lets you download files from the internet

- unzip

  This lets you extract the contents of compressed files like .zip, .tar, etc.

- curl

  This lets you send http requests.

Before installing packages, it is best to update everything to make sure we have the latest copy of sources and such. we can easily do that by running:


``` bash
sudo apt-get update
sudo apt-get upgrade
```

You can use apt to download those programs like so:

``` bash
# Notice how you use the program apt, telling it to install a list of space seperated packages
sudo apt install wget unzip curl
```

### Setting Up SSH for virtual machine

SSH is basically a way to access another computer's terminal from your own. If you haven't setup SSH when creating the droplet, you can do so at anytime like this. This step is optional, but logging in to digital ocean every time you want to access that terminal is a hassle so let us allow ourselves to SSH into it directly from our computer's terminal. To do that follow these steps: 

- If you did not create any SSH keys on your computer yet, create one now by typing the following:

  ``` bash
  # Generates an ssh public & private key pair. 
  ssh-keygen
  ```

- Copy your SSH public key.

  ``` bash
  # After generating ssh keys, they end up in the .ssh folder in your home directory
  # Windows: C:\\Users\YourUserName\.ssh\
  # Linux / Mac: /home/YourUserName/.ssh/
  # The ssh keys are stored in files, unless you specified a name for it, it is usually called id_rsa.pub for your public key, and id_rsa for your private key.
  # To view your public key, you can type this into your computer's terminal:
  cat /home/UserName/.ssh/id_rsa.pub
  # or you can use any text editor to open that file and copy it the string inside.
  ```

- In your Server Machine's terminal, go to ``` /home/yourServerUserName/.ssh/known_hosts``` file and paste the key you copied there. If the file doesn't exist create it.

- Now, you can simply access your server's terminal by going into your computer's terminal and typing:

  ``` bash
  ssh yourServerUserName@yourServerIP
  # example: johndoe@123.456.789.2
  
  # Note: If you don't know your server's IP, you can go to your cloud dashboard > droplets. and you will see a column called IP Address near the name of your droplet.
  
  # Note 2: The IP assigned to your droplet is a static IP that will be exclusive to your droplet until your destroy it.
  ```

By now, you have a dedicated machine to do your bidding that you can access anytime, congratulations ! But now its more or less a brand new computer and you need to download everything you need on it and run whatever programs you wish. In our case, we have a host machine and just need to install php and a web server so we can have our React website and Laravel API served on it.

## Setting Up a Backend Environment

Our backend is made using a Laravel Project, which uses PHP. Our backed probably uses a database, most likely MySQL. Having said that we have our work cut out for us:

### Installing PHP

Since we wrote our backend with PHP, our virtual machine needs to have PHP installed. I will be using php 7.3 for this guide. Configuring PHP environments is not as easy as Go environments for example mostly because it needs a lot of dependencies but we can summarize the steps as follows:

#### Add the PHP repository

This basically lets your machine know where to get PHP from.

``` bash
sudo apt -y install lsb-release apt-transport-https ca-certificates 
sudo wget -O /etc/apt/trusted.gpg.d/php.gpg https://packages.sury.org/php/apt.gpg
echo "deb https://packages.sury.org/php/ $(lsb_release -sc) main" | sudo tee /etc/apt/sources.list.d/php7.3.list
```

#### Install PHP 7.3

``` bash
sudo apt -y install php7.3
```

to check if php installed successfully, you can type this to check what version of php you have:

``` bash
php -v
```

#### Install PHP's standard extensions 

PHP has several extensions that are more or less always used nowadays but are still kept as separate packages that need to be installed for the sake of making our lives harder. Here is the install command for most of these extensions.

``` bash
sudo apt install php7.3-cli php7.3-fpm php7.3-json php7.3-pdo php7.3-mysql php7.3-zip php7.3-gd  php7.3-mbstring php7.3-curl php7.3-xml php7.3-bcmath php7.3-json
```

If you want to confirm a single extension is installed you can use this command: 

``` bash
apt policy <extension-name>
# Example: apt policy php7.3-cli
```

#### Common Issues

The reason I said PHP environments are not that easy to configure is because most of the time, your will get an error saying this package was not found (either php or the php extensions). This is because you need a miracle to know what the right PHP repository / source is. For php 7.3, this should work.

### Installing Composer

Unlike other languages whos package managers are installed when installing the language itself, PHP's package manager, Composer, needs to be separately installed. Unfortunately, this one is not a simple apt install, also for the reason of making our lives harder, we need to do the following:

- Install a php script that will install composer for us

  ``` bash
  php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
  # php -r "" runs the php code given inside the quotes
  # The copy function will download the file https://getcomposer.org/installer serves us and stores it in a file called composer-setup.php
  ```

- We need to verify that our setup file is write and no one sent us a funny version. Composer's page on github tells us what the hash of the setup file should look like, so all we need to do is hash our file's content and compare it to the hash GitHub told us about.

  ``` bash
  # Get the hash Composer tells us about on their github page and store it in a variable called HASH
  HASH="$(wget -q -O - https://composer.github.io/installer.sig)"
  # Hash our file's content and compare that hash witht the variable we made above
  php -r "if (hash_file('SHA384', 'composer-setup.php') === '$HASH') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
  
  # if no funny business happened, we should get the following message: Installer verified
  ```

- Now that we verified we have a good setup file, let's run it so it can actually download composer for us

  ``` bash
  # To Run a php file, simply type 'php file-program-is-in.php'
  # We run the program written in composer-setup.php which accepts the following arguments:
  sudo php composer-setup.php --install-dir=/usr/local/bin --filename=composer
  ```

We are done with the needlessly complicated process of installing PHP's package manager. To test if it installed correctly you can just type: 

``` bash
composer
# Output: 
#______
#/ ____/___  ____ ___  ____  ____  ________  _____
#/ /   / __ \/ __ `__ \/ __ \/ __ \/ ___/ _ \/ ___/
#/ /___/ /_/ / / / / / / /_/ / /_/ (__  )  __/ /
#\____/\____/_/ /_/ /_/ .___/\____/____/\___/_/
```



### Installing MySQL Server

Usually Databases are stored on separate servers, most decently-scaled platforms have a separate Database cluster that their backend communicates with. This is good because you don't want any trouble in your code hosting machine affect the availability of your database. However, most cloud platforms offer that as a separate feature that costs more money. 

Being the smart cheapskates we are, we realize that a database cluster is basically a VM running a database server. Well, we have a VM. All we need to do is install and configure a database server on it. We will be installing MySQL in this guide.

To install MySQL Server, simply run

``` bash
sudo apt-get install mysql-server
```

A screen should show now asking you to configure stuff. If it didn't run this: 

``` bash
sudo mysql_secure_installation utility
```

Congratulations, you got a database server without paying extra for a database cluster. All you need to do is turn it on. There are the commands you can use for starting, stopping, restarting, etc.

``` bash
# To Start the database service once, run
sudo systemctl start mysql
# To make it start whenever the machine is turned on, run 
sudo systemctl enable mysql
# If you ever need to restart it, run 
sudo systemctl restart mysql
```

If you want to be able to access your database server from outside the machine (in our case we don't but in case you do), you will need to tell your firewall to allow connections to MySQL since they are blocked by default. To do that, run

``` bash
# Add rule to allow incoming traffic to the database server from outside: 
sudo ufw allow mysql
# Enable rules
sudo ufw enable
```



## Setting Up a Frontend Environment

In the good old days, having a web server that simply serves HTML, CSS & JavaScript files was all you need. We used to make a website using jQuery and Bootstrap and it would be amazing. The web server will serve our files to the client and everything just works. If that is your case now, feel free to skip this part.

Sadly, we no longer live in the time where jQuery reigns supreme. If you are not using a modern JavaScript framework like React, Angular, or Vue you will be judged by our pretentious community of developers. I found React to be the simplest of them but the environment we set up should work for all of them.

### Installing Node JS

At some point, someone wanted to use JavaScript to write backend code, and thus NodeJS was born. Why do we care ? Well ever since that unfortunate event, JavaScript frameworks now need Node to work as they use a lot of dependencies you will need to download with Node's package manager, ```npm```.  Thankfully, unlike PHP, installing node & npm is as simple as: 

``` bash
sudo apt-get install nodejs
# in some cases, npm will be installed with it, in case its not:
sudo apt-get install npm
```

To check the versions you installed you can run 

``` bash
node -v # outputs node js version
npm -v  # outputs npm version
```



## Install and Configure Nginx

Up till now, we installed all the environments we need to run the projects. But we still need to install the most important part, the web server; The program responsible for listening on web ports and serving our web files. Nginx lets you do many cool stuff too like routing, proxying, and will help you with scenarios you might run into in the future. So lets get it up and running.

### Install Nginx

``` bash
sudo apt-get install nginx
```

### Configure Nginx

We just installed Nginx, all that's left to do is turn it on and we're done, right ? sadly no... Nginx can do all that stuff but you still need to tell it what you want. 

if you go to ```/etc/nginx/nginx.conf``` you will find instructions telling Nginx what to do, this is the main configuration file, you don't want to mess with it. If you read it you will find it is also importing any file ending in ```.conf``` extension from certain directories. one of those directories is ``` /etc/nginx/sites-enabled/```. This means we can write config files for each project in that directory and Nginx will automatically import it.

Great, so we need to write some rules for our backend and frontend projects so Nginx can know how to handle requests and route them properly to our projects.

#### Rules For Our Backend Project

The result ```.conf``` version of the rules we set that Nginx would understand look like this:

``` bash
# We create a file called /etc/nginx/sites-enabled/api.somesite.conf
# The name of the file doesn't really matter but it'll help us know what file is for what project.
server {
    listen 80; # Listen on port 80
    server_name api.somesite.com; # The name of our server is this
    
    gzip on; # Enable gzip compression to the response smaller

    charset utf-8; # Use UTF 8 instead of default ASCII so our response can support more characters

    root /var/www/html/somesite-api/public; # this is where our project's entry point is
    
    # $uri is the url the user requested.
    # First, see if it matches an exact file name, if so serve it
    # If the above fails, see if it matches a directory name, if so go to it
    # if the above also fails, serve index.php but add all the URL info in the request
    # The third scenario is actually how most backend frameworks work nowadays, they all go to index.php and based on the query_string, they will go to different code paths. 
    try_files $uri $uri/ /index.php?$query_string;
    
    # An index file is the file we serve if no specific file is requested.
    # We tell Nginx that an index file will be named one of these: ['index.htnm', 'index.htm', 'index.php']
    index index.html index.htm index.php;

	# When location maps to these two files, ['/favicon.ico', '/robots.txt']
    # Disable Logging on these paths as they are unnecessary to log
    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }

	# When location is a php file, use our installed php program to interpret them 
    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php7.3-fpm.sock; # Note you might need to change this directory depending on your installation details.
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
    }

	# if the location is a hidden file, don't serve it since it is forbidden...
    location ~ /\.(?!well-known).* {
      deny all;
    }
}
```

#### Rules for Our Frontend Application

``` bash
# We create another file called /etc/nginx/sites-enabled/somesite.conf
server {
   listen 80; 
   server_name somesite.com; 

   gzip on;

   charset utf-8; 

   root /var/www/html/somesite-web/build; 
   index index.html index.htm;

   # Whatever the location is, try to match an exact file (for handling assets like css, js files & images)
   # If no exact file is found just go to index.html (React JS will handle routing from there).
   location / {
     try_files $uri /index.html;
   }
}
```



And that's it, we finished configuring Nginx ! for common operations you will need to do like starting, stopping, etc. you can run 

``` bash
# To Start the web server service once, run
sudo systemctl start nginx
# To make it start whenever the machine is turned on, run 
sudo systemctl enable nginx
# If you ever need to restart it, run 
sudo systemctl restart nginx

# Note whenever you change an Nginx configuration file, you will need to restart Nginx
```



## Setting Up Your Projects

So now, we have the environments that allow our code to run set up. Our web server is configured and ready. All we need is to setup our actual projects on our virtual machine. This is the easy part, hopefully you already have your repositories somewhere on GitHub or GitLab.  

### Setting Up GitHub Permissions for Your Server

Since you will be pulling new changes often, it is a hassle to provide your GitHub credentials every time you pull. So just like how you SSH into your server with your keys instead of credentials, let's create an SSH key pair on our server and tell GitHub to allow our Server's public key so it doesn't need to authenticate every time.  You will only do this step once.

``` bash
ssh-keygen # To Create SSH Key Pairs on your server
cd /home/YourUserName/.ssh/id_rsa.pub # Display your public key to your terminal
```

Copy your Key and go to your [GitHub Settings](https://github.com/settings/keys) and click on **SSH and GPG keys** > **New SSH key**. Paste your key into the **key** text area and give it whatever title you want. Now your server should be able to authenticate via SSH key instead of having to login every time !

### Setting Up Your Backend Project

In the configuration file for our API, we specified a root directory for our project so we better create it in the same path we told Nginx it will be in, pull our code files, and install any libraries we are using so it can run.

``` bash
# Go to your web server's root directory
cd /var/www/html 
# Create your project's folder with same name as the one you specified in api.somesite.conf
mkdir somesite-api 
# Link the folder to your repository and pull your code
git init
git remote add origin git@github.com:YouGitHubUserName:YourProjectName.git
git pull origin master
# Install all PHP library dependencies our project uses
composer install
```

Now, our code will run smoothly, but do not forget to change the database credentials your using in your code to match the ones on the server. *(Set them as environment variables or in the ```.env``` file for Laravel. Just never include them in your actual code as it is dangerous.)*

### Setting Up Your Frontend Project

Just like our API project, we need to create the directory we promised Nginx our code will be in, pull our code, install any dependencies and build our project.

``` bash
# Go to your web server's root directory
cd /var/www/html 
# Create your project's folder with same name as the one you specified in api.somesite.conf
mkdir somesite-web 
# Link the folder to your repository and pull your code
git init
git remote add origin git@github.com:YouGitHubUserName:YourProjectName.git
git pull origin master
# Install all JavaScript library dependencies our project uses
npm install
# Build our project.
# This applies to modern JavaScript applications. You often need to run a build command that essentially compile the advanced JavaScript syntax you use into simple JavaScript browsers understand, minifies resources like JavaScript files and CSS Files, etc.
# Our React template has a script set to run that does this building process called 'build'. So to run it, we will need to tell npm to run a script called 'build'. Which is done by saying:
npm run build
# My build script creates a build/ folder with the final version of html and simplified CSS / JS. Which is why in somesite.conf we defined the root to go to this build/ folder. If your files are built into a different folder like 'public/' or something like that you need to change the *root* directive in the Nginx conf file to match your case. 
```




## Allow Online Traffic

### Configure Firewall Rules to allow incoming http(s) traffic

Right now our **nginx** is listening on whatever ports we configured it to listen to. However, the firewall on our server is probably blocking all traffic coming from outside to those ports. 

*(so if you curl localhost from within your server's terminal, you will get the nginx index page, but if you curl your Server's IP from your computer's terminal, you get shit)*

However, you've wasted part of your life developing this crappy app, and if it's not public, you won't get paid. Assuming you are trying to deploy a normal web application, you will probably be interested in opening 2 ports to make it publicly accessible:

- **port 80** for http 
- **port 443** for https

The default firewall configuration tool for Ubuntu is ```ufw```. In case you do not have it installed by default you can install it by running:

``` bash
sudo apt install ufw
```

Now, we have a simple way of opening those ports which is as follows: 

``` bash
# Shortcut to 'sudo ufw allow 80/tcp'
# This will allow incoming TCP (and consequently http) traffic on port 80 
sudo ufw allow http
# Shortcut to 'sudo ufw allow 443/tcp'
# This will allow incoming TCP (and consequently http) traffic on port 443
sudo ufw allow https

# you told the firewall some rules but you need to tell it to start following all the rules it now has (think of it as a refresh)
sudo ufw enable
```

You can check what is being allowed on your firewall at any time by running: 

``` bash
 ufw status
```



## Setting Up Your Domain & Sub Domains

By now, you probably have you web app working fine when you enter your server's IP in your browser. But people won't ever get to that application with an IP.  Your client is eager to type his new business' domain and see his brand new site. So how can I make the internet map a domain name to my application ? 

### Buying a Domain

First off, you need to own the domain. You can buy a domain from countless sites like [GoDaddy](https://ae.godaddy.com/domains/domain-name-search?isc=jodomUSD1&currencyType=USD&gclid=CjwKCAjwvtX0BRAFEiwAGWJyZPSiV4wdGXcWJutsDzidUV7zfrqDU_5qbi44i4juet6l-99zKK8LexoCIkMQAvD_BwE&gclsrc=aw.ds) 

### Configuring Name Servers

At some point, you want to tell GoDaddy that I want any requests to this domain to go to my cloud provider where my virtual machine exists. To do that, GoDaddy will allow you to edit your domain's **name servers**. Usually cloud providers provide 3 name servers you need to map your domains to, in the case for digital ocean, these name servers are:

- ns1.digitalocean.com
- ns2.digitalocean.com
- ns3.digitalocean.com

### Registering Your Domain to Your Cloud Provider

So now, requests to our domain will be forwarded to Digital Ocean. But Digital Ocean has no clue its our domain, so we need to register it in from our [Dashboard](https://cloud.digitalocean.com/). Go to Networking > Domains and add a new domain. By default you will notice they already have 3 NS records. 

### Adding Records for the Domain & Subdomain

So now Digital Ocean knows to wait for any calls to your domain. But where should it send these calls to ? The next step is to add some records to point that out.

- Add an **A Record**, Hostname = somesite.com, Value = YourServersIPAddress

  *This route is for our web project. This will tell Digital Ocean that any web request to this domain should be forwarded to your virtual machine. After it gets to your virtual machine, Nginx was configured to handle it from that point and serve our React Website*

- Add another A Record, Hostname = api.somesite.com, Value = YourServersIPAddress

  *This route is for our API project.* Notice how adding a subdomain for our API project is as simple as just adding an **A Record**.

- Add a **CNAME Record**, Hostname = www.somesite.com, Value = somesite.com

  *This will tell Digital Ocean that www.somesite.com is the same as somesite.com*

  

  **Note**: Notice how both somesite.com & api.somesite.com point to the same IP yet end up referring to different projects. Nginx was able to handle that because of the ```server_name``` directive in both ```.conf``` files we wrote for our projects. server block that had ```server_name api.somesite.com``` had all our API project root and info. Similarly, the server block that had ```server_name somesite.com``` had our React project's root and info. 

Now give it some time and the mapping should take effect in up to 15 minutes. 