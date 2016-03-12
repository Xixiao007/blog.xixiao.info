---
title: "ddclient - DNS Update for Multiple Domains"
layout: post
tags: ddclient, DynamicDNS
comments: true
---


<p class="message">
This is part 2 of 5 in the series: <a href="/2016/01/01/setup-docker-container-services">Setup Docker Container Services</a>
</p>
The entire series are:

0. [Dockerize Multiple Services in Ubuntu14.04 Server]({% post_url 2016-01-01-setup-docker-container-services %})
1. [ddclient - DNS Update for Multiple Domains]({% post_url 2016-01-12-dynamic-dns-setup-by-ddclient %})
2. OpenVPN Server - *`coming soon!`*
3. [Dockerized Nginx RP with Letsencrypt HTTPS]({% post_url 2016-01-05-dockerized-nginx-rp-with-letsencrypt-https %})
4. [Nginx Docker - Hosting Multi-domain Static Websites]({% post_url 2016-01-20-nginx-docker-hosting-multi-domain-static-websites %})

Introduction
-------

Many cloud service providers, such as Azure, provision virtual machines with dynamic IP addresses by default. This means that the server would possibly occupy different IP address after rebooting. In this scenario, how to associate domain names with constantly changing server IP is a must-to-solve problem. `DynamicDNS update` could help tackle this problem. In this article, we will use `ddclient`, a lightweight linux utility, to enable dynamicDNS update for two root domains,*xixiao.info and dockertechie.com*, in my Ubuntu14.04 server. 

Prerequisites
-------
- Ubuntu14.04 or higher server edition. 
- sudo permission
- owning at least one domain name (I have my two domains purchased from [namecheap](www.namecheap.com))


ddclient Installation and Configuration
-------


- **Install ddclient**

{% highlight bash %}
sudo apt-get update && sudo apt-get install ddclient
{% endhighlight  %}

<p class="message">
Note. ddclient, with version older than or equal to 3.8.2, is unable to update multiple namecheap domains. That is why we take following steps to hack its binary file. If you have only one root domain to update, please ignore the next step (patch ddclient) and continue from subsequent step (manipulate ddclient.conf file.
</p>

-  **Patch ddclient to support multi-domain dynamic update**

Stop ddclient service

{% highlight bash %}
sudo service ddclient stop
{% endhighlight  %}

Copy out the existing ddclient binary to a new path and enter the new path

{% highlight bash %}
mkdir ~/patch-purpose
cp /usr/sbin/ddclient ~/patch-purpose
cd ~/patch-purpose
{% endhighlight  %}

Save the following entire content to a text file and name it `ddclient.patch` under same path `~/patch-purpose`
{% highlight conf %}
--- ddclient    2012-08-22 18:49:15.643067531 -0500
+++ ddclient    2012-08-22 20:32:23.671069165 -0500
@@ -3375,8 +3375,11 @@

         my $url;
         $url   = "http://$config{$h}{'server'}/update";
-        $url  .= "?host=$h";
-        $url  .= "&domain=$config{$h}{'login'}";
+        my $domain = $config{$h}{'login'};
+        my $host = $h;
+        $host  =~ s/(.*)\.$domain(.*)/$1$2/;
+        $url  .= "?host=$host";
+        $url  .= "&domain=$domain";
         $url  .= "&password=$config{$h}{'password'}";
         $url  .= "&ip=";
         $url  .= $ip if $ip;
{% endhighlight  %}

then we apply the patch to the copied ddclient binary and copy the newly patched binary back

{% highlight bash %}
patch -b -p0 -i ddclient.patch
patching file ddclient
cp ~/patch-purpose /usr/sbin
{% endhighlight  %}

- **Manipulate ddclient.conf file**

If you have only one domain to update, here is a sample conf file content at `/etc/ddclient.conf`. It will automatically update the *http://xixiao.info* root domain.

{% highlight conf %}
daemon=3000
use=web, web=dynamicdns.park-your-domain.com/getip
protocol=namecheap
ssl=yes
server=dynamicdns.park-your-domain.com
login=xixiao.info
password=your-dynamic-dns-password
@
{% endhighlight  %}

If you have multiple domains, like me, we should modify ddclient configuration file at `/etc/ddclient.conf` and save the following content into it in order to update *http://xixiao.info* and *http://dockertechie.com*. 
Note. You need to modify the domain names to meet your own purpose. If your domain provider is other than namecheap, you may also need to tinker this file content.

{% highlight conf %}
daemon=3000
use=web, web=dynamicdns.park-your-domain.com/getip
protocol=namecheap
ssl=yes
server=dynamicdns.park-your-domain.com
login=xixiao.info
password=your-dynamic-dns-password
@.xixiao.info, www.xixiao.info, blog.xixiao.info, vpn.xixiao.info

protocol=namecheap
server=dynamicdns.park-your-domain.com
login=dockertechie.com
password=your-dynamic-dns-password2
@.dockertechie.com

{% endhighlight  %}

Note. Change the `your-dynamic-dns-password` to the password string offered by domain service provider. You should be given the password string when enabling dynamic DNS setting in your domain admin page.

- **Auto-start ddclient and final test**

One more important step is to set ddclient daemon up so it will be started when system boots up. This can be ensured by verifying `run_daemon="true"` is set in `/etc/default/ddclient`.

Now we can start ddclient by `sudo service ddclient start` and verify ddclient is up and running correctly by checking `ps -aux | grep ddclient`

The very last step is to test that the domain name is automatically linked to the new IP address of server whenever it changes.


Conclusion
------
With the help of ddclient tool, we now have bundled our multiple domains to the dynamic IP address of the Ubuntu server. In my case, root domain and various subdomains of *http://xixiao.info* as well as the root domain of *http://dockertechie.com* is linked to my Ubuntu server.
