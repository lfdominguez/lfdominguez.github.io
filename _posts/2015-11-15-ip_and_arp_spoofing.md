---
layout: post
title: IP & ARP spoofing
tags: [ip, arp, spoofing, network, security]
summary: Some concepts and a pen test about the spoofing techniques in IP and ARP.
---

## Introduction

In the post *[ARP Spofing]({{ site.url }}{% post_url 2015-4-20-arp_spoofing %})* i was spoke about the ARP spoofing and how use it to get the network packages sended to another PC. In this post now, i will speak about the same thing, but with the IP, in others words, how to change the source IP of a package.

## IP Spoofing

Into the IP layer of network,

{% ditaa %}

+    +     +     +     +     +     +     +     +
+----------------------------------------------+
| Ver| IHL |  Service  |    Total Length       |
+----------------------------------------------+
| Identification       | F | Fragmention Off   |
+----------------------------------------------+
| Ver| IHL |  Service  |    Total Length       |
+----------------------------------------------+
| Ver| IHL |  Service  |    Total Length       |
+----------------------------------------------+
| Ver| IHL |  Service  |    Total Length       |
+----------------------------------------------+
| Ver| IHL |  Service  |    Total Length       |
+----------------------------------------------+


/----+  DAAP /-----+-----+ Audio  /--------+
| PC |<------| RPi | MPD |------->| Stereo |
+----+       +-----+-----+        +--------+
    |                 ^ ^
    |     ncmpcpp     | | mpdroid /---------+
    +--------=--------+ +----=----| Nexus S |
                                  +---------+
{% endditaa %}
