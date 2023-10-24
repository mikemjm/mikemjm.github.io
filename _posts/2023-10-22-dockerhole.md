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

```yaml
services:
  pihole:
    container_name: pihole
    image: pihole/pihole:latest
    # For DHCP it is recommended to remove these ports and instead add: network_mode: "host"
    ports:
      - "53:53/tcp"
      - "53:53/udp"
      - "67:67/udp" # Only required if you are using Pi-hole as your DHCP server
      - "80:80/tcp"
    environment:
      TZ: 'America/Chicago'
      # WEBPASSWORD: 'set a secure password here or it will be random'
    # Volumes store your data between container upgrades
    volumes:
      - './etc-pihole:/etc/pihole'
      - './etc-dnsmasq.d:/etc/dnsmasq.d'
    #   https://github.com/pi-hole/docker-pi-hole#note-on-capabilities
    cap_add:
      - NET_ADMIN # Required if you are using Pi-hole as your DHCP server, else not needed
    restart: unless-stopped

### EXTRA NETWORK CONFIGURATION TO SETUP A 
### STATIC IP ADDRESS FOR THE PIHOLE DOCKER
### CONTAINER ###

    networks:
      network:
        ipv4_address: 172.50.0.2

networks:
  network:
    driver: bridge
    ipam:
      config:
        - subnet: 172.50.0.0/16
          gateway: 172.50.0.1
```

I've added additional network settings at the end of the configuration that will setup a new bridged network and a static IP address for the docker container running Pi-Hole. We don't want the IP address of this container changing if the container or host is restarted. In this configuration the container will always bee assigned the IP address of **172.50.0.2** and a gateway of **172.50.0.1**

* Save the file, E.g. pihole-docker.yml

Create the container

```bash
docker-compose -f pihole-docker.yml up -d
```

```bash
Creating network "mike_network" with driver "bridge"
Pulling pihole (pihole/pihole:latest)...
latest: Pulling from pihole/pihole
85e50d2242ce: Pull complete
205b3bcb04a1: Pull complete
4f4fb700ef54: Pull complete
efc47020d282: Pull complete
3a438191e6f2: Pull complete
a090ee43303f: Pull complete
10002067986b: Pull complete
30034a05debf: Pull complete
d61aa1a24eb4: Pull complete
Digest: sha256:562766abc805d5005bb2d2aa5d4bbf2d9b347380b1ea4504fb59b2100500f672
Status: Downloaded newer image for pihole/pihole:latest
Creating pihole ... done
```

The container is now created and started, you can verify by issuing the following command

```bash
docker ps
```
Verify you can access Pi-Hole's dashboard locally, http://hostIP/admin

#### Step 3: Installing Wireguard inside another Docker container

Now we need to setup a VPN server so we can access our private network remotely. We'll be using [LinuxServer's](https://hub.docker.com/r/linuxserver/wireguard) Docker image.

Paste the following contents into a new file on your host, make sure to save it with extension **.yml**. Be sure to make the necessary changes to suit your needs.

