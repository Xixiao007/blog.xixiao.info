---
title: "Dockerized Nginx RP with Letsencrypt HTTPS"
layout: post
comments: true
---

<p class="message">
This is part 4 of 5 in the series: <a href="/2016/01/01/setup-docker-container-services">Setup Docker Container Services</a>
</p>
The entire series are:

0. [Dockerize Multiple Services in Ubuntu14.04 Server]({% post_url 2016-01-01-setup-docker-container-services %})
1. [ddclient - DNS Update for Multiple Domains]({% post_url 2016-01-12-dynamic-dns-setup-by-ddclient %})
2. OpenVPN Server - *`coming soon!`*
3. [Dockerized Nginx RP with Letsencrypt HTTPS]({% post_url 2016-01-05-dockerized-nginx-rp-with-letsencrypt-https %})
4. [Nginx Docker - Hosting Multi-domain Static Websites]({% post_url 2016-01-20-nginx-docker-hosting-multi-domain-static-websites %})

Introduction
-------

Reverse Proxy (RP) is a type of proxy server that represents other services behind the scene and sends back resources to client as though they were originated from the proxy itself. Jason Wilder blogged [why automated reverse proxy is beneficial in dockerized environment](http://jasonwilder.com/blog/2014/03/25/automated-nginx-reverse-proxy-for-docker/) as well as created a [nginx reverse proxy docker image](https://github.com/jwilder/nginx-proxy). This docker image is able to, on the fly by nginx service reloading, add other popping up web server containers into its cover or remove them when they go offline.

[Let's Encrypt](https://letsencrypt.org/) is a free, automated, and open certificate authority. It offers free-of-charge TLS/SSL certificates for websites to enable encrypted HTTPS traffic.

In this article, we will go through:

1. how to download letsencrypt certificates for websites to be hosted in the very same server
2. how to setup a dockerized RP service and enable HTTPS feature

Prerequisites
-------

Before moving forward, I assume you have:

1. An Ubuntu14.04 (or above) server in place with public IP address available. Other linux flavors, such as CentOS, should go just fine as well
2. Docker application installed. If not, follow [Install Docker](https://docs.docker.com/engine/installation/) 
3. Git installed
4. Owned DNS domains that point to this server's public IP address. In my case, they are *dockertechie.com*, *xixiao.info*, and *blog.xixiao.info*


Retrieve Letsencrypt TSL/SSL certificates
-------

Letsencrypt has a fully-featured, automated [client](https://github.com/letsencrypt/letsencrypt) in github. We can run the commands as below to retrieve certificates for our websites. In my scenario, I ran these commands to obtain certificates:

{% highlight bash linenos%}
$ git clone https://github.com/letsencrypt/letsencrypt
$ cd letsencrypt
$ ./letsencrypt-auto certonly --standalone --email youremailaddress@something.com -d yourowndomain 
{% endhighlight %}

- Note, at the time of writing, letsencrypt standalone method uses **port 80** to engage the traffic with authority server, thus make sure that the port 80 is free when running the #3 command.
- According to the letsencrypt client [instruction](https://github.com/letsencrypt/letsencrypt#how-to-run-the-client), #3 command supports more than one `-d` parameter against multiple domains. But in my try, it only downloads certificates for the first served domain and ignored subsequent ones. Therefore, I have to run #3 multiple times in order to retrieve for all my domains. 

{% highlight bash %}
IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at
   /etc/letsencrypt/live/xixiao.info/fullchain.pem. Your cert will
   expire on 2016-04-04. To obtain a new version of the certificate in
   the future, simply run Let's Encrypt again.
 - If you like Let's Encrypt, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le
{% endhighlight %}

You will eventually reach this message above if the client runs successfully. Be aware that these certificates expires after **3 months**, so you will need to renew certificates within this time period. If you check the `/etc/letsencrypt` folder structure, you will notice that 

1. all certificates are stored at path `/etc/letsencrypt/archive/yourdomain` with soft links from `/etc/letsencrypt/live/yourdomain`
2. each domain will have 4 PEM-encoded files:

- cert1.pem: domain's certificate
- chain1.pem: letsencrypt chain certificate
- fullchain1.pem: contains both `cert.pem` and `chain.pem`
- privkey1.pem: certificate's private key

Finally, we need to rename `fullchain1.pem` to `yourfulldomain.crt` and `privkey1.pem` to `yourfulldomain.key` and copy these 2 out of 4 files to a prepared folder in order to feed them into RP in later step. In my case, I have certificates for 3 domains stored in `~/nginx-reverse-cert` folder:

{% highlight bash %}
finxxi@Fiend:~/nginx-reverse-cert > ls
blog.xixiao.info.crt  dockertechie.com.crt  xixiao.info.crt
blog.xixiao.info.key  dockertechie.com.key  xixiao.info.key
{% endhighlight %}


Pull and Run Dockerized Nginx Reverse Proxy 
------

Jason has made his RP docker image pretty easy to use. The command below will pull the docker image if not exists yet and run it:

{% highlight bash %}
$ docker run -p 80:80 -p 443:443 \
-v ssl-tsl-certificates-path:/etc/nginx/certs \
-v /var/run/docker.sock:/tmp/docker.sock:ro \
--rm --name proxy jwilder/nginx-proxy
{% endhighlight %}

- You should replace `ssl-tsl-certificates-path` with the location where letsencrypt SSL/TSL certificates were previously stored. In my case, it is `~/nginx-reverse-cert`
- port 443 is open along with port 80 in host machine for the purpose of HTTPS communication
- `--rm` is used so that when the docker container stops the container itself is purged automatically. The purpose is to avoid unused long list of containers sucking up disk spaces
- Do not understand this command? `docker run` [help page](https://docs.docker.com/engine/reference/run/) is your friend

Then we need to make sure our container is up and running by`docker ps`. You should see similar output:

{% highlight bash %}
$ docker ps
CONTAINER ID        IMAGE                 COMMAND                  CREATED             STATUS              PORTS                                      NAMES
800beee14cb5        jwilder/nginx-proxy   "/app/docker-entrypoi"   16 hours ago        Up 16 hours         0.0.0.0:80->80/tcp, 0.0.0.0:443->443/tcp   proxy
{% endhighlight %}


Auto Start RP Proxy when System is Up
-------

Once we ensured that RP docker containing is able to run correctly, we can add it into system upstart init daemon job as follow.

Firstly, we use `sudo` permission to create a `proxy.conf` file in `/etc/init` path with the content:

{% highlight conf %}
description "Docker container for nginx proxy server"
start on filesystem and started docker
stop on runlevel [!2345]
respawn
script
  exec docker run -p 80:80 -p 443:443 -v ssl-tsl-certificates-path:/etc/nginx/certs -v /var/run/docker.sock:/tmp/docker.sock:ro --rm --name proxy jwilder/nginx-proxy
end script
{% endhighlight %}

Note. remember to alter `ssl-tsl-certificates-path` to the letsencrypt certificate path.

Then we use #1 `ln -s` command to add a soft link into `/etc/init.d`, use #2 `service`command to kick off the service (note. make sure RP container was not already running by checking `docker ps`).
{% highlight bash linenos%}
ln -s /etc/init/proxy.conf /etc/init.d/proxy
sudo service proxy start
{% endhighlight %}

Now if we run `docker ps` should display RP container is up. We can `docker stop` the container or reboot the system, and expect that the container will come back automatically.


Conclusion
-------

The nginx RP docker container is now up and listening to port 80 and 443 with free Letsencrypt TLS/SSL certificates installed! 

It is ready to securely serve contents on behalf of web domains that has certificates fed. The subsequent step is to use whatever web server docker container to attach to this RP. In my upcoming blog thread, I will introduce how to use offcial nginx docker container to host my websites: *dockertechie.com*, *xixiao.info*, and *blog.xixiao.info*.
