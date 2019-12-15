---
title:  "FreeBSD Jailhost for VMs"
date:   2019-12-15 08:30:00
description: FreeBSD Jails with VNET Subnets for Virtual Machines
---

In this tutorial, we'll walkthrough the process of setting up a host for hacking targets.  Our host will be based on FreeBSD; we'll use jails to control the individual target accounts; we'll use virtual machines as the targets inside those accounts.  Our goal will be to turn a modest Dell i5 salvaged desktop into a machine that we can hack and reset at will.  

As part of building our target host, we'll want to include some isolating and stabilizing features.  We'll want each target account to be contained in a resettable jail; this way, if there are severe problems with the VM because of the hacking attacks, then we can reset the target by restarting the jail.  Each target will need its own VNET so that we can focus over-the-network attacks on the desired targets without accidentally spilling over into other areas.  We'll want each VM hypervisor to be able to start up about half a dozen virtual machines.  This way we could host them all, or change out targets for variety.  

We'll build up our jails, one by one, step by step.  We'll controll them manually.  We'll operate each virtual machine in a headless fashion using a simple script.  Along the way, we'll cover some of the major topics in virtualization for hosts and networks.  

We couldn't build this project without the advice we found through some important publications.  No hacker's work is their own, and this tutorial is no exception.  For learning about FreeBSD jails, we turned to Michael Lucas' books.  For learning about the headless operation of the VMs, we used Andrea Fortuna's blog posts about Virtualbox.  As general background material on the GreeBSD operating system, we have turned many, many times to the FreeBSD Handbook.  Our desire for hacking targets like these has been fueled by some great labs experienced to be found with Offensive Security's PWK:  Penetration Testing with Kali Linux course.  Citations for these resources and more can be found in the annotated bibliography section.    

Let's get started.

# Hardware
Our target machine will be built on a salvaged Dell Optiplex 990.  Purchased at a government auction for about $50, this salvaged machine has been cleaned, tested, and fitted with a small SSD.  For storage, we'll be using a refurbished Kingston 120GB SSD that sold for about $12.  We've outfitted the box with 8G of 10600U RAM, a dual port network card, and a graphics port that does Display and DVI formats.  The power supply has been tested; there's a DVD drive we'll use for installing the base system ISO.

For our tutorial, we'll rig up our main internet connection to port 0 on the network card.  We'll have to hook up our monitor to the graphics card to meet the BIOS requirements.  We've set the BIOS boot order to let us boot from CD/DVD disk, USB drive, and then the SSD.  

# Phases of Install
Based on a previous prototyping run, we're going to break up our project into some phases so that we can gradually build up our jailhost into a target machine.

- Host base install
- Host ZFS drive partitioning 
- Basic jail install
- VNET setup
- Jail template for VMs
- Jail replication and provisioning

A lot of the decisions we're about to make will be based on our intentions for the system.  For example, when we get to VNET, we'll see that we've already laid out a plan for subnetting and explicitly assigning many IP addresses.  Likewise, we have a corresponding plan for setting up multiple ZFS features.  Combined, the VNET and ZFS will give each VM its own contained communications and storage.  

![Asset plan for SALVAGE13B-COFFEEHOUSE]({{ site.url }}/assets/images/SALVAGE13B-COFFEEHOUSE/assetPlan.png)

Making these decisions can be a lot easier when we're designing a system with a clear task and purpose.  Oftentimes, when we build systems on our own we may not be too sure what we'll do with that machine in the future.  If we were to stumble and flouder our way to concluding this is the type of system we wanted, then we could have a very messy and problematic install.  We're sure to run into problems, but having a clear idea is a great help to us.  We can set goals that will let us build out the box we want.

# Building Our VNET Plan
We will want to have four main jails on the VNET.  Each will carry its own subnet.  Inside those jails, we'll want 6 hosts per net.  After some experimenting with an online CIDR subnet calculator \[[5]\], we've decided on a plan based on 192.168.1.192/27/29.  Robert Eisele's subnet calculator can help us lay out every IP in that net and confirm the relationship of each IP to the network.  What subnet masks we need, which are the first and last hosts in each part of the net, and what addresses are used for the nets and their broadcasts:  those topics are all instantly described with that online tool.  

