---
title: "ddclient - DynamicDNS Setup"
layout: post
comments: true
---


1. sudo apt-get install ddclient
2. put ddclient.conf to /etc/ddclient.conf
3. modify /etc/default/ddclient , ensure run_daemon="true"
4. sudo service ddclient start
5. ps -aux | grep ddclient, ensure `ddclient - sleep for xxx seconds`
   /etc/init.d/ddclient status, ensure 'Status of Dynamic DNS service update utility: ddclient is running.'
