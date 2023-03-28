---
layout: post
title: "Pi-Hole for DNS and Ad-Block - Simple and Easy"
date: 2023-03-10 09:00:00 +0200
categories: [dns,ad-block]
tags: [homelab,dns,ad-block,pi-hole]
---

Pi-Hole is a wonderful DNS sever with ad blocking for your network. Also we can use it as a Local DNS server (This would be another future document post).

All is about setting up Pi-Hole as our personal and private DNS server, which we gonna host ourself.

 ## What actually Pi-Hole is ?

It is a DNS server that we could host ourself that also also blocks ads when we attempt to visit webpages. In its standard configuration Pi-Hole is what is known as a forwarding DNS server, in that it only has a very specific list of websites that is has the IP address for resolution. An if its does not have that address it will foward the packet on to the next DNS provider that we have confgiured.\
So if we type in the webpage dafont.com into web browser that request is forwarded onto our DNS server, Pi-Hole. Since DNS server does not know where dafont.com, webpage IP address, is and if it is not an ad serving website it will forward that request onto the next DNS server that we have configured. That DNS server, in our case cloudflare - 1.1.1.1, 1.0.0.1 -, will then forward back down the IP address for webpage dafont.com through Pi-Hole and into our PC.\
However whilw dafont.com is loading it also wants to load up a whole bunch of websites that contain ads, when those request go to Pi-Hole it is int the adblock list and so they are filtered out.

  
> **Note**: By default Pi-Hole runs great for its advertised features. However there is a lot more power under the hood with a little bit of tweaking and that is called recursice dns. That is essentially what we forward in your requests form Pi-Hole to such as Google and OpenDNS. Whem we ask Pi-Hole where is dafont.com if it does not know the IP address it will actually seek out what is called the authoritative domain server of dafont.com and get the information from them directly. On the very first request of a website this will take a little bit longer than usual although Pi-Hole will cache that information for future use.
{: .prompt-info }

## Bulding our server

So let's go ahead and get to bulding our new Pi-Hole server.
First up we are going to create a new virtual machine, in my case i use Linux Container. However you can run this on a raspberry pi.

Generally create your virtual machine of your choice, any linux distro, and then go on with pi-hole installation https://github.com/pi-hole/pi-hole/#one-step-automated-install

### Hardware
Pi-hole is very lightweight and does not require much processing power

* CPU: 1 vCPU Minimum / 2 vCPU Recommended
* Memory: 512MB RAM
* Hard Disk: 2GB Minimum, 4GB Recommended

## Pi-Hole installation

After installing your Linux VM, connect via ssh to your server and execute the follwing script 

```shell
 sudo curl -sSL https://install.pi-hole.net | bash
```
* A wizard will show up with a few questions, we follow the default one settings.\
Hit ENTER
* Choose your DNS provider.\
OpenDNS is fine, but you can choose another one if you prefer.
* Pi-hole relies on third-party lists to block ads.\
By default, it will use StevenBlack’s host list, but you can deselect it if you want (not recommended). Laterlly we add more ad-lists.
Hit Enter to continue, and the space bar to select/deselect.
* Then, select the protocols you want to use (recommended keep both).
* Pi-Hole will use a static IP address, in order to avoid reconfiguring everything each time it uses a new IP.\
The next step is to configure if you want to keep the current IP or set another one manually. 
* Then confirm to enable the web interface and the web server PHP (highly recommended).
* Confirm if we want to log querries.\
Depends if you want or not for your privacy aspect. Then there are four privacy mode. Show everythong is fine, but you can choose everyone you prefer.
* Finally after a couple of minutes Pi-Hole server should be completed installed and we can access it up of web browser.

Below provided login details to access the Admin web page. We will be provided with a link to the site and a password, which is recommened to change.

![](/template/images/pi-hole-3.JPG)

Set the Web Admin Password 
```shell 
pihole -a -p [password]
```

## Admin Web Page

Navigate to web page

![](/template/images/pi-hole-1.JPG)

Enter your password and you have a dashboard like that

![](/template/images/pi-hole-2.JPG)

## Ad-block Lists
In order to secure more your network from anoing ads you could add addiotioanl ad-list.

![](/template/images/pi-hole-4.JPG)

Prefered lists from Firebog.net https://firebog.net/

## LAN settings

In order to use the new server and take advantage of the built-in ad-block you will need to change your network settings either in dhcp to mirror the new DNS server or change your computer's network settings.

If you have more than one network (VLANs) in your network, then you will need an additional option in the server settings.\
**Settings -> DNS** and at the **interface settings** section choose **_Permit all origins_**

## Local DNS
In this fast, simple, and easy guide we’ll walk through how to create DNS Entries (A Records) for the clients on your network and also set up Aliases (pointers to A Records) so that you can start using DNS at home instead of relying on IP addresses.

This will be future doc article