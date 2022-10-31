---
layout: ../../layouts/project.astro
title: My Homelab
client: Self
publishDate: 2022-11-1 00:00:00
img: 
description: I spend lots of time building and managing my Homelab. Have a read, what I selfhost and what my hard- and software looks like.
portfolio: false  
tags:
  - homelab
  - server
  - selfhosting  
---

### Whats a Homelab?
Well, as so often there's no real definition for a Homelab. If I had to choose a definition tho, I'd say it's the process of dedicating hardware for a specific task, often selfhosting applications and services to gain independence from paid solutions or big companies like Google. But it doesn’t stop there, and possibilities are near endless: Selfhosting cloud-storage, virtualizing whole operating systems, building up clusters and strengthen up your network and its security are only a few of many use-cases of a Homelab. But the most important thing: It's a fun experience and you acquire heaps of knowledge! I’ll show you how I do things in my Homelab in the following post.

![a picture showing three customer-grade servers](/images/homelab.jpeg "My Homelab")

### What Hardware do I own?
I have three main machines, each covering an important task in my infrastructure:
 - **Big Iron:** Equipped with an energy efficient Intel Celeron J4105 CPU, 8GB of RAM and a total raw capacity of 16TB in HDD Storage, Big Iron is my main storage server which I built in early 2022. The operating system is TrueNas Core, a FreeBSD-based server OS with ZFS as its file system. Big Iron is the production storage server, with all its shares for other machines and personal clouds for my girlfriend and me
- **VM Square:** My little virtual machine powerhouse is an Intel NUC with an i5-8259U CPU and 32GB of RAM. The CPU is plenty fast for my needs and can handle various concurrent VMs with ease. The OS is Proxmox, a popular OS for virtualization. VM Square holds all of my non-storage services, with the most prominent being Docker. You can find a list of all the services I run further down.
- **Synology DS 216+ II:** The first Homelab-ish server I owned. I opted for a pre-built solution back then, as I hadn’t much knowledge about hardware, software and networking in general. As my knowledge and needs for my infrastructure grew, I also quickly outgrew this Diskstation and opted for more powerful options with a less limiting operating system. The Diskstation now handles all of my backups for Big Iron locally and sends them encrypted to a cloud provider for an off-site backup for a 3-2-1 backup strategy.

### What am I running on VM Square?
I have three main virtual machines:

1. LXC Container with Docker + Portainer: Almost all of my services run containerized in Docker. Why? Docker is the easiest way to guarantee a uniform runtime for all applications and great community support ensures that lots of awesome images are available at all times. 
Here’s a list of services I run on docker and what they’re for:
   - **Vaultwarden:** An open-source branch of Bitwarden, a popular password manager. 
   - **Uptime-Kuma:** Status-monitor that alerts me automatically if a service goes down.
   - **PicoShare:** Fileshare service where you can upload files and more to share via a generated shortlink. No need for Dropbox anymore.
   - <a href="https://me.maiermanuel.website" target="_blank">**LittleLink**</a> = A personal landing page containing all my links and socials.
   - **Nginx Proxy Manager:** A easy to use reverse proxy used to route subdomains for my personal services while giving SSL to selfhosted services.
   - **ihateMoney:** Well no, but in that regard yes 😄. ihateMoney is a spending tracking tool mainly focusing on splitting bills the right way. My partner and I use it to keep track who paid what.
   - **Guacamole:** This has been a gamechanger for me lately. Guacamole is a web-based gateway to all your machines supporting various protocols like SSH, RDP, VNC and more. With that, you just have to log in to guacamole and click on the connection you want to establish, instead of fiddling around with credentials or keys in the console.
   ![video showing how to access windows from guacamole](/images/homelab/guacamole.gif)
   With that you could have access to a full-fletched Windows machine only in your browser!
   - **Flame:** Flame is a beautiful and minimal dashboard visualizing all of the above-mentioned services in one website. With that, I always have shortcuts to all my services increasing my navigation time drastically.
   ![flame dashboard](/images/homelab/flame.png)

2. LXC Container with Ubuntu Server running Pi-Hole. Pi-Hole is a DNS resolver that blocks unwanted tracking links and bad sites ensuring a more private and secure browsing experience.

3. Virtual Machine with Windows 11. As both, my partner and I, switched to M1 MacBook Airs in 2021, I simply virtualized a Windows Instance in case one of us has to use some software that is only available for Windows. In the future I might want to turn this instance to a gaming server plugging in a external GPU to VM Square passing through that GPU to the Windows instance.

### What about Networking?

As you saw in the GIF showcasing guacamole, I use my own domain for all of my services. If you have concerns right now that I accidentally leaked a domain of mine, I encourage you to type that URL in the search bar and see what happens 😌. You even could do a **nslookup** with that domain, it will return a private IP address. Huh, how’s that working out? I use a service called **Tailscale** to route traffic which targets my private infrastructure through a VPN. For a long time now, I’ve used a classic reverse proxy setup, opening up port 80 and 443 on my router and forward it to Nginx proxy manager, additionally exposing my public IP. I didn’t have any unsecured resources accessible via my public IP and everything was at least protected via authentication, but I wasn’t necessarily happy with my setup. The first step to improve security was to proxy any incoming traffic through Cloudflare, effectively hiding my public IP, but still leaving port 80/443 open. I searched for alternatives and a better approach to things, and this is where I stumbled across Tailscale. Tailscale *is* a VPN service, but it differs from any regular VPN. It establishes encrypted end-to-end connections for IP’s that normally would require a public IP while routing normal traffic the normal way. 

<figure>
  <img src="/images/homelab/direct-access.svg" alt="tailscale sketch"/>
  <figcaption> 
  <p>A client can establish a direct connection to a machine in my Homelab for example, no need for a VPN-endpoint</p> 
  <p>Taken from 
  <a href="https://tailscale.com/blog/how-tailscale-works/" target="_blank">tailscale.com</a></p>
  </figcaption>
</figure>

In other words, if I address a resource located in my Homelab, Tailscale will detect the destination and route that traffic through an E2E-connected tunnel, completely bypassing any ports. Normal traffic will go through its normal route, for example a router wherever I might be. Optionally I can declare an *exit-node*, an endpoint where all my traffic should be routed through, being ultimately the same as a VPN in a traditional manner.

### What’s the benefit of that? 
My services aren't accessible via the public internet and I don’t have to open any port at all. If you’d happen to scan my public IP for open ports (as botnets often do) you’d find nothing, therefore increasing security vastly. Often some people aren’t allowed to open ports through their ISP or are behind a double NAT, resulting in difficulties with a classic reverse proxy setup. Tailscale is more robust to that. Almost everything is addressable as long as it has access to the internet. 

<figure>
  <img src="/images/homelab/tailscale-remote-access.svg" alt="tailscale sketch"/>
  <figcaption> Taken from 
  <a href="https://tailscale.com/blog/how-tailscale-works/" target="_blank">tailscale.com</a>
  </figcaption>
</figure>

This has been my Homelab in a short run! It’s far from complete and my next project will be a custom-built router with pfsense as its OS to strengthen my knowledge about networking even more. So, stay tuned for a follow up to this post! :)