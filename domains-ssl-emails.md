# Domains, SSL & Emails

## Setting Up Your Domain

By now, you probably have you web app working fine when you enter your server's IP in your browser. But people won't ever get to that application with an IP.  Your client is eager to type his new business' domain and see his brand new site. So how can I make the internet map a domain name to my application ? 

### Buying a Domain

First off, you need to own the domain. You can buy a domain from countless sites like [GoDaddy](https://ae.godaddy.com/domains/domain-name-search?isc=jodomUSD1&currencyType=USD&gclid=CjwKCAjwvtX0BRAFEiwAGWJyZPSiV4wdGXcWJutsDzidUV7zfrqDU_5qbi44i4juet6l-99zKK8LexoCIkMQAvD_BwE&gclsrc=aw.ds) 

### Registering Your Domain to Your Cloud Provider

Digital Ocean still has no clue we have a domain, so we need to register it in from our [Dashboard](https://cloud.digitalocean.com/). Go to Networking > Domains and add a new domain. 

### Adding Records for the Domains / Subdomains

What you need to know to play around with linking domains & subdomains to servers is that when you buy a domain, you can set certain rules & mappings to it for the rest of the internet to acknowledge. These *rules* are called records, and these are the type of records you should concern yourself with to setup your domain.

| Record Type | Record Value         | Description                                               |
| ----------- | -------------------- | --------------------------------------------------------- |
| A           | an IP address        | an IP address your domain should redirect to              |
| CNAME       | a domain / subdomain | another domain / subdomain your domain should redirect to |
| NS          | a domain             | The domain of the name server of your cloud provider.     |



So what we need to do is the following 

- Tell whoever we baught our domain from to forward the traffic to our cloud provider by adding NS records that point to our cloud provider's name servers. In our case we need 3 NS records pointint to 
  - <yourdomain.com> -> <ns1.digitalocean.com>
  - <yourdomain.com> -> <ns2.digitalocean.com>
  - <yourdomain.com> -> <ns3.digitalocean.com>
- Tell our cloud provider to forward traffic to our domain to our specific virtual machine by adding an A record. 
  - <domain-subdomain> -> <IP of your virtual machine>



After this your are basically done. However we can top it off by doing som extra mappings to be more professional like 

- Saying www.mydomain.com is the same as mydomain.com by adding a CNAME record.
  - <www.mydomain.com> -> is an alias of <mydomain.com>



Note that changin NS records are usually done from the place you purchased the domain while everything else is done from your cloud provider's dashboard.



## Adding SSL / HTTPS to Domains

To enable ssl / https you will need an ssl certificate installed on your virtual machine and read by your webserver as it listens to port 443. 

an ssl certificate is made up of a pair of public & private keys. You can get a certificate from multiple places, the most popular place is from [LetsEncrypt](https://letsencrypt.org/). 

You would be pleased to know this process is very easy as LetsEncrypt provides us with [certbot](https://certbot.eff.org/) that can automatically download and add the necessary configurations to our web server config files. All you need to do is ssh into your server and do the following

``` bash
# Add the certbot repository to your system
sudo apt-get update
sudo apt-get install software-properties-common
sudo add-apt-repository universe
sudo add-apt-repository ppa:certbot/certbot
sudo apt-get update

# Install certbot 
sudo apt-get install certbot python3-certbot-nginx
```

And to enable ssl on a certain domain (since your server might have several websites on it) you can simply run:

``` bash
sudo certbot certonly -d thedomain.com
```

if you don't specify the ``` --nginx``` flag it will ask you what webserver you want to use.

When it finishes you will see something like this:

``` bash
IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/yourdomain.com/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/yourdomain.com/privkey.pem
   Your cert will expire on 2020-10-23. To obtain a new or tweaked
   version of this certificate in the future, simply run certbot
   again. To non-interactively renew *all* of your certificates, run
   "certbot renew"
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le

```

Note the paths it gave you for the public and private keys for the domain you registered.



**Note:** By now it should have worked.  In the case where certbot failed to edit your nginx .conf file on its own, you can enable ssl from your .conf file manually by adding this piece of code in your server block:

``` conf
    listen 443 ssl; # managed by Certbot
    
    ssl_certificate path-to-fullchain.pem; # managed by Certbot
    
    ssl_certificate_key path-to-privkey.pem; # managed by Certbot
    
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot
```

Make sure to replace ``` path-to-fullchain.pem``` with the path to your public letsencrypt key and ``` path-to-privkey``` to your private letsencrypt key.



## Creating Domain Emails

