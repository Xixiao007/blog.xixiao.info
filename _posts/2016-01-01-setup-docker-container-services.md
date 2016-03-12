---
title: "Dockerize Multiple Services in Ubuntu14.04 Server"
layout: post
tags: basic
comments: true
---


<p class="message">
This is part 1 of 5 in the series: <a href="/2016/01/01/setup-docker-container-services">Setup Docker Container Services</a>
</p>
The entire series are:

0. [Dockerize Multiple Services in Ubuntu14.04 Server]({% post_url 2016-01-01-setup-docker-container-services %})
1. [ddclient - DNS Update for Multiple Domains]({% post_url 2016-01-12-dynamic-dns-setup-by-ddclient %})
2. OpenVPN Server - *`coming soon!`*
3. [Dockerized Nginx RP with Letsencrypt HTTPS]({% post_url 2016-01-05-dockerized-nginx-rp-with-letsencrypt-https %})
4. [Nginx Docker - Hosting Multi-domain Static Websites]({% post_url 2016-01-20-nginx-docker-hosting-multi-domain-static-websites %})

[Docker](https://www.docker.com/what-docker) is the main theme of this blog. It is the next generation cutting-edge hypervisor technology with many incredible powerful features. Among those features, *lightweight size*, *fast boot-up time*, and *isolation from host*, are key reasons for me to utilize docker in my personal Ubuntu server.

The verb `Dockerize` means to put/wrap something, such as service or application, into a docker container. I will go through the setups from scratch for dockerizing three common services: **vpn server**, **reverse proxy**, and **web server with multiple vhost** in one Ubuntu server in following threads.


![my docker microservices](//i.imgur.com/HoeXTwn.png "my docker microservices")

The final topology diagram is showed as above. We will eventually have:

- a OpenVPN server container to serve VPN services.
- a nginx reverse proxy container listening to port 80 and 443
- a nginx web server hosting 3 domain website: *https://xixiao.info, https://blog.xixiao.info, https://dockertechie.com*


