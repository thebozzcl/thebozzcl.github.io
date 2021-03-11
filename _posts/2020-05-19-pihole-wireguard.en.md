---
layout: post
permalink: /posts/2020-05-19-pihole-wireguard.html
title: "How to take your Pi-hole anywhere using WireGuard"
date: 2020-05-19 19:00:00 -0800
tags: [post, coding, privacy]
lang: en
---

I've been using [Pi-hole](https://pi-hole.net/) for several years. It's just a fantastic way to both improve your online experience and protect your privacy... but it has one problem: you can't bring it everywhere with you. Let's fix that by serving it through a home VPN!

> ⚠️ **Update on 2021/03/11**: I missed some steps regarding Dynamic DNS during the original write-up. The article should be complete now.

<!--more-->

## Why would you want to do this?

My main motivation is ad blocking. Anybody who knows me will be able to tell you this: I hate ads and activity trackers with a passion. I go out of my way to remove them from my life. Being able to use my Pi-hole from anywhere I want is a huge boon.

There's other reasons you could want something like this:
* Access your home network remotely
* Give your friends secure access to a home server
* Protect your privacy on untrusted networks

It's very easy to do and can be done on the cheap.

## Requirements

You're going to need a few things:
* A Raspberry Pi. I use a [RPi 4 model B](https://www.raspberrypi.org/products/raspberry-pi-4-model-b/), but I think older models should work as well.
* A domain name.
* A router that supports port forwarding (most do, as far as I know).
* An account in a DNS hosting service that supports Dynamic DNS.
  * I use [Hurricane Electric](https://dns.he.net/). Don't let the spartan web site fool you... it performs well, it's got enough features and it's completely free! (At least for the scope of this project)
* Your router's public IP address.

## First part: set up your WireGuard domain

Once you've bought your domain and registered on a DNS service, you need to:
1. Add your domain to the DNS service, following any instructions they give you. You should have at least the following records:
   * SOA (start of authority)
   * NS (nameserver)
2. Once you have these ready, add an A (domain-to-IP mapping) record with the following settings:
   * Name: your domain name (or a subdomain, if you want to use the domain name for something else)
   * IP address: your public IP address
   * TTL: no more than 300 (5 minutes)
   * Dynamic DNS enabled. This will ensure your VPN works even if your public IP address changes.
3. Create a Dynamic DNS password for your A record and write it down. This is required so your Raspberry Pi or router can update the A record.
4. Set up Dynamic DNS updates:
   * If you want to do them through your Raspberry Pi, you can use `ddclient`.
   * I just used my router's Dynamic DNS settings (which probably uses `ddclient` anyway).
   * For Hurricane Electric, these are the settings I used:
     * Provider: custom
     * Dynamic DNS server: `dyn.dns.he.net`
     * Request: `/nic/update`
     * Domain: your domain or subdomain name
     * Username: your domain or subdomain name
     * Password: the Dynamic DNS password you created in step `3.`

This will allow you to connect to your router remotely. **Make sure to disable WAN access to your router's admin UI!**

## Next: set up Pi-hole

**NOTE**: most of these instructions link to the manufacturer or software's websites. If any links break let me know and I'll do my best to fix them.

So you got your Raspberry Pi out of the box and ready to go? Setting up Pi-hole and WireGuard on it is very easy:
1. [Install your OS of choice](https://www.raspberrypi.org/documentation/installation/installing-images/). While you're doing this, I recommend you [enable SSH](https://www.raspberrypi.org/documentation/remote-access/ssh/).
2. SSH into your Raspberry Pi.
3. Give your Raspberry Pi a static LAN IP in your router's DHCP settings, then write it down
   * Reboot your Raspberry Pi (or its networking service) to apply the new IP address
4. [Install Pi-hole](https://github.com/pi-hole/pi-hole/#one-step-automated-install).
   * I recommend the automated install to keep things easy: `curl -sSL https://install.pi-hole.net | bash`
   * The install tool will let you choose your upstream DNS providers. Some of the options it gives you are not privacy-friendly, so choose carefully.
   * During installation, the tool will allow you to choose to set your Pi's current local IP address as static. Do choose this option. Make sure you allocate this IP on your router as well!
5. Once you've confirmed your Pi-hole installation is working (check the dashboard available at http://pi.hole/admin/), set your router to use the Raspberry Pi as your *only* DNS server.
   * If you want redundancy in case your Pi breaks, you could add a secondary DNS server.

Done! Now any devices on your home network should be able to use your Raspberry Pi to resolve URLs and block ad or tracking domains.

## Finally: setting up WireGuard

Of course, this is still not usable remotely, so we need to install WireGuard:
1. [Install PiVPN](https://www.pivpn.io/)
   * This one also has an automated installer: `curl -L https://install.pivpn.io | bash`
   * During installation, choose WireGuard as your VPN protocol. Choose whatever port you want, as long as you write it down.
   * When the installer asks you to choose between using a public IP and using a public DNS, choose the DNS. In the next screen, write the (sub)domain you set up earlier.
   * The installer should detect your Pi-hole installation and ask if it should use it. Accept it!
2. In your router's settings, enable port forwarding to your Raspberry Pi for the WireGuard port. By default, this is `51820`.

And that's it! Now you need to test your new VPN. I recommend using your phone:
1. In the Raspberry Pi, set up a user profile by running `pivpn -a`.
2. Once it's ready, you can use that profile in other devices by:
   * Copying the generated config file over to your device.
   * **Recommended**: using a QR code generated by the `pivpn -qr` command. We're gonna use this one, which should draw a QR code in your terminal.
3. Whip out your phone and install the WireGuard client:
   * [Android](https://play.google.com/store/apps/details?id=com.wireguard.android&hl=en_US)
   * [iOS](https://apps.apple.com/us/app/wireguard/id1441195209)
4. Open the app, and set up a new tunnel. If you followed my advice and used the QR code, you should be able to just point your phone camera at it and the app will do the rest.

Time to test it out! Disconnect your phone from your WiFi, then try to open http://pi.hole/ again. If you did it right, it should open just like that. If that worked, congratulations!

## What if I don't want to do all of this?

...I mean, I think every step of this tutorial is easy to follow, but that's fair. There's VPN services that offer ad-blocking DNS as part of the bundle, like [Windscribe](https://windscribe.com/). Or you can use a local VPN solution like [DNS66](https://f-droid.org/en/packages/org.jak_linux.dns66/). There's others, these two are just the ones I've tried in the past and liked.

Both options work very well, but they have their shortcomings:
* They only work on a limited number of devices at a time (you could set up the VPN in your router, but it will slow down your connection).
* In the case of the VPN, you're trusting the provider not record, share or sell your browsing habits. That's a risk I wasn't willing to take.

Whichever option you choose, the internet really changes for the best once you remove most of the noise. Enjoy!

## Common troubleshooting steps

These are some of the issues I've run when re-configuring my WireGuard VPN:

* Make sure your Dynamic DNS is updating properly. This could be either a wrong password or Dynamic DNS settings.
* Make sure your port forwarding is set up correctly. You have no idea how many times I got the port wrong.
* If you're having trouble accessing devices on your home network remotely, try the following:
  1. Open the WireGuard client in your device.
  2. Edit the WireGuard tunnel.
  3. On "Allowed IPs", add a IP range that matches your home network. Some typical values are `192.168.0.0/24` and `192.168.1.0/24`.
  4. Save and try again.
