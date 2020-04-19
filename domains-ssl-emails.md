# Domains, Subdomains, SSL & Emails

## Setting Up Your Domain

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

- Add a **CNAME Record**, Hostname = www.somesite.com, Value = somesite.com

  *This will tell Digital Ocean that www.somesite.com is the same as somesite.com*

  

## Setting Up Subdomains

Add another A Record, Hostname = api.somesite.com, Value = YourServersIPAddress

*This route is for our API project.* Notice how adding a subdomain for our API project is as simple as just adding an **A Record**.

**Note**: Notice how both somesite.com & api.somesite.com point to the same IP yet end up referring to different projects. Nginx was able to handle that because of the ```server_name``` directive in both ```.conf``` files we wrote for our projects. server block that had ```server_name api.somesite.com``` had all our API project root and info. Similarly, the server block that had ```server_name somesite.com``` had our React project's root and info. 

Now give it some time and the mapping should take effect in up to 15 minutes. 



## Setting Up SSL (https)

// write certbot stuff here



## Setting Up Domain Emails

// figure it out and note it down here

