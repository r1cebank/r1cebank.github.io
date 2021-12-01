---
title: Using docker with ufw
copyright_author_href: 'https://github.com/r1cebank'
date: 2021-01-11 01:49:00
categories:
- Learning
- Docker
tags:
- docker
- linux
- ufw
cover: /covers/docker-ufw.jpg
---
Recently I realized that the personal service server I built literally has every port accessible over LAN. I used `docker-compose` to build all the services I ran on the server and made sure `ufw` is enabled and only allow port **22** and port **443/80.** During a test with a reboot lock down script,  I realized that even I disable all the ports from `ufw`, these ports are still accessible. After googling, I realized it has something to do with docker's own iptables rule somehow will take precedent over ufw's. 

Detailed in the post: [https://blog.viktorpetersson.com/2014/11/03/the-dangers-of-ufw-docker.html](https://blog.viktorpetersson.com/2014/11/03/the-dangers-of-ufw-docker.html)

To disable iptables, create a file `/etc/docker/daemon.json` with the following content:

    {
      "iptables": false
    }
    

I am very interested and kinda shocked this hasn't came up in any of the post I read around docker. It also makes me wonder if this wasn't a bug, since in production the reverse proxy or load balancer is always seperate from the actual hosts that runs the containers.
