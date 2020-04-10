This document explains the steps of configuring an Ubuntu Linux 18.04 Server into a php enabled web server & getting your web application online. 

I will be deploying 2 seperate projects : 
- **Frontend:** React JS Website
- **Backend:** Laravel API

In most cases, what you will have after following this guide should work for most other solutions / frameworks. If not, you will have a good understanding to enable you to do whatever small change is needed to get it working.

## Introduction

Usually, my job finishes as soon as I push my final git commit ***(if you are working on a real life project without version control, please consider using git to avoid unnecessary headaches :)***. I never had to deploy projects myself. You might be just like me in that regard. At some point, you figured out that the web application you made in a month was sold for four times your salary. You got pissed, started looking for freelancing oppurtunities to get paid decently for once, you got a project, congratulations ! you worked a bit on it and want to show your client what you have so far, well fuck... you have no idea how to get it online do you? Devops has always been a mystery to you and the fact your application is in PHP is not making it any easier. You sure as hell are not paying for those quick deploy apps like [**Laravel Forge**](https://forge.laravel.com/).

Here you will find everything you need to know about how applications work on the internet and how you can get your application online yourself.

## Get a Server

First thing's first, so far you have been seeing your application on your computer. you have some kind of web server installed locally like wamp, xamp, mamp or maybe even apache2 or nginx. Your computer is acting as the host machine, with a web server that makes your computer listen on port 80 so when you type ```localhost``` in the browser, that web server will serve the files in your server's root html directory. (/var/www/html, C://wamp/www/, C://xamp/htdocs/, etc..). 

Great, so all you need is to hook up your computer to the internet right ? well you can do that, but that won't really workout for you. 

### Why you can't rely on your computer to host your website

I am sure you have internet at home / your office. for that to work your internet provider gave your router a public IP address so it can communicate with everyone else. you can see how the rest of the world sees you by checking your network's public IP [here](https://whatismyipaddress.com/). so what's the problem ? 

- notice how I said *your **network's** public ip*, **NOT** you computer's. yes, all computers / phones on your network will share that same public IP address. So that IP address is not really mapped to your host machine specifically which means the rest of the world wont be able to reach your computer by that IP. (you can configure the router depending on how advanced it is to forward messages sent to port 80 to your computer but because of the other issues I will list below it's not really gonna help).

- Remember that IP address you just saw [here](https://whatismyipaddress.com/), It actually changes. Visit that website in another time and you will see another public IP address. The one your provider gives your router is temprary and variable. At somepoint you want to map your website's domain (somesite.com) to an IP address where it can find your web server, but if that IP keeps changing, your domain - IP mapping will break the second your public IP changes. That is why you need a **static ip** which is a public IP address your provider gives you that will never change. Why don't you just ask him to give you one ? well because not all Internet providers offer this and it costs a lot of money and you just cheaped out on using quick deploy apps so I'm sure you can't afford it.

- Mirculously, none of the issues above apply to you, what if your computer turns off ? your web application will be down, can you guarantee your computer will never turn off or break ? 

- It's not scalable. Say you have a really good computer, but your web application is gaining publicity and you are getting millions of users in traffic at once, your computer will be phyisically unable to handle that kind of load, and it will take you way too long to but a new CPU, biiger RAMs, more storage, assemble them ... 

So I think we established your computer won't do it. What you need is a server provided by a good cloud platform like Google Cloud, AWS, Digital Ocean, etc... 

### Benefits of using a cloud platform

- Assign a static ip, exclusively for your server or VPC for next to no cost.
- Google / AWS / Digital Ocean server machines don't simply get turned off unexpectedly.
- If you need to scale it up and add more specs, you can do it with the press of a button with very little down time.
- Renting such a server will cost you very little *(cost varies across different cloud platforms, usually Google Cloud is expensive, Digital ocean is more affordable, starting $5 / month)*
- 
## Setting up your cloud platform

I will be using Digital Ocean in this guide.

// Talk about setup here
  
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
