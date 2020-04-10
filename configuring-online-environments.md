This document explains the steps of configuring an Ubuntu Linux 18.04 Server into a php enabled web server & getting your web application online. 

I will be deploying 2 seperate projects : 
- **Frontend:** React JS Website
- **Backend:** Laravel API

In most cases, what you will have after following this guide should work for most other solutions / frameworks. If not, you will have a good understanding to enable you to do whatever small change is needed to get it working.

## Introduction

Usually, my job finishes as soon as I push my final git commit ***(if you are working on a real life project without version control, please consider using git to avoid unnecessary headaches :)***. I never had to deploy projects myself. You might be just like me in that regard. At some point, you figured out that the web application you made in a month was sold for four times your salary. You got pissed, started looking for freelancing oppurtunities to get paid decently for once, you got a project, congratulations ! you worked a bit on it and want to show your client what you have so far, well fuck... you have no idea how to get it online do you? Devops has always been a mystery to you and the fact your application is in PHP is not making it any easier. You sure as hell are not paying for those quick deploy apps like [**Laravel Forge**](https://forge.laravel.com/).

## Get a Server

// insert why and how here

## Install and Configure Nginx

### Update your system


``` bash
sudo apt-get update
sudo apt-get upgrade
```

### Install Nginx

``` bash
sudo apt-get install nginx
```

### Configure Nginx

You installed a web server, all you need to do 


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
