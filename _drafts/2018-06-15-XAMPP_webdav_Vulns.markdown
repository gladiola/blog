---
title:  "Hacking XAMPP WEBDAV Vulns"
date:   2018-06-15 08:30:00
description: Exercising Known Vulns in XAMPP for Pentesting Practice
---


## Abstract
The goal is to hack into a box that is running a Windows VM that's holding an older version of XAMPP with known vulns.  We'll target webdav so that we can use a default metasploit module.  Both the attacker box and target box are in the same physical room on the same net, addressed by the same router.  With user-side access to both machines, they are available to troubleshoot.  When the initial hacks were attempted, the hack failed.  These notes are about why the hack failed, what was done to fix the lab, and what was discovered along the way.

#### Setup
<img src="https://github.com/gladiola/blog/blob/master/assets/images/XAMPPKali/Diagram_XAMPP_Kali.png" height="430" width="576">

#### Summary Properties of the Machines
Both boxes are running VMs with VirtualBox.  On the attacker box, we are using FreeBSD 10.3 and a Kali VM.  Since the box is FreeBSD, the usual sharing tools are not available.  Since I've used this VM to attack another VM on the same box before, I am aware that it can work.  On the target box, we are running WIN10 with VirtualBox running a Microsoft VM for WIN7 Enterpreise with IE11.  Inside that VM, I have installed XAMPP 1.6.0a.  

<table>
    <caption>Troubleshooting:  Initial Default Values Provided</caption>
    <tr>
    <th>Property</th>
    <th>Value</th>
    <th>Logical Impact</th>
    </tr>
    <tr><th>XAMPP Version</th><td>1.6.0a</td><td>Vuln to known CVEs</td></tr>
    <tr><th>Target Machine:  VirtualBox VM</th><td>Windows 7 with IE11 VM</td><td>Target platform simulated OS</td></tr>
    <tr><th>Target Machine:  Host OS</th><td>WIN 10</td><td>Target platform host OS</td></tr>
    <tr><th>Attacker Machine:  VirtualBox VM</th><td>Kali</td><td>Attack platform inside VM</td></tr>
    <tr><th>Attacker Machine:  Host OS</th><td>FreeBSD 10.3 RELEASE</td><td>Attack platform host OS</td></tr>
    <tr><th>Metasploit Module</th><td>xampp_webdav_upload_php</td><td>Known working exploit</td></tr>
    <tr><th>RHOST</th><td>192.168.1.9</td><td>Target address</td></tr>
    <tr><th>RPORT</th><td>80</td><td>Target port</td></tr>
    <tr><th>LHOST</th><td>192.168.1.10</td><td>Listen address</td></tr>
    <tr><th>LPORT</th><td>4444</td><td>Listen port</td></tr>
    <tr><th>USER</th><td>wampp</td><td>Default user for auth</td></tr>
    <tr><th>PASSWORD</th><td>xampp</td><td>Default password for auth</td></tr>
</table>

## Metasploit Attack Tools and The Problem
Based on the Exploit-db entry, and a review of the Metasploit module code [7], it looked like XAMPP would be setup with default credentials and be in use automatically.  A later review of the setup for webdav on a box [6] suggested that the tools for webdav might be installed, but the service might not necessarily be running if it was not fully enabled.  Meanwhile, there were tutorials that showed the exploit running in a default config [5] with no trouble.  The trouble I observed was that when the `exploit` command was given, the file would not upload.  It was as if a `201` was the HTTP Status given and received by the module code.  [7]

The questions that were developed were:
- Was the problem with the XAMPP webdav?
- Was the signal getting through to the target?
- Was there a permissions conflict?
- Did XAMPP automatically enable webdav in the target directory?
- Did we know that webdav was working on the target?
- Were there problems with the way the Metasploit module was being used?
- Can we prove that the attack box could get near the target files?
- Why was the exploit file not being uploaded to target as in the examples?



