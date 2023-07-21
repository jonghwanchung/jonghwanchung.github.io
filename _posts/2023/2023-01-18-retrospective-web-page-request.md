---
title: "Retrospective: a day in the life of a web page request"
tags:
  - study
  - computer network
toc: true
toc_sticky: true
toc_label: "Table of Contents"
classes: wide
---


This post covers the `network protocol stack` by taking an integrated view of each layers. It uses a simple example: *downloading a web page from web server to browser*.


## Assumption
The following figure is an initial configuration of network components including end device, router, IPS, and web server. The router is connected to its ISP(comcast.net) which provides DHCP and DNS services for the local network.

![image](https://github.com/jonghwanchung/jonghwanchung.github.io/assets/97339878/7c7ae5b6-22a9-4ae2-8f60-fac068b02799){: .align-center}



## Operations of DHCP (UDP, IP, Ethernet)
When the `end device` is first connected to the local network, it cannot do anything because it has **no IP address**. Thus, it runs the `DHCP protocol`{: style="color: red"} to obtain an `IP address` (including other information, such as subnet mask, first-hop router address, local DNS address) from the local DHCP server. Detailed DHCP interaction process is described in [here](https://datatracker.ietf.org/doc/html/rfc2131). This post starts with DHCP request process.

| Layer | # | Action(s)|
|---|---|---|
| `DHCP client`{: style="color: red"} | (L5) | it creates a [DHCP request message](https://www.ietf.org/rfc/rfc2131.txt){: style="color: red"} and passes it into the L4 (transport layer). |
| Transport layer | (L4) | it encapsulates the DHCP request message with {source port = `68`, dest port = `67`} into a [UDP segment](https://www.ietf.org/rfc/rfc768.txt). |
| Network layer | (L3) | it encapsulates the UDP segment with {source IP = `0.0.0.0`, dest IP = `255.255.255.255`} into an [IP datagram](https://datatracker.ietf.org/doc/html/rfc791). |
| Link Layer | (L2) | it encapsulates the IP datagram with {source MAC = `00:16:d3:23:68:8a`, dest MAC = `ff:ff:ff:ff:ff:ff`} into an [Ethernet frame](https://en.wikipedia.org/wiki/Ethernet_frame).<br> * *It sends the Ethernet frame on its all attached subnets!* (`broadcast`) |


When the `gateway` (router) receives the Ethernet frame (containing the DHCP request message), it is interrupted.

| Layer | # | Action(s)|
|---|---|---|
| Link layer | (L2) | it checks whether the dest MAC address matches its own MAC address. <br>Becuase the dest MAC address is `ff:ff:ff:ff:ff:ff` (broadcast), it extracts the IP datagram, and passes it up to the network layer. |
| Network layer | (L3) | it indexes its `forwarding table` using the dest IP address, and forwards the IP datagram to the proper outgoing link interface. <br>Because the dest IP address is `255.255.255.255` (broadcast), it copies the IP datagram to all its outgoing link interface. |
| Link layer | (L2) | it encapsulates the IP datagram with {source MAC = `00:22:6b:45:1f:1b`(own), desc MAC = `ff:ff:ff:ff:ff:ff`} in to an Ethernet frame. <br> * *It sends the Ethernet frame on its all attached subnets!* (`broadcast`) |


When the `DHCP server` receives the Ethernet frame (containing the DHCP request message), it is interrupted.

| Layer | # | Action(s)|
|---|---|---|
| Link layer | (L2) | it checks whether the dest MAC address matches its own MAC address. <br>Becuase the dest MAC address is `ff:ff:ff:ff:ff:ff` (broadcast), it extracts the IP datagram, and passes it up to the network layer. |
| Network layer | (L3) | it checks whether the dest IP address matches its own IP address. <br>Because the dest IP address is `255.255.255.255` (broadcast), it extracts the UDP segment, and passes it up to the transport layer. |
| Transport layer | (L4) | it extracts the DHCP request message from the UDP segment, and demultiplexes it to the DHCP server using the dest port (`67`). |
| `DHCP server`{: style="color: red"} | (L5) | it *"now"* receives the `DHCP request message`. <br>It creates a [DHCP ACK message](https://www.ietf.org/rfc/rfc2131.txt){: style="color: red"} with {yiaddr = `68.85.2.101`, subnet = `68.85.2.0/24`, default gateway = `68.85.2.1`, local DNS server = `68.87.71.226`}, and passes it into the lower layers.<br> * L4, L3, L2 encapsulate the DHCP ACK message with {source port = `67`, dest port = `68`}, {source IP = `68.85.2.226`, dest IP = `255.255.255.255`}, and {source MAC = `00:22:6b:45:1f:1c`, dest MAC = `00:16:d3:23:68:8a`}, respectively. <br> * *It sends the Ethernet frame on the proper attached subent!* (`unicast`) |


Wehn the `end device` receives the Ethernet frame (containing the DHCP ACK message), it is interrupted.

| Layer | # | Action(s)|
|---|---|---|
| Link layer | (L2) | it checks whether the dest MAC address matches its own MAC address. <br>Because the dest MAC address is `00:16:d3:23:68:8a` (own), it extracts the IP datagram, and passes it up to the network layer. |
| Network layer | (L3) | it checks whether the dest IP address matches its own IP address. <br>Because the dest IP address is `255.255.255.255` (broadcast), it extracts the UDP segment, and passes it up to the transport layer. |
| Transport layer | (L4) | it extracts the `DHCP ACK message` from the UDP segment, and demultiplexes it to the DHCP client using the dest port (`68`). |
| `DHCP client`{: style="color: red"} | (L5) | it *"now"* receives the DHCP ACK message. <br>It assigns the received {yiaddr = `68.85.2.101`, subnet = `68.85.2.0/24`, default gateway = `68.85.2.1`, local DNS server = `68.87.71.226`}. <br> * From now, all messages is transferred via the default gateway to the outside of its subnet! |



## Operations of DNS (ARP)
When the `end device` enters a `URL` (*www.google.com*) into a web browser, the web browser needs to know the **IP address of URL** for creating a TCP socket. At this time, it runs the [DNS protocol](https://datatracker.ietf.org/doc/html/rfc1035){: style="color: red"} to retrieve the `IP address` of the URL.

| Layer | # | Action(s)|
|---|---|---|
| `DNS client`{: style="color: red"} | (L5) | it creates a [DNS query message](https://datatracker.ietf.org/doc/html/rfc1035){: style="color: red"} with the {URL = `www.google.com`}, and passes it into the transport layer. |
| Tansport layer | (L4) | it encapsulates the DNS query message with {source port = `1023`, dest port = `53`} into a UDP segment. |
| Network layer | (L3) | it encapsulates the UDP segment with {source IP = `68.85.2.101` (*newly assigned*), dest IP = `68.87.71.226`} into an IP datagram. |
| link layer | (L2) | it does *"not know"* the MAC address of default gateway. <br>It runs the [ARP protocol](https://en.wikipedia.org/wiki/Address_Resolution_Protocol){: style="color: red"} to retrieve the **MAC address of default gateway**. |


### ARP Operations
The `ARP module` (in end device; L2) creates an [ARP query message](https://en.wikipedia.org/wiki/Address_Resolution_Protocol) with {IP = `68.86.2.1`}, and passes it into the adapter's link layer.
 - The adapter's link layer encapsulates the ARP query message with {source MAC = `00:16:d3:23:68:8a`, desc MAC = `ff:ff:ff:ff:ff:ff`} into an Ethernet frame. <br> * *It sends the Ethernet frame on its all attached subnets! (`broadcast`)*

When the `default gateway` receives the Ethernet frame (containing the ARP query message), it is interrupted.
 - The ARP module (in default gateway; L2) checks whether its ARP table contains the mapping for the requested IP.
    - Because the requested IP is `68.86.2.1` (own), it indexes its ARP table using the requested IP address.
    - It creates an [ARP response message](https://en.wikipedia.org/wiki/Address_Resolution_Protocol) with {IP = `68.86.2.1`, MAC = `00:22:6b:45:1f:1b`}, and passes it into the adapter's link layer.
    - The adapter’s link-layer encapsulates the ARP response message with {source MAC = `00:22:6b:45:1f:1b`, dest MAC = `00:16:d3:23:68:8a`} into an Ethernet frame. <br> * *It sends the Ethernet frame on proper attached subnet! (`unicast`)*

When the `ARP module` (in end device; L2) receives the Ethernet frame (containing the ARP response message), it is interrupted.
 - It updates its `ARP table` using the mapping information of {IP = `68.86.2.1`, MAC = `00:22:6b:45:1f:1b`}.
 - And then, it encapsulates the IP datagram with {source MAC = `00:16:d3:23:68:8a`, destination MAC = `00:22:6b:45:1f:1b`} into an Ethernet frame. <br> * *It sends the Ethernet frame on the proper attached subnet! (`unicast`)*.



## Operations of HTTP (TCP, IP, Ethernet)




## References
- [IETF RFCs](https://www.ietf.org/standards/rfcs/)
