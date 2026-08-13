---
title: Setting up Beszel
published: 2026-06-18
description: 'Monitoring for my home server'
image: ''
tags:
- self-hosted
category: ''
draft: false
lang: ''
---
I had an issue where my server stopped working around 5AM. When I woke up, I saw that the health checks for my services were showing as unhealthy and I was not able to ssh in. REISUB didn't work on my server either and it was not responding to the lid being opened.

After a force shutdown, I checked the boot logs with `journalctl -r -b -1` (show the logs from the previous boot with the latest logs first). I saw frequent messages regarding memory pressure and some errors from docker for two containers (karakeep and pihole) saying that it timed out with starting the health check.

I checked the currently running containers and it seemed to be within limits but thought it would be prudent to setup a metrics monitoring dashboard so that I can see if there are any spikes in resource usage. Beszel seems to be the best pick for this.

I decided to keep the hub running on my VPS with two agents for monitoring. One would monitor my VPS while the other would monitor my home server. I did this because incase my home server goes down for any reason, the hub with the dashboards would not be affected and I can see that my server is offline.

The dashboard is beautiful with a lot of metrics but the thing I found most useful is being able to see container logs directly within beszel which means I don't need to ssh in to my server anymore and run `docker logs` or `docker stats`. However, I do wish that the docker memory limits that I have imposed on most of my compose files (due to my system not having a lot of RAM) would show up as well.

After adding a whitelist for the relevant endpoints on my reverse proxy, I have all of the same data available on my iphone as well (I used the Beszel app by Bruno Durand). I have set up alerts on the app but I will need a more robust alerting system in the future.(maybe pass the alert from beszel to gotify/ntfy?)

An issue that I am currently facing: when there is a power outage in my house, the router goes out first (when the battery backup dies) which means that there is no way for my home server to send the alert. Plus my homeserver is a laptop which has it's own battery with decent backup.(I actually do need to shut it down to see if the alert triggers)

Now I have observability in place for my homelab!

![My Metrics Dashboard](src/assets/images/setting-up-beszel/1.png)
<figcaption>My Metrics Dashboard</figcaption>

![On my Phone](src/assets/images/setting-up-beszel/2.png)
<figcaption>Viewing metrics on my phone</figcaption>
