---
title:  "A Better Mousetrap for USB Drop Attacks"
date:   2018-07-23 08:30:00
description: Notes on building better USB drops with improved elicitation and pretexting support for various attacks
---

The first time I fired up a SET USB attack, as soon as I plugged the USB into the machine, my antivirus went off.  Immediately, the scanner had detected the "autorun" file that was to impersonate a PDF and launch a reverse shell.  After a moment, the antivirus scanner had not only alerted me to two of these malicious programs, which I had planted on the drive, but also it removed them from the drive.  There had to be a better way, I thought.

There is.  In this blog post we'll explore some options for USB attacks, including the world-famous Rubber Ducky.  In our recent research, we found that it's possible to detect USB drop attack vectors compromising friendly computers without installing any malware on the devices at all.  Also, we'll present some examples of improving the likelihood that the dropped USBs seem more realistic to those victims we're seeking with the drops.  We'll do the homework to improve pretexting and elicitation; the psychological, social engineering techniques that encourage users to respond to the USBs and interact with them in ways that promote compromise.

## Why ATK with a USB drop?

### AutoRun
### NIST 800s and USBs

## Outfitting a USB drive using SET

## Dressing a plain USB for better elicitation

## Using Get-USBHistory to recover a known USB drive name

## Outfitting a Rubber Ducky with PowerShell to collect system data

## Preparing Rubber Ducky for use with PowerShell Empire 

## Outfitting a USB with a harmful exe 
### Plausible situation
### Preparing an exe for use with msfvenom 

Targeted exe:  The Etcher program for Raspberry Pi.
REF:  Georgia's CH. on AV evasion

## Preparing a Warberry for use as a waypoint for attacks

## Using a server with a Dynamic DNS service as a receiver
### Establishing ddclient on FreeBSD
### Establishing TLS on FreeBSD with Tomcat

## Annotated Bibliography


\[1\]  Brotherston, Lee and Amanda Berlin.  Defensive Security Handbook:  Best Practices for Securing Infrastructure.  O'Reilly, April 2017. pp. 6-9. ISBN 978-1-491-96038-7. 

A description of Lockheed-Martin's Intusion Kill Chain.  Brotherston and Berlin applied their use case of an overview of a ransomware attack to the phases of Lockheed's kill chain.  Their definitions and example serves as guidance for the contstruction of our own, here.


\[2\]  INTERNET: [`https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-115.pdf`](https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-115.pdf)
NIST documents includes descriptions of vulnerability scanning, penetration testing, password cracking, and other critical terms of applicable techniques.

