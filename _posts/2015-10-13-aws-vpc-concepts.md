---
layout: post
title: "Amazon VPC - Conquering the Virtual Fort"
date: 2015-10-13
author: Pradeep Aradhya
tags: aws vpc cloud iaas security networking
category: tutorial
excerpt: "This is a Amazon VPC tutorial, which provides information about your virtual private cloud (VPC) and its subnets. Know more about your infrastructure."
---


This is a Amazon VPC tutorial, which provides information about your virtual private cloud (VPC) and its subnets.

## [Amazon VPC][6] 
`Virtual Private Cloud` is your way of controlling the networking layer on which your infrastructure run. It is a virtual network and logically linked to your AWS account.


`Subnets` are basic building blocks, logical subdivision of a VPC which is used in logical grouping like environments or layers of an application (like web, app and DB). Every other service like Route Table, Internet Gateway etc facilitates subnets to construct a VPC. 

__Let's have a closer look__:

- `Route Table:` A set of rules, which controls routing for the subnet
- `Internet Gateway:` Connects your VPC to the internet 
- `Network ACL's:` They act as a firewall, controlling traffic in and out of a subnet. They are stateless
- `Security Groups:` They act as firewall for an instance
- `DHCP Option Set:` It takes care of assigning the IP address and Internal DNS for the VPC 


As subnets are the basic building block. lets dig more into subnets. So, what are subnets? Let me answer in a word

        > **Subnet ~= VLAN**

**What is VLAN?** "VLAN is a group of devices on one or more LANs that are configured to communicate as if they were attached to the same wire, when in fact they are located on a number of different LAN segments. Because VLANs are based on logical instead of physical connections, they are extremely flexible."  -- **Cisco**

VLAN's are used in [OSI][7] Layer-2 (Data Link layer) to isolate in a network. Subnets are used to do the same. 

So, what is VPC ?
        
        > **VPC ~= VRF** (virtual routing and forwarding)

## [Virtual routing and forwarding][8] 

`VRF` is a technology that allows multiple instances of a routing table to co-exist within the same router at the same time. Because the routing instances are independent, the same or overlapping IP addresses can be used without conflicting with each other.
All the ISP providers and Data Centre uses this technology. 

In english, VRF is same thing as VLAN. But, for OSI layer-3. 

VPC is almost works like VRF. However, it uses a proprietary service called **"Mapping Service"** which acts as an anchor in mapping VPC + Instance IP to Physical Server. This service is not exposed directly to customer.


## Let's understand the Mapping Service concept in detail

Let's see how a node in a network talks to other node.

> Let's say for example, `10.0.1.5` wants to talk to `10.0.1.8`. 

![L2-Ethernet][9] 

Following steps takes place in knowing each other

- `10.0.0.5` broadcasts an ARP request for `10.0.0.8`
- Ethernet Switch broadcasts that ARP request to everyone connected to that switch.
- `10.0.0.8` responds back with its MAC address
- Ethernet Switch will forward that MAC address to `10.0.0.5`
- Now, `10.0.0.5` knows where is `10.0.0.8` and can send the packets to same. 

**Now, let's replicate the process in VPC.** 

Amazon got their physical servers interconnected with physical network, on those servers are the instances grouped together as VPC's with their vpc-id's (here we are using colours for ease of understanding). And there is a Mapping Service. These are the basic building blocks.


![L2-VPC][10] 

Following steps occurs in the communication:

- `10.0.0.5` broadcasts an ARP request for Orange (vpc-id) `10.0.0.8`
- The physical server forwards the request to Mapping Service with its physical IP address querying for Orange `10.0.0.8`
- Mapping Service responds with the MAC address of Orange `10.0.0.8` with the physical IP address of the residing server
- `10.0.0.5` Physical Server responds back to the instance with the `10.0.0.8` MAC address
- As `10.0.0.5` knows the MAC address of `10.0.0.8`; it encapsulate the packet with L2 Source and Destination and L3 Source and Destination address and sends
- Mapping Service captures that packet and wrap it with a VPC header (here it is Orange) and wrap it with physical network IP header and send the packet directly to the destination host

![L2-VPC2][11] 

- On receiving the packet, physical server (here `192.1698.2.4`) asks Mapping Service about the authenticity of the origin of the packet. 
- Once the authentication is complete, the packet is delivered to the destination `10.0.0.8` instance. 
- If by any chance authentication fails; the packet is dropped.


The above scenario is for the two instances within a subnet communication. That is Layer-2. How does packets travel between subnets (Layer-3). 
Lets first see, how packets travel in physical network.

