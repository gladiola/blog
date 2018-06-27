---
title:  "Hacking XAMPP WEBDAV Vulns"
date:   2018-06-15 08:30:00
description: Exercising Known Vulns in XAMPP for Pentesting Practice
---


asdfasdfasdf

<table>
    <caption>Troubleshooting XAMPP Hack</caption>
    <tr>
    <th>Property</th>
    <th>Value</th>
    <th>Logical Impact</th>
    </tr>
    <tr><th>XAMPP Version</th><td>1.6.0a</td><td>Vuln to known CVEs</td></tr>
    <tr><th>VirtualBox VM</th><td>Windows 7 with IE11 VM</td><td>Simulated OS</td></tr>
    <tr><th>Metasploit Module</th><td>xampp_webdav_upload_php</td><td>Known working exploit</td></tr>
    <tr><th>RHOST</th><td>192.168.1.9</td><td>Target address</td></tr>
    <tr><th>RPORT</th><td>80</td><td>Target port</td></tr>
    <tr><th>LHOST</th><td>192.168.1.10</td><td>Listen address</td></tr>
    <tr><th>LPORT</th><td>4444</td><td>Listen port</td></tr>
    <tr><th>USER</th><td>wampp</td><td>Default user for auth</td></tr>
    <tr><th>PASSWORD</th><td>xampp</td><td>Default password for auth</td></tr>
</table>

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