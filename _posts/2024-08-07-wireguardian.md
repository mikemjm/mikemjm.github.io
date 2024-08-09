---
title: WireGuardian (Ad Blocking on the Go, AdGuard Edition!)
date: 2024-08-07 12:00:00 -500
categories: [security,raspberry_pi,docker]
tags: [security,raspberry_pi,docker,adguard,wireguard]
---

![featured_image](/assets/adguardian/adguardian.jpg)

### Similar to [Docker-Hole](/posts/dockerhole/), WireGuardian is easier to setup and replaces Pi-Hole with AdGuard Home!

Enjoy an ad free experience at home and on the go with with WireGuard + AdGuard Home. Easy to setup with just one docker compose file, follow along below!

* Step 1: Install Docker and Docker Compose
* Step 2: Create and start the containers with the provided docker compose contents
* Step 3: Forward the appropriate port on your firewall/router
* Step 4: Copy the WireGuard peer configurations to your mobile devices
* Step 5: Verify everything works

---

#### Step 1: Download and install Docker and Docker-Compose for your OS (I will be using Raspbian).

```bash
sudo apt install docker docker-compose
```

> You may need to start and enable the service depending on your OS
{: .prompt-tip }

```bash
systemctl start docker.service
systemctl enable docker.service
```

> If you are unable to run docker commands under your user, you need to add yourself to the "docker" group. Be sure to logout/login for this to take effect.
{: .prompt-tip }

```bash
usermod -a -G docker username
```

---

#### Step 2: Create and start the Adguard and WireGuard containers

Copy and save the docker compose contents below and adjust to your requirements

```yaml
services:
  adguardhome:
    image: adguard/adguardhome
    container_name: adguardhome
    hostname: adguard
    ports:
      - 53:53/tcp
      - 53:53/udp
      - 67:67/udp
#     - 68:68/udp  #not needed, for DHCP clients only
      - 784:784/udp
      - 853:853/tcp
      - 853:853/udp
      - 3000:3000/tcp
      - 80:80/tcp
      - 443:443/tcp
      - 443:443/udp
      - 8853:8853/udp
      - 5443:5443/tcp
      - 5443:5443/udp
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /home/mike/docker/adguard/workdir:/opt/adguardhome/work
      - /home/mike/docker/adguard/confdir:/opt/adguardhome/conf
    restart: unless-stopped

    networks:
      services:
        ipv4_address: 172.50.0.2 #the static IP address of the Adguard container

####################################################################################

  wireguard:
    image: lscr.io/linuxserver/wireguard:latest
    container_name: wireguard
    hostname: wireguard
    cap_add:
      - NET_ADMIN
      - SYS_MODULE #optional
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=America/Los_Angeles #replace with your timezone
      - SERVERURL-X.X.X.X #WAN IP address here
      - SERVERPORT=51820
      - PEERS=peer1,peer2 #declaration of peers (see variables https://github.com/linuxserver/docker-wireguard?tab=readme-ov-file)
      - PEERDNS=172.50.0.2 #the ip address of the AdGuard container
      - INTERNAL_SUBNET=10.13.13.0
      - ALLOWEDIPS=0.0.0.0/0
      - PERSISTENTKEEPALIVE_PEERS= #optional
      - LOG_CONFS=true
    volumes:
      - /etc/localtime:/etc/localtime:ro #optional
      - /home/mike/docker/wireguard/config:/config
      - /lib/modules:/lib/modules #optional
    ports:
      - 51820:51820/udp
    sysctls:
      - net.ipv4.conf.all.src_valid_mark=1
    restart: unless-stopped

    networks:
      services:
        ipv4_address: 172.50.0.3 #the static IP address of the wireguard container

### Creates a new docker network named "docker_configs_services" in which the two above containers will reside

networks:
  services:
    driver: bridge
    ipam:
      config:
        - subnet: 172.50.0.0/16
          gateway: 172.50.0.1
```

* Save the file, E.g. wireguardian-docker.yml

Create the container

```bash
docker-compose -f wireguardian-docker.yml up -d
```

```bash
Creating network "mike_services" with driver "bridge"
Pulling adguardhome (adguard/adguardhome:)...
latest: Pulling from adguard/adguardhome
5c905c7ebe2f: Pull complete
a387caba1320: Pull complete
a5e08f26b570: Pull complete
0e31fdbeff37: Pull complete
4f4fb700ef54: Pull complete
Digest: sha256:d16cc7517ab96f843e7f8bf8826402dba98f5e6b175858920296243332391589
Status: Downloaded newer image for adguard/adguardhome:latest
Pulling wireguard (lscr.io/linuxserver/wireguard:latest)...
latest: Pulling from linuxserver/wireguard
3b9a5ab1d346: Pull complete
df25a931801a: Pull complete
15410b37d6a0: Pull complete
67b4f3e802d4: Pull complete
e4b9de32bfa4: Pull complete
3849eecba27d: Pull complete
41ccddb2c3bd: Pull complete
a7b5af4a3a0b: Pull complete
Digest: sha256:f7feb3d014d5b5aff6d69d1430ef04e7742f425ecb61173ba0fec27890e890ef
Status: Downloaded newer image for lscr.io/linuxserver/wireguard:latest
Creating wireguard   ... done
Creating adguardhome ... done
```

The two containers are now created and started, verify by issuing the following command

```bash
docker ps
```
Browse to AdGuard on your host and click through the setup guide, http://hostIP:3000

---

#### Step 3: Port Forwarding

Forward WireGuard port **51820** to the host running your Docker containers

Here is what that looks like on my EdgeRouter

![port_forward](/assets/dockerhole/port_forward.png)

---

#### Step 4: Copy WireGuard peer configurations to your mobile devices

WireGuard has a cool built in feature that will generate a QR code for each peer that you can scan with your mobile device.

```bash
docker exec -it wireguard /app/show-peer peer1 #in this case we used peer1 and peer2 when we created the container
```

A QR code should be displayed for "peer1"

If you are unable to use QR codes, client configuration files are saved at the location specified in the docker-compose configuration file for WireGuard. In this case it's */home/mike/docker/wireguard/config*

![query_log](/assets/adguardian/query_log.png)

### Success, AdGuard is filtering DNS queries it's receiving from WireGuard! Note the IP address of the incoming queries.

For an ad free experience at home, be sure point your local devices to the host running AdGuard, E.g. 192.168.1.250 (my Raspberry Pi's address)

> The addresses shown on the dashboard will not work since they are internal to Docker only
{: .prompt-warning }

![web_interface](/assets/adguardian/admin_web_interface.png)

## Administration:

#### Upgrading Containers

* Step 1 - Identify the container that needs to be updated

```bash
docker container ls
```

Copy the container ID and stop/remove the container

```bash
docker container stop container_id && docker container rm container_id
```

* Step 2

Identify the container image

```bash
docker image ls
```

Copy the image id and remove it

```bash
docker image rm image_id
```

* Step 3

Run the docker-compose file again, it will pull the new image and start the container, files are persistent and all settings will remain.

```bash
docker-compose -f wireguardian-docker.yml up -d
```

Enjoy an ad free internet!