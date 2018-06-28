---
title:  "Recon VulnHub's Bad Store"
date:   2018-06-15 08:30:00
description: Practice Notes on Reconaissance of BadStore.iso
---

In this post, we'll cover adapting some of the recon techniques outlined in Georgia Weidman's book [1] to an unknown set of problems found in VulnHub's "Bad Store" ISO.  Our goal will be to use the VulnHub VM as a target.  We'll find what's possible by doing some scanning and enumeration.  

#### Setup
<img src="https://github.com/gladiola/blog/blob/master/assets/images/KaliBadStore/Diagram_Kali_BadStore.png" height="361" width="436"

#### Summary Properties of the Machines
One box is holding the VulnHub VM; it's running VirtualBox; conducted some simple `ifconfig` and `ping` checks to make sure that it could communicate with other machines on the SOHO network.  Target box is the VM running on a WIN10 laptop.  Attacker box is the aquarium computer; it's running Kali3 in a mineral oil bath with an SSD and a submerged WiFi dongle.  




## Annotated Bibliography
\[1\]  Weidman, Georgia.  "Chapter 4:  Using the Metasploit Framework," Penetration Testing:  A Hands-On Introduction to Hacking.  No Starch Press.  pp.87-109.  ISBN 978-1-89327-564-8.



[//]: # (Hyperlinks)
[1]:  ``
