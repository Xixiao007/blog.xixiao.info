---
title: "Nginx Docker - Hosting Multi-domain Static Websites"
layout: post
comments: true
---


<p class="message">
This is part 5 of 5 in the series: <a href="/2016/01/01/setup-docker-container-services">Setup Docker Container Services</a>
</p>
The entire series are:

0. [Dockerize Multiple Services in Ubuntu14.04 Server]({% post_url 2016-01-01-setup-docker-container-services %})
1. [ddclient - DNS Update for Multiple Domains]({% post_url 2016-01-12-dynamic-dns-setup-by-ddclient %})
2. OpenVPN Server - *`coming soon!`*
3. [Dockerized Nginx RP with Letsencrypt HTTPS]({% post_url 2016-01-05-dockerized-nginx-rp-with-letsencrypt-https %})
4. [Nginx Docker - Hosting Multi-domain Static Websites]({% post_url 2016-01-20-nginx-docker-hosting-multi-domain-static-websites %})

Introduction
---------

I have the very need to host three static websites, including two Jekyll blogs and one personal mainpage. Originally I wanted the two Jekyll blogs running under [Jekyll official docker image](https://github.com/jekyll/docker), while the mainpage under another general web server. However, I realize that Jekyll docker image does not yet support multi-blog hosting. Moreover, it is also slightly lame to separate into multiple containers without good reason.

After some studies, I figured out that running one nginxi docker container can perfectly serve three webistes with different domain names.

Prerequisites
----------

I assume that you have 

- the multi-domain static website contents at hand. In my case, I have in prior compiled Jekyll blogs and output to static content by `jekyll build` command, i.e. http://blog.xixiao.info, http://dockertechie.com, http://xixiao.info contents are ready to go.
- Ubuntu 14.04 or above with docker installed


Main Course
-----------

In this section, we will elaborate how to set things up. You can find my up-to-date config files also **[here](https://github.com/Xixiao007/nginx-config)**.

1. Put the 3 website under different folders => `~/websites/xixiao.info` `~/websites/blog.xixiao.info` `~/websites/dockertechie.com` so that they are ready to serve as *data volumes*(https://docs.docker.com/engine/userguide/dockervolumes/) for nginx docker container

2. create `conf.d` nginx configuration file as well as individual configuration file per site. In my case, they are all saved under one folder as the structure below.
http://i.imgur.com/xH7gyFT.png
![folder structure](//i.imgur.com/xH7gyFT.png "my nginx config files")

My nginx-jekyll-docker.conf  
{% highlight conf %}
daemon off;
user nginx nginx;
error_log /var/log/nginx/error.log;
worker_rlimit_nofile 8192;
pid /var/run/nginx.pid;
worker_processes 4;

events {
  multi_accept on;
  worker_connections 2048;
  use epoll;
}

http {
  log_format docker '[$host]: $remote_addr - $remote_user [$time_local] '
    '"$request" $status $body_bytes_sent '
    '"$http_referer" "$http_user_agent"';
  access_log /var/log/nginx/access.log docker;
  default_type application/octet-stream;

  charset utf8;
  rewrite_log off;
  gzip_comp_level 9;
  include mime.types;
  keepalive_timeout 24;
  lingering_close on;
  lingering_time 24;
  gzip_vary on;
  gzip on;

  gzip_types text/xml;
  gzip_types text/javascript;
  gzip_types application/json;
  gzip_types application/x-javascript;
  gzip_types application/javascript;
  gzip_types application/x-font-ttf;
  gzip_types application/ttf;
  gzip_types image/x-icon;
  gzip_types text/plain;
  gzip_types text/css;
  gzip_types image/gif;
  gzip_types image/jpeg;
  gzip_types image/png;

  include /etc/nginx/conf.d/*.conf;
}
{% endhighlight %}

My blog.xixiao.info.conf
{% highlight conf %}
server {
  root /usr/share/nginx/blog.xixiao.info/;
  listen 80;
  server_name blog.xixiao.info;

  location / {
    etag off;
    expires off;
    try_files $uri $uri.html $uri/index.html =404;
  }
}
{% endhighlight %}

My xixiao.info.conf
{% highlight conf %}
server {
  root /usr/share/nginx/xixiao.info/;
  listen 80;
  server_name xixiao.info;

  location / {
    etag off;
    expires off;
    try_files $uri $uri.html $uri/index.html =404;
  }
}
{% endhighlight %}

My dockertechie.com.conf
{% highlight conf %}
server {
  root /usr/share/nginx/dockertechie.com/;
  listen 80;
  server_name dockertechie.com;

  location / {
    etag off;
    expires off;
    try_files $uri $uri.html $uri/index.html =404;
  }
}
{% endhighlight %}

Lastly, we need to start our nginx docker container to serve these sites. In my case, I have used *nginx reverse proxy* in [4th article of this series]({% post_url 2016-01-05-dockerized-nginx-rp-with-letsencrypt-https %}), so `-e VIRTUAL_HOST` variable is present as below.

{% highlight bash %}
  docker run --rm --name sites \
  -v /home/user/websites/dockertechie.com/_site:/usr/share/nginx/dockertechie.com:ro\
  -v /home/user/websites/blog.xixiao.info/_site:/usr/share/nginx/blog.xixiao.info:ro\
  -v /home/user/websites/xixiao.info:/usr/share/nginx/xixiao.info:ro\
  -v /home/user/websites/nginx-config/conf.d:/etc/nginx/conf.d:ro\
  -e VIRTUAL_HOST=dockertechie.com,blog.xixiao.info,xixiao.info nginx
{% endhighlight %}

If you do not use reverse proxy as I do, you can remove `-e VIRTUAL_HOST` parameter section and add `-p 80` to expose port 80 to external.


Conclusion
-----------------

It is very easy to host multiple static websites under nginx proxy. As we went through, we only need to serve the site content as data volume and craft a corresponding nginx config file.

Note that my up-to-date config files demonstrated in this article can be found **[here](https://github.com/Xixiao007/nginx-config)**.
