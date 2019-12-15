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

# Preliminary Planning
## Hardware
Our target machine will be built on a salvaged Dell Optiplex 990.  Purchased at a government auction for about $50, this salvaged machine has been cleaned, tested, and fitted with a small SSD.  For storage, we'll be using two refurbished Kingston 120GB SSDs that sold for about $12/each.  We've outfitted the box with 8G of 10600U RAM, a dual port network card, and a graphics port that does Display and DVI formats.  The power supply has been tested; there's a DVD drive we'll use for installing the base system ISO.

For our tutorial, we'll rig up our main internet connection to port 0 on the network card.  We'll have to hook up our monitor to the graphics card to meet the BIOS requirements.  We've set the BIOS boot order to let us boot from CD/DVD disk, USB drive, and then the SSD.  

## Phases of Install
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

## Building Our VNET Plan
We will want to have four main jails on the VNET.  Each will carry its own subnet.  Inside those jails, we'll want 6 hosts per net.  After some experimenting with an online CIDR subnet calculator \[[5]\], we've decided on a plan based on 192.168.1.192/27/29.  Robert Eisele's subnet calculator can help us lay out every IP in that net and confirm the relationship of each IP to the network.  What subnet masks we need, which are the first and last hosts in each part of the net, and what addresses are used for the nets and their broadcasts:  those topics are all instantly described with that online tool.  

![Subnet plan for SALVAGE13B-COFFEEHOUSE]({{ site.url }}/assets/images/SALVAGE13B-COFFEEHOUSE/subnetPlan.png)

## Choosing an OS Release for the Host and the Jails
With jails, we must run the jailhost with the youngest version of the operating system.  We can run jails with older versions of the OS, but they can never be younger than what's running on the host.  With this rule in mind, we'll run 12.1 on the jailhost.  We simply go to the website at FreeBSD.org, download 12.1, and burn it to DVD disc using another working computer connected to the Internet.  

Based on past experiments, I found it was helpful to choose the "DVD" version of the FreeBSD installer.  While others may work, the DVD version holds the most data on the installer disk.  When troubleshooting and hunting around for resources, I felt it was most comfortable and familiar.  I think that it's possible that USB memstick and memstick-mini installer versions will work just as well; however, to cut down on the nagging feeling I might be missing something critical during troubleshooting, we chose our most familiar installer:  the dvd.  \[[6]\] \[[7]\] \[[8]\] \[[9]\]

# Host Base Install
## Installing FreeBSD Onto the Hardware
With our disk of a FreeBSD ISO on hand, we'll drop it into the target machine and let the installer take the lead.  We'll make choices that allow for the download of base, src, and ports files.  We'll allow the installer to make the usual choices for our machine  \[[10]\].  We'll choose a ZFS encrypted install, to accept every secure option in the installer dialog for hardening, and to download a copy of the FreeBSD handbook.  We'll name our machine SALVAGE13B-COFFEEHOUSE.  

All of these are mostly personal choices that won't have a bearing on the features we'll add on later.  However, we're going to make careful note of account names and passphrases that are decided during our install.  We'll make a root account and a user account called "barista".  We'll add barista to our wheel and operator groups.  This will give us an alternate to root that we can use with ssh later.  With the ZFS GELI encryption for the disk, we'll emerge from this phase with three critical passwords recorded.  

## Initial checks
With the system installed, one of the first things we'll do is check those passphrases.  Make sure each account works.  Any problems with those could complicate or confound anything we do after this.

We'll do a quick update of the FreeBSD installation and ports  \[[11]\] \[[12]\].
- freebsd-update fzfszpooetch
- freebsd-update install
- portsnap fetch
- portsnap extract

We'll install nano, one of my favorite text editors.

`cd /usr/ports/editors/nano`
`make -DBATCH install clean`

The `-DBATCH` helps us avoid all of the pauses in the ncurses dialogs for configuring the downloads and compilations.  But, if there is difficulty, we'll have to delve into `make config`, see what options are presented; maybe we'll also have to hop into the ports that threw the errors, and then recompile those individually.  Sometimes we might have to do a `pkg install <PORTNAME>` to substitute the compiliation of a program with a strait binary install.  With those troubleshooting tasks under our belt, we'll continue with our project.

To check our current ZFS configuration:
`zfs list`
`zpool list`

To take a snapshot some datasets at this stage of installation:
`zfs snapshot zroot/ROOT/default@<SOME_DATE_AND_TIME>`
`zfs snapshot zroot/usr@<SOME_DATE_AND_TIME>`

\[[13]\]

