---
title:  "Exploring XAMPP WEBDAV Vulns"
date:   2018-06-26 08:30:00
description: Exercising Known Vulns in XAMPP for Pentesting Practice
---


## Abstract
The goal is to hack into a box that is running a Windows VM that's holding an older version of XAMPP with known vulns.  We'll target webdav so that we can use a default metasploit module.  Both the attacker box and target box are in the same physical room on the same net, addressed by the same router.  With user-side access to both machines, they are available to troubleshoot.  When the initial hacks were attempted, the hack failed.  These notes are about why the hack failed, what was done to fix the lab, and what was discovered along the way.

#### Setup
![setup diagram]({{ site.url }}/assets/images/XAMPPKali/Diagram_XAMPP_Kali.png)

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

## Prove Commo
The first thing to do was to prove that the two boxes could communicate.  A simple `ping -c 5 <TARGET IP>` did it.  Using a browser, it was possible to call `http://<TARGET IP>/webdav` and see Apache serving a "Webdav test page," although no login challenge and reply was given.  It was just a plain html page.  

## Prove Default Exploit
The next thing to do, among many trials, was to make sure that I didn't get turned around and somewhow mess up the setting on the Metasploit properties.  Checking directions against Georgia's book [9], I felt this part was probably going okay.  

## Is webdav Vulnerable the Way They Say It Is?
So, a question that emerged was that I could not see the default password in the config files for XAMPP's webdav config.  The Metasploit module and the CVE tell me that the credentials are a default of `wamp:xampp`, but I'm skeptical.  When I looked into the config file, I could not see the pass in plain text; not much of a surprise, it was hashed.  Hash looks like MD5.  So, can we find out if the string provided is a hashed value of `xampp`?  I suspect that it is because I saw the same string in my config also mentioned in some forum posts asking about webdav and XAMPP.  No matter, I want to see for myself.  

If the pass is not set to the expected value, then the default creds are not in the older form of XAMPP as it was downloaded from Sourceforge.  Maybe someone fixed it?  Or, if they were as they were all along, unadjusted as we would expect they would be, then a reversal of the hash should show the expected pass amoung the values in an MD5 collision.  I know from grad school research that I came across a paper on MD5 collisions; I'm not sophisiticated enough to haphazardly recreate that research with some ad hoc programs; but, someone else has already made a collision engine for MD5. [10]

## Abandoned.
Cut my losses with this project.  Abandoned it for now.  Maybe I will come back to it later.  Trials included re-hashing clue-words for use as a password; those candidates did get close in value; but, none of them was an easy match.  Did find a PowerShell script to quickly run the MD5's on the strings.

WEBDAV may be installed as an option by default, but if it is not set up and running at all; well, then how would we expect to exploit it?  I was not so sure that I actually saw the service working.  Since the exploits are based on the premise that the service is somehow running unobserved in the background by default, I am not so sure that we are actually meeting the conditions necessary to have something that we can work against.


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

\[10\] _____.  "Is it possible to decrypt md5 hashes?"  INTERNET: [`https://stackoverflow.com/questions/1240852/is-it-possible-to-decrypt-md5-hashes`](https://stackoverflow.com/questions/1240852/is-it-possible-to-decrypt-md5-hashes) Stack Overflow.

Forum question and answers on reversing MD5 hashes.  While a perfect, one-answer reversal is not possible, finding a known answer among a set of collision results would be acceptable.

\[11\] _____. [`https://www.bishopfox.com/download/3486/`](https://www.bishopfox.com/download/3486/)

C program to generate MD5 collisions.

\[12\] _____. [`https://www.bishopfox.com/resources/downloads/`](https://www.bishopfox.com/resources/downloads/)

Download page that had to be deduced from some hyperlinks in an answer in the StackOverflow article related to this topic.

\[13\] Wang, . [`http://merlot.usc.edu/csac-f06/papers/Wang05a.pdf`](http://merlot.usc.edu/csac-f06/papers/Wang05a.pdf) 

PDF holding Wang's paper on reversing MD5.  Reading over this was necessary in order to understand 

\[14\] `https://gallery.technet.microsoft.com/scriptcenter/Get-StringHash-aa843f71`

Run a script to try a strait collision.  Fails.

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
[10]: https://stackoverflow.com/questions/1240852/is-it-possible-to-decrypt-md5-hashes
[11]: https://www.bishopfox.com/download/3486/
[12]: https://www.bishopfox.com/resources/downloads/
[13]: http://merlot.usc.edu/csac-f06/papers/Wang05a.pdf
[14]: https://gallery.technet.microsoft.com/scriptcenter/Get-StringHash-aa843f71