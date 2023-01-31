---
title: "How to Host a Minecraft Server From Home"
description: "Hosting a minecraft server doesn't have to be hard lets use docker and infrared to get it all setup"
categories: [Docker, Reverse Proxy, Infrared]
date: 2023-01-27T11:49:26-05:00
draft: false
---

# Introduction
While hosting a Minecraft server isn't difficult it can be tricky. I'll be using Docker and a reverse proxy to show you how to setup and host your Minecraft server safe and securely! This all came about because while I thought it was going to be easy it ended up being a lot harder then it should have been. Especially when it came to setting up the reverse proxy.

# Prerequisites
Before we get started you'll need the following.
- Docker and docker compose
- The ability to port forward on your router
- A domain name (for this we'll be using 'minecraftserver.com' as the domain name)
- The ability to setup A records on that domain (I recommend cloudflare but it's up to you)

# Setting Up The Minecraft Server
First we need to setup a Minecraft server. I'll be using the itzg/minecraft-server image as it is extremely flexible and allows for a lot of customization with mods. Create a file called 'docker-compose.yml' this is where we will be deploying the containers. Paste the following into your docker compose file.

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


