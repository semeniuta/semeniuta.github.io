---
layout: post
title: "Configuring jumbo frames in Ubuntu"
comments: true
permalink: ubuntu-jumbo-frames
---

When working with GigE Vision cameras, it is important to be able to receive Ethernet frames of larger sizes than one gets in the default configuration. In particular, the default value of Maximum Transmission Unit (MTU) is 1500 bytes. By setting this parameter to the maximum (9000 bytes), a video streaming application handles smaller number of frames per second. This results in a decreased CPU load by cutting the number of operations for reconstructing image data from multiple small Ethernet frames.

In Ubuntu such configuration can easily be done by adding `mtu 9000` entry to the end of the respective network adapter configuration. For example, the `enp38s0` adapter with static IP (say `192.168.1.2/24`) can be configured in `/etc/network/interfaces` as follows:

```
auto enp38s0
iface enp38s0 inet static
address 192.168.1.2
netmask 255.255.255.0
mtu 9000
```

To read more on the matter of jumbo frames and their benefits in GigE Vision applications, check out these resources:

[The Promise and Peril of Jumbo Frames](https://blog.codinghorror.com/the-promise-and-peril-of-jumbo-frames/)

[GigE Vision - Jumbo Frames](https://www.stemmer-imaging.co.uk/en/knowledge-base/gige-vision-jumbo-frames/)
