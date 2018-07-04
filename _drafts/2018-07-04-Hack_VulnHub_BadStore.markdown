---
title:  "Hacking Vulnhub's Bad Store"
date:   2018-07-04 08:30:00
description: Notes on Hacking Vulnhub's "Bad Store" VM
---

In this post, we'll cover adapting some of the recon techniques outlined in Georgia Weidman's book \[1\]\[2\]\[3\] to an unknown set of problems found in VulnHub's "Bad Store" ISO.  Our goal will be to use the VulnHub VM as a target.  We'll find what's possible by doing some scanning and enumeration. From there, we'll look at some exploits.  

<blockquote>
    ### Spoilers
    Throughout the post, we'll reveal some aspects of the "Bad Store" target.  Please note that by looking at some of these details, the VM's instructional quality as a target of unknown composition might be diminished to hackers who use the target for technical growth and professional development.  Experience recommends thorough use of the taget VM before referring to resources such as these.
</blockquote>

Published in 2004, "Bad Store" is one of the oldest VMs available for these kinds of penetrations.  The VM target contains a poorly secured storefront website running on a Linux system with a MySQL database and some CGI programming.  The age of the target is significant in that we see much less CGI programming than we used to; also, we will see that the passing of time has revealed new vulnerabilities in the technologies used in the VM that may not have been a part of the initial exercises.

#### Setup
<img src="https://github.com/gladiola/blog/blob/master/assets/images/KaliBadStore/Diagram_Kali_BadStore.png" height="361" width="436">

#### Summary Properties of the Machines
One box is holding the VulnHub VM; it's running VirtualBox; conducted some simple `ifconfig` and `ping` checks to make sure that it could communicate with other machines on the SOHO network.  Target box is the VM running on a WIN10 laptop.  On the attacker box, we are using FreeBSD 10.3 and a Kali VM.

<table>
    <caption>Initial Default Values Provided</caption>
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

Had a pretty good session.  

## Annotated Bibliography
\[1\]  Weidman, Georgia.  "Chapter 4:  Using the Metasploit Framework," Penetration Testing:  A Hands-On Introduction to Hacking.  No Starch Press.  pp.87-109.  ISBN 978-1-89327-564-8.

Review of techniques to implement a simple attack with default settings with a metasploit module using the <code>msfconsole</code>. 

\[2\]  Weidman, Georgia.  "Chapter 6:  Finding Vulnerabilities," Penetration Testing:  A Hands-On Introduction to Hacking.  No Starch Press.  pp.133-153.  ISBN 978-1-89327-564-8.

Review of techniques to implement Nessus scans, <code>nmap</code> scans, and <code>nikto</code> scans. 

\[3\] Weidman, Georgia.  "Chapter 14:  Web Application Testing," Penetration Testing:  A Hands-On Introduction to Hacking.  No Starch Press.  pp.313-338.  ISBN 978-1-89327-564-8.

Review of techniques to implement substitution of values sent with Burp Suite, using <code>sqlmap</code> for SQLi, and understanding local and remote file inclusion attacks.




[//]: # (Hyperlinks)
[1]:  ``
[2]: ``
[3]: ``
[4]:  
