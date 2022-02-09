# Buy A Domain Name For Your Deployed Flask Application

During this tutorial, I will show you how you can purchase a domain name for your deployed application. A domain name makes it easy for users to remember your site rather than an IP (Internet Protocol) address. For your reference, these are the topics in our discussion:

1. [Deploy your flask app on Linode](/linode/deploy_on_linode.md)
2. [Buy a domain name for your deployed application](/linode/buy_domain.md)
3. [Secure your domain with SSL](/linode/secure_domain_with_ssl.md)

## Table of Contents

If you would like to skip to a particular section within this tutorial, you can click on any of these links below:

- [Register a domain name](#register-a-domain-name)
- [Connect domain name to your application](#connect-domain-name-to-your-application)

## Register a Domain Name

The very first thing I will need to do is to register a domain name from a domain registrar. This will cost me a small amount of money, usually about $12 per year. There are several domain registrars, for example, [Google Domains](https://domains.google/) and [Godaddy](https://www.godaddy.com/en-uk). The registrar that I will use is [Namecheap](https://linode.gvw92c.net/15oBBg).

![Namecheap](/images/linode/buy_domain/namecheap.png)

In the search bar, find yourself a domain name that you want to register. You may find some domain names already taken, so be sure to check the availability before you buy. It is easier to simply get a new domain name rather than bid for one that is already taken. 

I will search for _bolderlearner.com_. If it is available, I will add it to my cart. You will find the name with possible other extensions such as _.net_ or _.academy_. Pick the one that is most appropriate for your application. Namecheap normally offers monthly discounts when you register a domain name. Be sure to check that out and take advantage of the discount. 

Once the name is in my cart, I will click in the checkout button to begin the process of confirming my order. I will register this domain for one year. You can choose to register it for longer periods of time by clicking the dropdown menu and selecting the number of years you would like to register the domain for.

By default, domain privacy is enabled just to protect you and your information when you register a domain. You can disable this by clicking the privacy buttons, but I don't know why you'd want to do that. There is a linux terminal command that you can use to find out more information about a domain name. So, for example, if I want to find out more information about _bolderlearner.com_, I can run the command `whois bolderlearner.com`. This simple command will reveal all the information about the domain name. If your domain privacy is disabled, all your personal details will be visible to anyone who runs this command. However, since the domain privacy is enabled by default, I am not be concerned about this since everything about me is blocked.

Namecheap also offers other addons such as SSL, VPN, Professional Email, and more. I will not be using any of these addons because I can do them myself. That is all I need to register my domain name. I will proceed to confirm my order. If you are interested in discounts, you will see a Promo Code field. Enter the promo code on offer during the month of your purchase, click "Apply" before clicking "Confirm Order". Since I am not a first time registrant, the coupon shown below will be invalid.

![Purchase domain](/images/linode/buy_domain/purchase.png)

I will be redirected to the billing page where I can enter my credit card information. For this domain, I will disable autorenewal for my purchase. Once that is done, I will click "Continue". Namecheap will then redirect me to a confirmation page just to make sure that the details I have entered are correct. If everything looks good, I will confirm this purchase. 

Back to my namecheap dashboard, I can click the [Domain List](https://ap.www.namecheap.com/domains/list/) link in the sidebar navigational menu to see the domain I just purchased. There is a "Manage" button next the domain. Clicking this button will redirect me to the domain management page. 

![Domain management](/images/linode/buy_domain/domain_management.png)

## Connect Domain Name to Your Application

On my Linode dashboard, there is a "Domains" link in the sidebar navigational menu. Clicking this link will show me a list of all the domains I have registered. You can see that I currently have one domain registered.

![Domain list](/images/linode/buy_domain/domain_list.png)

To add a new domain, I will make use of the [DNS Manager Documentation](https://www.linode.com/docs/guides/dns-manager/). This documenation can be found on top of your domains list, clearly labeled as _Docs_. Clicking this link should take you to the DNS Manager documentation.

### DNS Set-Up Checklist

1. [Register a domain name](#register-a-domain-name)
2. [Set your domain name to use Linode's nameservers](#set-your-domain-name-to-use-linode-s-nameservers)
3. [Use the DNS Manager to add a new record to your domain](#use-the-dns-manager-to-add-a-new-record-to-your-domain)
4. [Set up revers DNS](#set-up-revers-dns)
5. [Add additional DNS records](#add-additional-dns-records)

### Set Your Domain Name to Use Linode's Nameservers

The five entries below are the nameservers that Linode uses. I will need to add these to my domain name in Namecheap.

- `ns1.linode.com`
- `ns2.linode.com`
- `ns3.linode.com`
- `ns4.linode.com`
- `ns5.linode.com`

On my domain management page, there is a section called "NAMESERVERS". By default, the nameservers are set to use _Namecheap BasicDNS_. I will click on the dropdown menu and select "Custom DNS". This will allow me to enter Linode's nameservers.

![Custom DNS](/images/linode/buy_domain/custom_dns.png)

I will paste the nameservers into the text boxes then click on the :heavy_check_mark: to save the changes.

![Save nameserver changes](/images/linode/buy_domain/save_nameserver_changes.png)

Both Linode and Namecheap say that the changes I have made will take some time, usually 24 - 48 hours, to take effect. In computer jargon, this is known as a "DNS propagation delay".

### Use the DNS Manager to Add a New Record to Your Domain

Within my domain list in Linode's dashboard, there is a button called "Create Domain". 

![Create domain](/images/linode/buy_domain/create_domain.png)

To add my new domain name, I will click on this button.

![Create domain in linode](/images/linode/buy_domain/create_domain_in_linode.png)

I have added a temporary email _admin@bolderlearner.com_ which does not exist yet. "Create Domain" button will create this new domain and redirect me to Linode's domain management page.

![Linode domain management](/images/linode/buy_domain/linode_domain_management.png)

At first, this might look very intimidating, but it is actually very simple. What I am interested in here is to add some DNS records to my domain. Example of DNS records are A records, CNAME records, MX records, and TXT records.

The first record that I am going to add is an A record. This record is used to point my domain name to a specific IP address. I will use the IP address of my Linode. This record requires the following fields:

- `Hostname`: This is the domain name that I want to point to my IP address.
- `IP Address`: This is the IP address that I want to point to my domain name.
- `TTL`: This is the time to live of the record. It sets the lifespan of the cache for the DNS record

I will click on "Add an A/AAAA Record" button to update my hostname, IP address, and TTL.

![A record](/images/linode/buy_domain/a_record.png)

I have added my host name as `www`, my IP address as 212.71.247.241 and I left TTL as it was, that is "Default". The IP address that I am using currently is different from the one you saw in [the previous tutorial](deploy_on_linode.md). This is because I recreated the entire project and obtained a new IP address. Moving forward, I will be using this new IP address.