## Carve Up Some ZFS Datasets
To later accomodate our jails, we'll want to create some ZFS datasets.  Combined with ZFS snapshot procedures, we'll then be able to make snapshots of our progress (or regress) when constructing the jails.  Since there's nothing really on the disk right now, we can declare the datasets with no worry.  We won't need to specify how much space each one takes up.  We just need to describe how we'll carve up the data.

To lay the foundation, we'll give:
`zfs create -o canmount=off zroot/jail`

To lay out the main divisions of the jail directories, we'll give:
`zfs create zroot/jail/coffeehouse`
`zfs create zroot/jail/redeye`
`zfs create zroot/jail/cappucino`
`zfs create zroot/jail/latte`

Within each of those, we'll lay out six subdirectories, for the VMs we'll install later.  Typically, we'll give:
`zfs create zroot/jail/coffeehouse/vm1`
`zfs create zroot/jail/coffeehouse/vm2`
... and so on, until we have given six subdirectories for each of the four jail directories.  

To see our progress, we can:
`zfs list` and it will show us all of the ZFS datasets.

We'll want to associate each of those datasets with a group, for user access control.  We can create groups for each of the four jail directories by using commands like:

`pw groupadd coffeehouse`
`pw groupmod coffeehouse -m barista`

... and so on, until we have given our barista account access to all four of the groups.  We should build out those directories, and we can assign those groups as owners of the datasets with commands like:

`mkdir /jail/coffeehouse`
`chown -R barista:coffeehouse /jail/coffeehouse`

We can see our system is unified with our barista account owning the four directories with each under its own group.  

 \[[]\]
## Annotated Bibliography
\[1\]  FreeBSD Foundation.  INTERNET: [`https://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/`](https://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/)

\[2\]  Lucas, Michael W.  Absolute FreeBSD:  The Complete Guide to FreeBSD.  No Starch Press.  ISBN 978-1-59327-892-2.

\[3\]  Lucas, Michael W.  FreeBSD Mastery:  Jails.  Tilted Windmill Press.  ISBN 978-1-64235-024-1.

\[4\]  Offensive Security.  Penetration Testing with Kali Linux.  Laboratory Manual, v. 1.1.6.  Controlled distribution, proprietary manual for commercial certification testing, made available to registered students with Offensive Security course for Penetration Testing with Kali Linux.

\[5\]  Eisele, Robert.  "CIDR Subnet Calculator"  INTERNET: [`https://www.xarg.org/tools/subnet-calculator/?q=192.168.1.192%2F27%2F29`](https://www.xarg.org/tools/subnet-calculator/?q=192.168.1.192%2F27%2F29)

A free online subnet calculator that uses CIDR.

\[6\]  FreeBSD Foundation.  INTERNET: [`https://www.freebsd.org/releases/12.1R/announce.html`](https://www.freebsd.org/releases/12.1R/announce.html)

Information about the release.
    
\[7\]  FreeBSD Foundation.  INTERNET: [`https://www.freebsd.org/where.html`](https://www.freebsd.org/where.html)

\[8\]  FreeBSD Foundation.  INTERNET: [`https://download.freebsd.org/ftp/releases/amd64/amd64/ISO-IMAGES/12.1`](https://download.freebsd.org/ftp/releases/amd64/amd64/ISO-IMAGES/12.1)
   
Page that allows us to choose the type of image to download.   

\[9\]  FreeBSD Foundation.  INTERNET: [`https://download.freebsd.org/ftp/releases/amd64/amd64/ISO-IMAGES/12.1/FreeBSD-12.1-RELEASE-amd64-dvd1.iso`](https://download.freebsd.org/ftp/releases/amd64/amd64/ISO-IMAGES/12.1/FreeBSD-12.1-RELEASE-amd64-dvd1.iso)

Chosen download for our host install.

\[10\]  FreeBSD Foundation.  INTERNET: [`https://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/using-bsdinstall.html`](https://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/using-bsdinstall.html)

Directions for using bsdinstall.

\[11\]  FreeBSD Foundation.  INTERNET: [`https://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/updating-upgrading-freebsdupdate.html`](https://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/updating-upgrading-freebsdupdate.htmll)

Using freebsd-update and related commands to install security updates.

\[12\]  FreeBSD Foundation.  INTERNET: [`https://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/ports-using.html`](https://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/ports-using.html)

Using portsnap fetch and related commands to install fresh scripts for ports.  This may help ensure that the latest security patches are available to programs compiled from ports.

\[13\] Lucas.  \[2\], pp. 258, 264, 271-2.

\[14\] Lucas.  \[2\], pp. 262.

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
[10]: https://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/using-bsdinstall.html
[11]: https://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/updating-upgrading-freebsdupdate.html
[12]: https://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/ports-using.html
[13]: ``
[14]: ``
[15]: 