> In the scenario `10.0.0.5` wants to talk to `10.0.1.8`. Let me skim through this scenario

![L3-IP_Routing][12]

Following are the steps in physical network:

- `10.0.0.5` broadcasts for `10.0.0.1` (router of its network), Switch responds back with the MAC address of `10.0.0.1`
- Now, `10.0.0.5` sends the packet with L2 Source (itself) and Destination (MAC address of `10.0.0.1`) and L3 Source (`10.0.0.5`) and Destination(10.0.1.8) address. Because Ethernet Switches forwards packets based on MAC address only. 
- `10.0.0.1` (Router) swaps the destination MAC address of `10.0.1.8` and sends the packet to the correct switch.
- Appropriate switch delivers the packet to `10.0.1.8`


**Now lets put the same analogy in VPC**

- `10.0.0.5` broadcasts an ARP request for the default router MAC address
- Mapping Service sends back MAC address of the router to `10.0.0.5`
- Now `10.0.0.5` broadcasts an ARP request for Layer-3 IP address of `10.0.1.8`
- Mapping service responds back the MAC address of Orange 10.0.1.8 with the IP address of the physical server it resides.
- Now, `10.0.0.5` encapsulate the packet with L2 source as its own MAC address and L2 destination as routers MAC address and L3 source as its own and L3 destination as `10.0.1.8`
- Mapping Service captures that packet and wrap it with a VPC header (here it is Orange) and wrap it with physical network IP header and send the packet directly to the destination host

![L3-VPC][13] 

- Once the packet is revived by the physical destination host; it does the exact same authentication. On successful authentication, it perform the MAC swaping and delivers the packet to the destination instance  `10.0.1.8`

![L3-VPC2][14] 

> **NOTE:** Only condition for the above both VPC L2 and L3 scenarios to work is that both source and destination physical servers to be online and can talk to each other and can talk to the MApping Service, they are good to exchange the packets across the router. 


Now you have a high level overview of how Amazon VPC works. This helps in configuring your VPC efficiently.


### There are many VPC services we can take advantage of


- Multiple connectivity options - NAT, Customer Gateways, Virtual Private Gateways, VPN Connections and Direct Connect
- VPC Flow Logs - Its a VPC feature that is used in capturing IP traffic in and out from network interfaces from one's VPC
- VPC ENDPoints - It enables one to create private connection between VPC and other AWS services without using internet or any other VPC service like NAT, Direct Connect or VPN 
- Resource Grouping - As every thing resides inside a VPC, tagging  becomes an important part of VPC
- Private DNS - Its a Route 53 feature, configured hold the information about how one wants to route traffic for a domain or sub-domains within one or more VPC


**In coming days, we will dig into every one of these VPC services and will see how these services can be leveraged.**

Please leave your comments below if you have any doubts or questions.

#####Need DevOps help? - Get in touch with [The Remote Lab][1] 
[LinkedIn][2] [Facebook][3] [Github][4] [Twitter][5]

Credits: [Amazon VPC][6], [Amazon Blog][15], [AWS re:Invent][16]

_Pradeep is DevOps extraordinaire. He's an AWS expert and loves sharing his knowledge. This is a guest post by him._


  [1]: http://theremotelab.io
  [2]: https://www.linkedin.com/company/the-remote-lab
  [3]: https://www.facebook.com/TheRemoteLab
  [4]: https://github.com/TheRemoteLab
  [5]: https://twitter.com/TheRemoteLab
  [6]: http://aws.amazon.com/vpc/
  [7]: https://en.wikipedia.org/wiki/OSI_model
  [8]: https://en.wikipedia.org/wiki/Virtual_routing_and_forwarding
  [9]: https://s3-ap-southeast-1.amazonaws.com/trl-blog/L2-Ethernet.png 
  [10]: https://s3-ap-southeast-1.amazonaws.com/trl-blog/L2-VPC.png
  [11]: https://s3-ap-southeast-1.amazonaws.com/trl-blog/L2-VPC2.png
  [12]: https://s3-ap-southeast-1.amazonaws.com/trl-blog/L3-IP_Routing.png
  [13]: https://s3-ap-southeast-1.amazonaws.com/trl-blog/L3-VPC.png
  [14]: https://s3-ap-southeast-1.amazonaws.com/trl-blog/L3-VPC2.png
  [15]: https://aws.amazon.com/blogs/aws/
  [16]: https://www.portal.reinvent.awsevents.com
