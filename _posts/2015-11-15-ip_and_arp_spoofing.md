---
layout: post
title: IP & ARP spoofing
tags: [ip, arp, spoofing, network, security]
summary: Some concepts and a pen test about the spoofing techniques in IP and ARP.
---

> **Disclaimer**: This is for educational use only.

## Introduction

In the post *[ARP Spofing]({{ site.url }}{% post_url 2015-4-20-arp_spoofing %})* i was spoke about the ARP spoofing and how use it to get the network packages sended to another PC. In this post now, i will speak about the same thing, but with the IP, in others words, how to change the source IP of a package.

## IP Spoofing

Into the IP layer of network, as is explained into this image

{% figure posts/ip_and_arp_spoofing/ip_layer gif 'IP package description' %}

Here, will can see the `Source Address` field, so when a network packet (by default ;)) leave our computer, the operating system set the ip address of the output interface into that field, that is because the remote computer need response to us.

So, for example, we use **cntlm** proxy program with the gateway option filter by IP, the configuration

{% highlight yaml linenos %}
Gateway        yes

Allow          127.0.0.1
Allow          192.168.12.3
Deny           0/0
{% endhighlight %}

In this example, there are 2 ips allowed to enter `127.0.0.1` for local purpose and `192.168.12.3`, for example for our cellphone or another PC. Now, the atacker PC has the IP `192.168.12.4` and sniffing the network (in another post i will take about this 8)) detect what IP has access to that proxy. Our pc (has **cntlm**) use the IP `192.168.12.1`.

Now, an atacker can do this with **iptables** that is the standard firewall of linux and can manipulate the network packets:

{% highlight bash linenos %}
iptables -t nat -A POSTROUTING -p tcp --destination-port 3128 -d 192.168.12.1 -j SNAT --to-source 192.168.12.3
{% endhighlight %}

Explaining:

- `-t nat` Use the table nat of iptables, this is the table of routing.
- `-A POSTROUTING` Apply this rule to that packets that's are already routed.
- `-p tcp` Use TCP protocol.
- `--destination-port 3128` The port of the proxy.
- `-d 192.168.12.1` Apply this rule to the packets that's go to that ip `192.168.12.1`, if not, all the packets that will exit from our pc will change the source IP.
- `-j SNAT` This is the acction to apply, in this case, change the source of the packet.
- `--to-source 192.168.12.3` No comments :).
