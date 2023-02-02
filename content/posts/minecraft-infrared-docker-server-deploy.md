---
title: "How to Host a Minecraft Server From Home"
description: "Hosting a minecraft server doesn't have to be hard lets use docker and infrared to get it all setup"
categories: [Docker, Reverse Proxy, Infrared]
date: 2023-01-27T11:49:26-05:00
draft: true
---

{{TOC}}

# Introduction
While hosting a Minecraft server isn't difficult it can be tricky. I'll be using Docker and a reverse proxy to show you how to setup and host your Minecraft server safe and securely! This all came about because while I thought it was going to be easy it ended up being a lot harder than it should have been. Especially when it came to setting up the reverse proxy.

# Prerequisites
Before we get started you'll need the following.
- Docker and docker compose
- The ability to port forward on your router
- A domain name (for this example we'll be using 'mc.minecraftexample.com' as the domain name)
- The ability to setup A records on that domain (I recommend Cloudflare but it's up to you)
- Your devices IP address

# Setting Up The Minecraft Server
First we need to setup a Minecraft server. I'll be using the [itzg/minecraft-server](https://github.com/itzg/docker-minecraft-server) as it is extremely flexible and allows for a lot of customization with mods. Create a file called 'docker-compose.yml' this is where we will be deploying the containers. Paste the following into your docker compose file.

```yml
version: "3.3"

services:
  vanilla:
    container_name: vanilla
    restart: unless-stopped
    image: itzg/minecraft-server
    environment:
      EULA: "true"
```
This will create a vanilla server and start it with the eula accepted. Take note of the container name that will be important later. Also did you notice we didn't expose any ports on this container? That's because Infrared is going to be taking care of that!

Now run the following command this will spin up the server and generate the world. Do this before the next step just so you can double check and make sure that the container starts up correctly.

```
docker-compose up
```

# Setting Up Your DNS Records
For infrared to work you need a couple things. You need to forward port 25565 on your router as udp/tcp. I will not be getting into that here because it is out of the scope of this article. It's fairly easy to do. A quick google search will send you down the right path. Once you have that setup come back!

You'll also need to know your public IP I will not be sharing mine here but you can use [https://whatismyipaddress.com](https://whatismyipaddress.com) to find it.

Now that we have forwarded a port and know our public IP address we can setup our DNS records. For this example I'll be using Cloudflare. Let's go ahead and create our A record.
dnssetup.png
Make sure the ipv4 address points to your public ip and if you're using Cloudflare that the proxy is turned off. I haven't been able to get the proxy feature to work with it on sadly.

# Setting Up The Reverse Proxy
Now here comes the fun part. We’re going to setup [Infrared](https://github.com/haveachin/infrared). It’s a really neat reverse proxy that sits on top of your Minecraft server. It then forwards the traffic to your server depending on the request made to it. It even lets you auto shutdown the server if there’s no traffic to it and it even starts it up when a request is made! This allows you to host multiple Minecraft servers on one system and have them all use a single entry port. No more opening a bunch of random ports!

Now that we know what it is and what it does it’s time to add it to our docker compose. Delete everything you have in there and copy and paste this in.

```yaml
version: "3.3"

services:
  vanilla:
    container_name: vanilla
    restart: unless-stopped
    image: itzg/minecraft-server
    environment:
      EULA: "true"
    volumes:
      - /home/noahvlncrt/docker/mc:/data
  infrared:
    image: haveachin/infrared
    container_name: infrared
    restart: unless-stopped
    stdin_open: true
    tty: true
    ports:
      - 25565:25565/tcp
    volumes:
      - /<path-to-infrared-config>/:/configs
      - /var/run/docker.sock:/var/run/docker.sock
    expose:
      - "25565"
    environment:
      INFRARED_CONFIG_PATH: "/configs"
```

Don't start it yet! If you've been paying attention you'll notice there is a volume mapped for infrared configs. Now this is truly where all the magic happens. Go ahead and create a folder to put them in. Once you do that create a file called mc.minecraftserver.com.json. Paste the following in.

```json
{
    "domainName": "mc.noahvlncrt.cloud",
    "proxyTo": "vanilla:25565",
    "listenTo": "0.0.0.0:25565"
}
```

Let's go over what we're looking at here. The first line "domainName" is what domain name the server is under. This is where the A record you setup earlier comes in handy. The second line "proxyTo" tells infrared what server to send the traffic too. This is where the magic of Dockers DNS comes in handy. I kept getting hung up here because even if I set it to my devices ip I would still run into issues. The only way I was able to get it to work is if I set it to the containers name. Which in my opinion is a cleaner way of handling it. The third line "listenTo" tells infrared what ip to listen to for incoming connections.

# Finishing Up
Now that everything is all setup up correctly all you have to do is run.

```sh
docker-compose up
```

This will spin up both containers. After a few minutes open up Minecraft and try connecting to your server at mc.minecraftexample.com and boom! It should all be working perfectly. I recommend checking out the documentation for both the [itzg/minecraft-server](https://github.com/itzg/docker-minecraft-server) and [haveachin/infrared](https://github.com/haveachin/infrared) containers. They have lots of config options that are super handy. Infrared even has an API server that lets you dynamically add entries to it. Enjoy your server and have fun playing!