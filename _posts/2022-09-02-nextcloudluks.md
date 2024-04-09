---
title: LUKS Encrypted Nextcloud Storage + Docker
date: 2022-09-02 12:00:00 -500
categories: [nextcloud,raspberry_pi,security]
tags: [nextcloud,raspberry_pi,security]
---

### Setting up a LUKS encrypted external storage device to store Nextcloud data with Docker.

I always encrypt all of my data, however I also like to have that data available to me as conveniently as possible. Nextcloud provides a convenient way to access my data remotely while maintaining the level of protection I am comfortable with. Before we get started it’s important to note that Nextcloud comes equipped with its own server side data encryption, which can be enabled under the “apps” section. I will be forgoing Nextcloud encryption and using LUKS for my external storage device.

![luks_stick](/assets/nextcloudluks/LUKS_STICK.webp)

I’m not going to be port forwarding my Nextcloud container out to the internet. I will only be able to access it when I VPN into my home network.

#### Installation

Download and install Docker and Docker-Compose for your OS, I will be using ArchARM.

* Raspbian

```bash
sudo apt install docker docker-compose
```

* ArchARM

```bash
sudo pacman -S docker docker-compose
```

#### Create the Required Folders

Create these folders wherever you see fit, I am going to create mine inside of my home directory.
```bash
mkdir /home/mike/docker/nextcloud/config #Nextcloud configuration files will be stored here
```

```bash
mkdir /home/mike/docker/nextcloud/crypt_data #encrypted drive mount point
```
#### Partition the Device

* First we need to identify the device we wish to encrypt by calling the “lsblk” command.

* The device we want is /dev/sda, lets go ahead and create a new partition using fdisk.

![lsblk](/assets/nextcloudluks/lsblk.webp)

Open /dev/sda with fdisk.

```bash
sudo fdisk /dev/sda
```

1. Type “o” to clear any existing partitions.
2. Type “n” and then “p” to begin creating a new primary partition.
3. Press “enter” to accept the default primary partition number.
4. Press “enter” to accept the default first sector.
5. Press “enter” again to accept the default last sector.
6. Type “w” to write the changes to the disk and exit fdisk.

#### Encrypting and Mounting the Partition

Encrypt the new partition using LUKS.

```bash
sudo cryptsetup -y -v luksFormat /dev/sda1
```

Follow the prompts and choose a passphrase.

Now we need to mount and format the encrypted partition so we can use it.

```bash
sudo cryptsetup open /dev/sda1 crypt_data
```

I named my crypt container “nextcloud_crypt” but you can call it whatever you desire.

Now that the container is open we can format it.

```bash
sudo mkfs.ext4 /dev/mapper/crypt_data
```

Lets mount the encrypted partition to the Nextcloud data folder we created earlier

```bash
sudo mount /dev/mapper/crypt_data docker/nextcloud/crypt_data
```

### Installing Nextcloud with Docker

[LinuxServer.io](https://www.linuxserver.io/) is a great place to get some awesome Docker images, we will be pulling their Nextcloud image from their Docker Hub.

1. Navigate to your Docker/nextcloud folder and create a new file, name it docker-compose.yml and paste the following inside.

```yml
version: "2.1"
services:
  nextcloud:
    image: lscr.io/linuxserver/nextcloud:latest
    container_name: nextcloud
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=America/Pacific #set your timezone
    volumes:
      - /home/mike/docker/nextcloud/config:/config #your path may differ
      - /home/mike/docker/nextcloud/crypt_data:/data #your path may differ
    ports:
      - 443:443
    restart: unless-stopped
```

2. Now run docker-compose.

```bash
sudo docker-compose up -d
```

3. Verify the container is running.

```bash
sudo docker ps
```

4. Open a web browser and navigate to [https://HostAddress/](https://127.0.0.1) and finish the setup.

![nextcloud_dash](/assets/nextcloudluks/nextcloud_dash.webp)

You can verify that your files are being stored on your encrypted drive by navigating to your encrypted mount point.

```bash
cd /home/USERNAME/docker/nextcloud/crypt_data/USERNAME/files/
```

You’ve successfully installed Nextcloud with Docker and have a LUKS encrypted storage folder for all of your files!