---
title: My Self Hosting Journey
published: 2025-06-06
description: 'An overview of my self hosting journey thus far and where I am going with it in the future'
image: ''
tags: [Selfhosted,Blog]
category: 'Tech'
draft: false 
---

## How it started 
I came across selfhosting during my Reddit doomscrolling back in 2020 while I was still a college student. At the time, the kind of content that I watched was very privacy conscious and it seemed like a good idea to me to have control over the hardware that would store my data. Thus, I went into the rabbit-hole with the very simple goal of hosting my own cloud (ala Nextcloud). I did get things set up but was immediately faced with the issue that I couldn't access it outside my local network and the project was kept on hold for a while.

The year is 2023 and I was working as a Software Engineer at a large services MNC. While I had been keeping abreast of some of the developments in the self hosting scene, I did not try anything out on my own hardware until I got to know of Tailscale. This was a VPN that I could set up which would solve my issue of not being able to access my services through the internet. I simply had to add all of my devices to the 'Tailnet' and when I turned on the VPN, my device would behave as if it was connected to my local home network. As soon as I got it working I was very excited and immediately started working on a setup that would help me remote play from my Steam Deck to my PlayStation through Chiaki. I wrote a script that would query arp for the device name which would never change and get the ip from that. However I did face a lot of artifacting when streaming with packets being dropped which was an issue I could not solve and my homelabbing was put on hold again.
## How it's going
I moved back home in 2024 and decided to give selfhosting another try. This time I had a clear requirement which was to have a cloud based library for my jailbroken kindle. I decided on calibre and calibre-web to manage the library and portainer to keep an eye on my docker containers. Since I was only using this locally, I didn't need to worry about a reverse proxy or anything like that and would just access my calibre instance directly with the ip of the server and the port the service was running on. This setup worked well and allowed me to sync books from multiple sources and access everything on my kindle.
## I can't stop tinkering!
Some improvements that I am currently in the middle of trying out include: getting a domain and connecting my server to that so I can access it from anywhere (probably going to try out Pangolin- which is an alternative to CF tunnels), try running other services such as nginx (reverse proxy), heimdall(dashboard), build my own application and spin up a container for that (this would be a webpage that would allow me to write reviews for the books I read and sync that to my calibre library metadata), try out ansible (help me replicate my server infrastructure easily incase something goes wrong), set up syncthing so that I can sync my obsidian vault across my devices (Linux and iOS).
