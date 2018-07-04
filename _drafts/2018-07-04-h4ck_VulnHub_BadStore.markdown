---
title:  "Hacking Vulnhub's \"Bad Store\""
date:   2018-07-04 08:30:00
description: Notes on Hacking Vulnhub's "Bad Store" VM
---

In this post, we'll cover adapting some of the recon techniques outlined in Georgia Weidman's book \[1\]\[2\]\[3\] to an unknown set of problems found in VulnHub's "Bad Store" ISO. \[[4]\]  Our goal will be to use the VulnHub VM as a target.  We'll find what's possible by doing some scanning and enumeration. From there, we'll look at some exploits.  

### Spoilers
<blockquote>
    Throughout this post, we'll reveal some aspects of the "Bad Store" target.  Please note that by looking at some of these details, the VM's instructional quality as a target of unknown composition might be diminished to hackers who will want use the target for technical growth and professional development.  Experience recommends thorough use of the target VM before referring to resources such as these.
</blockquote>

Published in 2004, "Bad Store" \[[4]\] is one of the oldest VMs available for these kinds of penetrations.  The VM target contains a poorly secured storefront website running on a Linux system with a MySQL database and some CGI programming.  The age of the target is significant in that we see much less CGI programming than we used to; also, we will see that the passing of time has revealed new vulnerabilities in the technologies used in the VM that may not have been a part of the initial exercises.

### Setup
<img src="https://github.com/gladiola/blog/blob/master/assets/images/KaliBadStore/Diagram_Kali_BadStore.png" height="361" width="436">

### Summary Properties of the Machines
One box is holding the VulnHub VM; it's running VirtualBox; conducted some simple `ifconfig` and `ping` checks to make sure that it could communicate with other machines on the SOHO network.  Target box is the VM running on a WIN10 laptop.  On the attacker box, we are using FreeBSD 10.3 and a Kali VM.

<table>
    <caption>Initial Commo Checks and Basic Configuation</caption>
    <tr>
    <th>Property</th>
    <th>Value</th>
    <th>Logical Impact</th>
    </tr>
    <tr><th>Target Machine:  VirtualBox VM</th><td>VulnHub Bad Store iso in VM</td><td>Target of unknown composition</td></tr>
    <tr><th>Target Machine:  Host OS</th><td>WIN 10</td><td>Target platform host OS</td></tr>
    <tr><th>Attacker Machine:  VirtualBox VM</th><td>Kali</td><td>Attack platform inside VM</td></tr>
    <tr><th>Attacker Machine:  Host OS</th><td>FreeBSD 10.3 RELEASE</td><td>Attack platform host OS</td></tr>
    <tr><th>Target</th><td>192.168.1.9</td><td><code>ifconfig</code> OK</td></tr>
    <tr><th>Attacker</th><td>192.168.1.10</td><td><code>ifconfig</code> OK</td></tr>
</table>

## Proving COMMO
Since we don't have a need for stealth and do have direct access to the controls of both VMs, we'll start off with a simple commo check between the boxes.  By using `ping -c 5 <DESTINATION IP ADDRESS>` and `ifconfig` on both VMs, we could reasonably see that the boxes were communicating.  

If communication among the boxes cannot be established, then a common point to check is the VM network adapter.  Another similar troubleshooting check will be to see if a command line or terminal on the host system can communicate with the VM, and vice versa.  

These commo checks may seem elementary; but, knowing IP addresses and having smooth commo among the boxes can be of help when exporting data.  Kali, in the configuration I was using, did not have an `ftp` command in the terminal.  Also, the FreeBSD version I was using did not have a simple `automount` So, to get files into and out of the attack box, I used FTP uploads through an intermediary website, PuTTY, `ssh`, and `scp`.  Since those details are particular to my setup, I won't cover most of those here.  Meanwhile, in most situations where I have had to use VMs on a variety of systems, being proficient with those kinds of file transfers was a handy skill to have.

## Kill Chains and Attacking Actions
Throughout our discussion, we'll try to relate the use of commands in Kali to analysis actions that use the Lockheed-Martin Intrusion Kill Chain.  Brotherston and Berlin, writing in the Defensive Security Handbook, presented an example use case in this format that allowed us to see all sides of the attack. \[[5]\] They included defensive actions and monitoring in correlations with phases of the kill chain.  Their chart headers looked similar to the one below.

<table>
    <caption>Brotherston-Berlin Use Case Format [5]</caption>
    <thead>
       <tr><th>Kill Chain Step</th><th>Malicious Action</th><th>Defensive Mitigation</th><th>Potential Monitoring</th></tr> 
    </thead>
</table>

