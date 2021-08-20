---
layout: post
title: "How to start No-IP during boot in a Raspberry Pi"
date: 2021-08-20 17:44:02 +0100
categories: noip raspberrypi
---
[No-Ip](https://www.noip.com/) is a great service that offers subdomains that work even if you don't have a fixed IP, which can be expensive. This is achieved by running a service in your system that regularly checks your public IP address and, upon each change, updates the records assigned to your chosen subdomain. And in its pricing plans, there's a free tier!

Installing No-IP in a Raspberry Pi is quite simple: you just follow the [instructions](https://www.noip.com/support/knowledgebase/install-ip-duc-onto-raspberry-pi/) and that's it. Now you can just execute it.

But if there's, e.g., a power outage and Raspberry Pi reboots, you need to physically get to its keyboard and manually execute No-IP to have it up and running again, and that's a problem! There are some articles and forum discussions on how to do it, but I didn't get any to work; maybe they worked in the distant past, but not nowadays. You can't run it immediately after boot because there's no networking at the time and you must run it with administrator privileges, so my solution was to edit **crontab** as root:

    sudo crontab -e

Then add an entry waiting 10 seconds and executing No-IP after reboot:

    @reboot sleep 10 && /usr/local/bin/noip2

Save, exit and that's it!
