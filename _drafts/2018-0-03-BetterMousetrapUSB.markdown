---
title:  "A Better Mousetrap for USB Drop Attacks"
date:   2018-07-23 08:30:00
description: Notes on building better USB drops with improved elicitation and pretexting support for various attacks
---

The first time I fired up a SET USB attack, as soon as I plugged the USB into the machine, my antivirus went off.  Immediately, the scanner had detected the "autorun" file that was to impersonate a PDF and launch a reverse shell.  After a moment, the antivirus scanner had not only alerted me to two of these malicious programs, which I had planted on the drive, but also it removed them from the drive.  There had to be a better way, I thought.

There is.  In this blog post we'll explore some options for USB attacks, including the world-famous Rubber Ducky.  In our recent research, we found that it's possible to detect USB drop attack vectors compromising friendly computers without installing any malware on the devices at all.  Also, we'll present some examples of improving the likelihood that the dropped USBs seem more realistic to those victims we're seeking with the drops.  We'll do the homework to improve pretexting and elicitation; the psychological, social engineering techniques that encourage users to respond to the USBs and interact with them in ways that promote compromise.

## Why ATK with a USB drop?




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
[21]: 