![Subnet plan for SALVAGE13B-COFFEEHOUSE]({{ site.url }}/assets/images/SALVAGE13B-COFFEEHOUSE/subnetPlan.png)

# Choosing an OS Release for the Host and the Jails
With jails, we must run the jailhost with the youngest version of the operating system.  We can run jails with older versions of the OS, but they can never be younger than what's running on the host.  With this rule in mind, we'll run 12.1 on the jailhost.  We simply go to the website at FreeBSD.org, download 12.1, and burn it to DVD disc using another working computer connected to the Internet.  

Based on past experiments, I found it was helpful to choose the "DVD" version of the FreeBSD installer.  While others may work, the DVD version holds the most data on the installer disk.  When troubleshooting and hunting around for resources, I felt it was most comfortable and familiar.  I think that it's possible that USB memstick and memstick-mini installer versions will work just as well; however, to cut down on the nagging feeling I might be missing something critical during troubleshooting, we chose our most familiar installer:  the dvd.  \[[6]\] \[[7]\] \[[8]\] \[[9]\]


 \[[]\]
## Annotated Bibliography
\[1\]  FreeBSD Foundation.  INTERNET: [`https://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/`](https://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/)

\[2\]  Lucas, Michael W.  Absolute FreeBSD:  The Complete Guide to FreeBSD.  No Starch Press.  ISBN 978-1-59327-892-2.

\[3\]  Lucas, Michael W.  FreeBSD Mastery:  Jails.  Tilted Windmill Press.  ISBN 978-1-64235-024-1.

\[4\]  Offensive Security.  Penetration Testing with Kali Linux.  Laboratory Manual, v. 1.1.6.  Controlled distribution, proprietary manual for commercial certification testing, made available to registered students with Offensive Security course for Penetration Testing with Kali Linux.

\[5\]  Eisele, Robert.  "CIDR Subnet Calculator"  INTERNET: [`https://www.xarg.org/tools/subnet-calculator/?q=192.168.1.192%2F27%2F29`](https://www.xarg.org/tools/subnet-calculator/?q=192.168.1.192%2F27%2F29)
    A free online subnet calculator that uses CIDR.

\[6\]  FreeBSD Foundation.  INTERNET: [`https://www.freebsd.org/releases/12.1R/announce.html`](https://www.freebsd.org/releases/12.1R/announce.html)
    
\[7\]  FreeBSD Foundation.  INTERNET: [`https://www.freebsd.org/where.html`](https://www.freebsd.org/where.html)

\[8\]  FreeBSD Foundation.  INTERNET: [`https://download.freebsd.org/ftp/releases/amd64/amd64/ISO-IMAGES/12.1`](https://download.freebsd.org/ftp/releases/amd64/amd64/ISO-IMAGES/12.1)
   
\[9\]  FreeBSD Foundation.  INTERNET: [`https://download.freebsd.org/ftp/releases/amd64/amd64/ISO-IMAGES/12.1/FreeBSD-12.1-RELEASE-amd64-dvd1.iso`](https://download.freebsd.org/ftp/releases/amd64/amd64/ISO-IMAGES/12.1/FreeBSD-12.1-RELEASE-amd64-dvd1.iso)


   
[//]: # (Hyperlinks)
[1]: https://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/
[2]: ``
[3]: ``
[4]: ``
[5]: https://www.xarg.org/tools/subnet-calculator/?q=192.168.1.192%2F27%2F29
[6]: https://www.freebsd.org/releases/12.1R/announce.html
[7]: https://www.freebsd.org/where.html
[8]: https://download.freebsd.org/ftp/releases/amd64/amd64/ISO-IMAGES/12.1/
[9]: https://download.freebsd.org/ftp/releases/amd64/amd64/ISO-IMAGES/12.1/FreeBSD-12.1-RELEASE-amd64-dvd1.iso


