---
title: Docker-Hole (Ad Blocking on the Go!)
date: 2023-10-22 12:00:00 -500
categories: [security,raspberry_pi,docker]
tags: [security,raspberry_pi,docker,pihole,wireguard]
---

![featured_image](dockerhole.jpg)

### Block Ads Remotely with Pi-Hole, WireGuard, and Docker!

If you're reading this blog, theres a good chance you already know about Pi-Hole, the popular tool for blocking unwanted ads within your home network. But wouldn't it be cool to bring this tool with you whever you may be? This can be done with a few simple tools which we'll be running inside docker containers for added simplicity.

* Step 1: Install Docker and Docker Compose
* Step 2: Create and start the Pi-Hole Docker container
* Step 3: Create and start the WireGuard Docker container
* Step 4: Forward the port for WireGuard on your firewall/router
* Step 5: Copy WireGuard peer configuartions to your mobile devices

#### Step 1: Download and install Docker and Docker-Compose for your OS (I will be using Raspbian).

```bash
sudo apt install docker docker-compose
```

You may need to start and enable the service depending on your OS

```bash
systemctl start docker.service
systemctl enable docker.service
```

#### Step 2: Installing Pi-Hole inside a Docker container

Head on over to [Pi-Hole's Docker Hub](https://hub.docker.com/r/pihole/pihole) and copy the docker-compose contents into a new **.yml** file on your host. Be sure to make the necessary changes to suit your needs.