## Annotated Bibliography
\[1\] _____.  INTERNET: [`https://sourceforge.net/projects/xampp/files/XAMPP%20Windows/`](https://sourceforge.net/projects/xampp/files/XAMPP%20Windows/) 

Sourceforge download page for XAMPP versions.  Using these hyperlinks, I navigated to the various version files; I downloaded a few older versions with known vulns for experimentation.

\[2\] _____.  INTERNET: [`https://www.cvedetails.com/cve/CVE-2007-2080/`](https://www.cvedetails.com/cve/CVE-2007-2080/)Offensive Security, Exploit Database.

This CVE outlined "multiple SQL injection vulnerabilities in XAMPP 1.6.0a for Windows."  Given that, and other CVEs that followed later in history, this one was a good version for use as a target.

\[3\] _____.  INTERNET: [`https://www.exploit-db.com/exploits/37396/`](https://www.exploit-db.com/exploits/37396/) Offensive Security, Exploit Database.

This Expolit-db entry showed some XSS vulns that I thought I could practice with in 1.6.0a.

\[4\] _____.  INTERNET: [`https://www.exploit-db.com/exploits/3738/`](https://www.exploit-db.com/exploits/3738/) Offensive Security, Exploit Database.

Exploit-db entry shows a remote buffer overflow that will probably overrun a MS Sql Server database and tell us users, passwords, and some schema information.

\[5\] Chandel, Raj.  "How to Hack XAMPP of Remote PC using Metasploit," Raj Chandel's Blog: Hacking Articles.  INTERNET: [`http://www.hackingarticles.in/how-to-hack-xampp-of-remote-pc-using-metasploit/`](http://www.hackingarticles.in/how-to-hack-xampp-of-remote-pc-using-metasploit/)

This exploit tutorial shows XAMPP being used with a Metasploit module to establish communications with the target computer.  

\[6\] _____.  INTERNET: [`https://www.mkyong.com/apache/how-to-enable-webdav-in-apache-server-2-2-x-windows/`](https://www.mkyong.com/apache/how-to-enable-webdav-in-apache-server-2-2-x-windows/)

Sample directions on the proper use and setup of webdav file sharing procedures.  If we were going to use XAMPP to implement webdav, then we would probably initiate procedures like these, with XAMPP holding the webdav directory.

\[7\] _____.  "XAMPP - WebDAV PHP Upload (Metasploit),"  INTERNET: [`https://www.exploit-db.com/exploits/18367/`](https://www.exploit-db.com/exploits/18367/) Offensive Security, Exploit Database.

Source code of the Metasploit module that I used as I was working against XAMPP 1.6.0a.

\[8\] _____.  "mssql_connect," PHP Man.  INTERNET:  [`http://php.net/manual/en/function.mssql-connect.php`](http://php.net/manual/en/function.mssql-connect.php) 

PHP Manual page on the proper use of the mssql_connect command that is associated with one of the CVEs.

\[9\] Weidman, Georgia.  "Chapter 4:  Using the Metasploit Framework," Penetration Testing:  A Hands-On Introduction to Hacking.  No Starch Press.  pp.87-109.  ISBN 978-1-89327-564-8.

Review of techniques to implement a simple attack with default settings with a metasploit module using the msfconsole.  

[//]: # (Hyperlinks)
[1]: https://sourceforge.net/projects/xampp/files/XAMPP%20Windows/
[2]: https://www.cvedetails.com/cve/CVE-2007-2080/
[3]: https://www.exploit-db.com/exploits/37396/
[4]: https://www.exploit-db.com/exploits/3738/
[5]: http://www.hackingarticles.in/how-to-hack-xampp-of-remote-pc-using-metasploit/
[6]: https://www.mkyong.com/apache/how-to-enable-webdav-in-apache-server-2-2-x-windows/
[7]: https://www.exploit-db.com/exploits/18367/
[8]: http://php.net/manual/en/function.mssql-connect.php
[9]: ``
[10]: 