Our needs here are a little different, so we'll modify our chart while following along from their example.  Since this is not a real attack; and since our view is mainly from the penetrator's perspective, we'll abbreviate a summary of some of the steps by relating an attacking action to a step in the kill chain and an observation, using tables like the one below.

<table>
    <caption>Command to Kill Chain Step Summary</caption>
    <thead>
       <tr><th>Kill Chain Step</th><th>Attacking Action</th><th>Observation</th></tr> 
    </thead>
    <tbody>
    <tr><th>Reconnaissance</th><td></td><td></td></tr>
    <tr><th>Weaponization</th><td></td><td></td></tr>
    <tr><th>Delivery</th><td></td><td></td></tr>
    <tr><th>Exploitation</th><td></td><td></td></tr>
    <tr><th>Installation</th><td></td><td></td></tr>
    <tr><th>Command and Control</th><td></td><td></td></tr>
    <tr><th>Actions and Objectives</th><td></td><td></td></tr>
    </tbody>
</table>

Later on, I would like to see if we can actually make modifications to repair, modify, or defend that VM, after our penetrations.  For now, that goal will have to remain part of our ambitions, as we work through getting in to the box and learning about the website that's covered in the VM.

## Vulnerability Scanning "Bad Store"
To support a Reconnaissance phase, we conducted several vulnerability scans.  Two were Nessus scans (basic network and web applications); one was a collection on `nmap` scans of TCP and UDP protocol ports; another was a `nikto` scan.  

We also did some manual observation of the website (directory traversal and simple injection probing), and a `sqlmap` scan; those will be covered separately.      


## Annotated Bibliography
\[1\]  Weidman, Georgia.  "Chapter 4:  Using the Metasploit Framework," Penetration Testing:  A Hands-On Introduction to Hacking.  No Starch Press.  pp.87-109.  ISBN 978-1-89327-564-8.

Review of techniques to implement a simple attack with default settings with a metasploit module using the <code>msfconsole</code>. 

\[2\]  Weidman, Georgia.  "Chapter 6:  Finding Vulnerabilities," Penetration Testing:  A Hands-On Introduction to Hacking.  No Starch Press.  pp.133-153.  ISBN 978-1-89327-564-8.

Review of techniques to implement Nessus scans, <code>nmap</code> scans, and <code>nikto</code> scans. 

\[3\] Weidman, Georgia.  "Chapter 14:  Web Application Testing," Penetration Testing:  A Hands-On Introduction to Hacking.  No Starch Press.  pp.313-338.  ISBN 978-1-89327-564-8.

Review of techniques to implement substitution of values sent with Burp Suite, using <code>sqlmap</code> for SQLi, and understanding local and remote file inclusion attacks.

\[4\]  _____.  "Bad Store 1.2.3," Vulnhub.  INTERNET:  [`https://www.vulnhub.com/entry/badstore-123,41/`](https://www.vulnhub.com/entry/badstore-123,41/) 

Website listing details of the VM on Vulnhub.  Includes screenshots and links to download URLs.

\[5\]  Brotherston, Lee and Amanda Berlin.  Defensive Security Handbook:  Best Practices for Securing Infrastructure.  O'Reilly, April 2017. pp. 6-9. ISBN 978-1-491-96038-7. 

A description of Lockheed-Martin's Intusion Kill Chain.  Brotherston and Berlin applied their use case of an overview of a ransomware attack to the phases of Lockheed's kill chain.  Their definitions and example serves as guidance for the contstruction of our own, here.


## Spare Parts
<table>
    <tr><td></td></tr>
</table>

<table>
    <thead>
       <tr><th>Kill Chain Step</th><th>Malicious Action</th><th>Defensive Mitigation</th><th>Potential Monitoring</th></tr> 
    </thead>
    <tbody>
    <tr><th></th><td></td><td></td><td></td></tr>
    </tbody>
</table>

<table>
    <thead>
       <tr><th>Kill Chain Step</th><th>Attacking Action</th><th>Observation</th></tr> 
    </thead>
    <tbody>
    <tr><th>Reconnaissance</th><td></td><td></td></tr>
    <tr><th>Weaponization</th><td></td><td></td></tr>
    <tr><th>Delivery</th><td></td><td></td></tr>
    <tr><th>Exploitation</th><td></td><td></td></tr>
    <tr><th>Installation</th><td></td><td></td></tr>
    <tr><th>Command and Control</th><td></td><td></td></tr>
    <tr><th>Actions and Objectives</th><td></td><td></td></tr>
    </tbody>
</table>

[//]: # (Hyperlinks)
[1]:  ``
[2]: ``
[3]: ``
[4]:  https://www.vulnhub.com/entry/badstore-123,41/