\[3\]  [`https://nvlpubs.nist.gov/nistpubs/specialpublications/nist.sp.800-61r2.pdf`](https://nvlpubs.nist.gov/nistpubs/specialpublications/nist.sp.800-61r2.pdf)

\[4\] [`https://nvlpubs.nist.gov/nistpubs/legacy/sp/nistspecialpublication800-122.pdf`](https://nvlpubs.nist.gov/nistpubs/legacy/sp/nistspecialpublication800-122.pdf)

\[5\] [`https://www.microsoft.com/en-us/wdsi/threats/threat-search?query=worm:win32/autorun.ahm&page=1&showall=false&sortby=relevance&sortdir=desc&size=10`](https://www.microsoft.com/en-us/wdsi/threats/threat-search?query=worm:win32/autorun.ahm&page=1&showall=false&sortby=relevance&sortdir=desc&size=10)
Windows Defender page related to worms with "autorun" in their names.  500 results found.

\[6\] [`https://www.zdnet.com/article/pentagons-failed-flash-drive-ban-policy-a-lesson-for-every-cio/`](https://www.zdnet.com/article/pentagons-failed-flash-drive-ban-policy-a-lesson-for-every-cio/)
ZDNet article on how a ban on USB removable storage was eroded over a few years by thousands of exceptions that were granted to users.

\[7\] [`http://www.scarfonecybersecurity.com/`](http://www.scarfonecybersecurity.com/)
Website of NIST technical writer Karen Scarfone.

\[8\] [`https://publicintelligence.net/fbi-qakbot-usb/`](https://publicintelligence.net/fbi-qakbot-usb/)
Publicintelligence post about an FBI press release stating that Qakbot malware was found on USB devices arriving directly from China.

\[9\] [`https://info.publicintelligence.net/FBI-QakbotUSB.pdf`](https://info.publicintelligence.net/FBI-QakbotUSB.pdf)
Download link for PDF referred to in \[8\].

\[10\] [`https://publicintelligence.net/ncsc-cyber-economic-espionage/`](https://publicintelligence.net/ncsc-cyber-economic-espionage/)
Publicintelligence post summarizing foreign intelligence economic espionage activities in the United States.

\[11\] [`https://info.publicintelligence.net/NCSC-CyberEconomicEspionage.pdf`](https://info.publicintelligence.net/NCSC-CyberEconomicEspionage.pdf)
Download link for PDF referred to in \[10\].

\[12\] [`https://www.wired.com/2008/11/army-bans-usb-d/`](https://www.wired.com/2008/11/army-bans-usb-d/)
Wired article about a US military ban on USB removable storage in 2008.

\[13\] [`https://www.symantec.com/security-center/writeup/2006-071111-0646-99?tabid=2`](https://www.symantec.com/security-center/writeup/2006-071111-0646-99?tabid=2)
Symantec post about the W32.SillyFDC worm referred to in \[12\].  "Technical Description" tab includes an explanation of how the worm would disable basic security features on the computer and modify the registry.  The worm would typically order itself to be copied onto any removable drive when Windows starts.

\[14\] [`https://hakshop.com/blogs/news/stealing-files-with-the-usb-rubber-ducky-usb-exfiltration-explained`](https://hakshop.com/blogs/news/stealing-files-with-the-usb-rubber-ducky-usb-exfiltration-explained)
Hak5 post summarizing directions demonstrated in three YouTube videos for preparing a Rubber Ducky HID keystroke injection attack tool to exfiltrate data.

\[15\] [`https://www.hak5.org/episodes/season-21/hak5-2112-stealing-files-with-the-usb-rubber-ducky`](https://www.hak5.org/episodes/season-21/hak5-2112-stealing-files-with-the-usb-rubber-ducky)
Part 1 of 3 Hak5 videos referred to in \[14\].

\[16\] [`https://www.hak5.org/episodes/season-21/stealing-files-with-the-usb-rubber-ducky-pt-2-hak5-2113`](https://www.hak5.org/episodes/season-21/stealing-files-with-the-usb-rubber-ducky-pt-2-hak5-2113)
Part 2 of 3 Hak5 videos referred to in \[14\].

\[17\] [`https://www.hak5.org/episodes/season-21/hak5-2114-stealing-files-with-the-usb-rubber-ducky-pt-3`](https://www.hak5.org/episodes/season-21/hak5-2114-stealing-files-with-the-usb-rubber-ducky-pt-3)
Part 3 of 3 Hak5 videos referred to in \[14\].

\[18\] [`https://www.powershellempire.com/?page_id=104`](https://www.powershellempire.com/?page_id=104)
Post describing features in Powershell Empire.  Includes the ability to generate reverse shells that work with listeners automatically generated by Empire.  These features include the ability of Empire to directly generate payloads encoded for immediate use with a Rubber Ducky.

\[19\] [`https://blogs.technet.microsoft.com/heyscriptingguy/2012/05/18/use-powershell-to-find-the-history-of-usb-flash-drive-usage/`](https://blogs.technet.microsoft.com/heyscriptingguy/2012/05/18/use-powershell-to-find-the-history-of-usb-flash-drive-usage/)
Microsoft Technet article on some scripts that can obtain the history of USB removable storage devices used on a machine.  Includes a remote option for systematic crawling of a network.

\[20\] [`http://gallery.technet.microsoft.com/scriptcenter/Get-USBHistory-707e43a3`](http://gallery.technet.microsoft.com/scriptcenter/Get-USBHistory-707e43a3)
Microsoft script repository with a copy of Get-USBHistory for PowerShell.  The description states, "This script uses Win32.Registry class to enumerate through the USBSTOR key in the registry to get a list of USB storage devices that have been use on a machine.  This script is designed to run locally or remotely."

\[21\] [`https://docs.kali.org/kali-on-arm/install-kali-linux-arm-raspberry-pi`](https://docs.kali.org/kali-on-arm/install-kali-linux-arm-raspberry-pi)
Directions for installing Kali Linux on a Raspberry Pi.

\[22\] [`https://www.offensive-security.com/kali-linux-arm-images/`](https://www.offensive-security.com/kali-linux-arm-images/)
Offensive Security downloads page for Kali on ARM devices.

\[23\] [`https://github.com/secgroundzero/warberry`](https://github.com/secgroundzero/warberry)
Repo for the Warberry Pi.

\[24\] [`https://www.kitploit.com/2016/05/warberrypi-turn-your-raspberry-pi-into.html`](https://www.kitploit.com/2016/05/warberrypi-turn-your-raspberry-pi-into.html)
Kitploit article on preparing the Warberry for use.  Includes photos and installation tips that can activate the Warberry with a mechanical switch. This article was the source of several other hyperlinks listed below.

\[25\] [`https://github.com/DanMcInerney/net-creds`](https://github.com/DanMcInerney/net-creds)
Github repo about sniffing password credentials from pcap files.

\[26\] [`https://github.com/commixproject/commix`](https://github.com/commixproject/commix)
Github repo for Commix, a command injection tool.

\[27\] [`https://github.com/sqlmapproject/sqlmap`](https://github.com/sqlmapproject/sqlmap)
Github repo for sqlmap.

\[28\] [`https://github.com/CoreSecurity/impacket`](https://github.com/CoreSecurity/impacket)
Github repo for Impacket, a packet-crafting library for Python.
"Impacket is focused on providing low-level programmatic access to the packets ..."

\[29\] [`https://github.com/samratashok/nishang`](https://github.com/samratashok/nishang)
Github repo for nishang, a collection of scripts for using PowerShell for offensive security.

\[30\] Gaffie, Laurent.  INTERNET:[`https://github.com/SpiderLabs/Responder`](https://github.com/SpiderLabs/Responder)
Github repo for Responder.  The repo description stated, "Responder is a LLMNR, NBT-NS and MDNS poisoner, with built-in HTTP/SMB/MSSQL/FTP/LDAP rogue authentication server supporting NTLMv1/NTLMv2/LMv2, Extended Security NTLMSSP and Basic HTTP authentication."

\[31\] [`https://github.com/wifiphisher/wifiphisher`](https://github.com/wifiphisher/wifiphisher)
Github repo for wifiphisher, a rogue access point.

\[32\] [`https://github.com/Dionach/CMSmap`](https://github.com/Dionach/CMSmap)
Github repo for CMSMap.  The repo description stated, "CMSmap is a python open source CMS scanner that automates the process of detecting security flaws of the most popular CMSs. The main purpose of CMSmap is to integrate common vulnerabilities for different types of CMSs in a single tool."

\[33\] [`https://github.com/PowerShellMafia/PowerSploit`](https://github.com/PowerShellMafia/PowerSploit)
Github repo for PowerSploit.  It is a collection of scripts that can be used for code injection, antivirus evasion, script modification, persistence, data exfiltration, privilege escalation, reconaissance, and system sabotage.

\[34\] [`https://github.com/secgroundzero/warberry/wiki/3G-Covert-Channel-Setup`](https://github.com/secgroundzero/warberry/wiki/3G-Covert-Channel-Setup)
Github wiki on using the Raspberry Pi as a Warberry with 3G connection through a dongle.

\[35\] [`https://raspberrypi.stackexchange.com/questions/44143/verified-4g-usb-dongles`](https://raspberrypi.stackexchange.com/questions/44143/verified-4g-usb-dongles)
StackExchange post on figuring out which USB 3G|4G dongles will work with Raspberry Pi.

\[36\] [`https://www.peerlyst.com/posts/warberrypi-the-complete-guide-secgroundzero`](https://www.peerlyst.com/posts/warberrypi-the-complete-guide-secgroundzero)
Blog post that explained using Warberry with a 3G dongle and a remoting service to bypass local networks.  Idea is to use the dongle as a telephone, and use the data plan on the SIM card to SSH into the Warberry.  From there, the Warberry can be controlled remotely to conduct the attack.

\[37\] [`https://www.remot3.it/web/remot3-it-connect-for-raspberry-pi.html`](https://www.remot3.it/web/remot3-it-connect-for-raspberry-pi.html)
Website advertisement for a company that offers third party remote administration for Raspberry Pi through 3G dongles.

https://raspberrypi.stackexchange.com/questions/44143/verified-4g-usb-dongles
https://publicintelligence.net/ncsc-cyber-economic-espionage/

[//]: # (Hyperlinks)
[1]:  ``
[2]: https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-115.pdf
[3]: https://nvlpubs.nist.gov/nistpubs/specialpublications/nist.sp.800-61r2.pdf
[4]: https://nvlpubs.nist.gov/nistpubs/legacy/sp/nistspecialpublication800-122.pdf
[5]: https://www.microsoft.com/en-us/wdsi/threats/threat-search?query=worm:win32/autorun.ahm&page=1&showall=false&sortby=relevance&sortdir=desc&size=10
[6]: https://www.zdnet.com/article/pentagons-failed-flash-drive-ban-policy-a-lesson-for-every-cio/
[7]: http://www.scarfonecybersecurity.com/
[8]: https://publicintelligence.net/fbi-qakbot-usb/
[9]: https://info.publicintelligence.net/FBI-QakbotUSB.pdf
[10]: https://publicintelligence.net/ncsc-cyber-economic-espionage/
[11]: https://info.publicintelligence.net/NCSC-CyberEconomicEspionage.pdf
[12]: https://www.wired.com/2008/11/army-bans-usb-d/
[13]: https://www.symantec.com/security-center/writeup/2006-071111-0646-99?tabid=2
[14]: https://hakshop.com/blogs/news/stealing-files-with-the-usb-rubber-ducky-usb-exfiltration-explained
[15]: https://www.hak5.org/episodes/season-21/hak5-2112-stealing-files-with-the-usb-rubber-ducky
[16]: https://www.hak5.org/episodes/season-21/stealing-files-with-the-usb-rubber-ducky-pt-2-hak5-2113
[17]:  https://www.hak5.org/episodes/season-21/hak5-2114-stealing-files-with-the-usb-rubber-ducky-pt-3
[18]: https://www.powershellempire.com/?page_id=104
[19]: https://blogs.technet.microsoft.com/heyscriptingguy/2012/05/18/use-powershell-to-find-the-history-of-usb-flash-drive-usage/
[20]: http://gallery.technet.microsoft.com/scriptcenter/Get-USBHistory-707e43a3
[21]: https://docs.kali.org/kali-on-arm/install-kali-linux-arm-raspberry-pi
[22]: https://www.offensive-security.com/kali-linux-arm-images/
[23]: https://github.com/secgroundzero/warberry
[24]: https://www.kitploit.com/2016/05/warberrypi-turn-your-raspberry-pi-into.html
[25]: https://github.com/DanMcInerney/net-creds
[26]: https://github.com/commixproject/commix
[27]: https://github.com/sqlmapproject/sqlmap
[28]: https://github.com/CoreSecurity/impacket
[29]: https://github.com/samratashok/nishang
[30]: https://github.com/SpiderLabs/Responder
[31]: https://github.com/wifiphisher/wifiphisher
[32]: https://github.com/Dionach/CMSmap
[33]: https://github.com/PowerShellMafia/PowerSploit
[34]: https://github.com/secgroundzero/warberry/wiki/3G-Covert-Channel-Setup
[35]: https://raspberrypi.stackexchange.com/questions/44143/verified-4g-usb-dongles
[36]: https://www.peerlyst.com/posts/warberrypi-the-complete-guide-secgroundzero
[37]: https://www.remot3.it/web/remot3-it-connect-for-raspberry-pi.html

