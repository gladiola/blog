---
title:  "h4cking Vulnhub's \"Bad Store\""
date:   2018-07-23 08:30:00
description: Notes on Hacking Vulnhub's "Bad Store" VM
---

In this post, we'll cover a  

### Spoilers
<blockquote>
    Throughout this post, we'll reveal some aspects of the "Bad Store" target.  Please note that by looking at some of these details, the VM's instructional quality as a target of unknown composition might be diminished to hackers who will want use the target for technical growth and professional development.  Experience recommends thorough use of the target VM before referring to resources such as these.
</blockquote>



Published in 2

### Setup
![Diagram showing Kali and Bad Store VM]({{ site.url }}/assets/images/KaliBadStore/Diagram_Kali_BadStore.png)


### Summary Properties of the Machines
One box is holding the VulnHub VM; it's running VirtualBox; we conducted some simple `ifconfig` and `ping` checks to make sure that it could communicate with other machines on the SOHO network.  This target box is the VM running on a WIN10 laptop.  On the attacker box, we are using FreeBSD 10.3 and a VirtualBox hypervisor running Kali in a VM.

<table>
    <caption>Initial Commo Checks and Basic Configuation</caption>
    <tr>
        <th>Property</th>
        <th>Value</th>
        <th>Logical Impact</th>
    </tr>
    <tr>
        <th>Target Machine:  VirtualBox VM</th>
        <td>VulnHub Bad Store iso in VM</td>
        <td>Target of unknown composition</td>
    </tr>
    <tr>
        <th>Target Machine:  Host OS</th>
        <td>WIN 10</td>
        <td>Target platform host OS</td>
    </tr>
    <tr>
        <th>Attacker Machine:  VirtualBox VM</th>
        <td>Kali</td>
        <td>Attack platform inside VM</td>
    </tr>
    <tr>
        <th>Attacker Machine:  Host OS</th>
        <td>FreeBSD 10.3 RELEASE</td>
        <td>Attack platform host OS</td>
    </tr>
    <tr>
        <th>Target</th>
        <td>192.168.1.9</td>
        <td><code>ifconfig</code> OK</td>
    </tr>
    <tr>
        <th>Attacker</th>
        <td>192.168.1.17</td>
    </tr>
</table>

![Kali VM]({{ site.url }}/assets/images/KaliBadStore/2018-07-07-182516_1366x768_scrot.png)

## Proving COMMO
Since we don't have a need for stealth and do have direct access to the controls of both VMs, we'll start off with a simple commo check between the boxes.  By using 
{% highlight shell %}
ping -c 5 <DESTINATION IP ADDRESS>
{% endhighlight %}




## Kill Chains and Attacking Actions
Throughout our discussion, we'll try to relate the use of commands in Kali to analysis actions that use the Lockheed-Martin Intrusion Kill Chain.  B


## Vulnerability Scanning 




We clicked around some to look up those vulnerabilities on hyperlinks related to the Tenable website; while well supported with CVE documentation and the like, there were easier to exploit possibilties likely.  We continued with some of the other vuln scans like `nmap`.

### `nmap` scan 
{% highlight shell %}
nmap -sV -oA badStore_nmapV 192.168.1.9
{% endhighlight %}

And provided code output files like the blocks below.  The small difference among the scans was to have `nmap` look for slightly different protocol-related ports.  The default scans look for 1000 ports; one search did TCP, another UDP, and the verbose provided a little more output about the versions found. \[2\]

{% highlight shell %}
Nmap 7.25BETA1 
{% endhighlight %}



## `nikto` scanning
![nikto recommendations]({{ site.url }}/assets/images/KaliBadStore/2018-07-07-183641_1366x768_scrot.png)






### Analysis Check
<table>
    <caption>Discovery and Attack Cycle 1</caption>
    <thead>
       <tr>
            <th>Action</th>
            <th>Observation</th>
        </tr> 
    </thead>
    <tbody>
        <tr>
            <td>nmap</td>
            <td></td>
        </tr>
        <tr>
            <td>nikto</td>
            <td></td>
        </tr>
        <tr>
            <td>Internet Research of Programs</td>
            <td></td>
        </tr>
    </tbody>
</table>

<table>
    <caption>Command to Kill Chain Step 1</caption>
    <thead>
       <tr>
            <th>Kill Chain Step</th>
            <th>Attacking Action</th>
            <th>Observation</th>
        </tr> 
    </thead>
    <tbody>
    <tr>
        <th rowspan="5">Reconnaissance</th>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>nmap</td>
        <td>Open ports, server versions</td>
    </tr>
    <tr>
        <td>nikto</td>
        <td>Diretories and open ports</td>
    </tr>
    <tr>
        <td>Internet OSINT</td>
        <td></td>
    </tr>
    </tbody>
</table>



## Cracking hashed passwords with hashcat


To crack one, commands go like:
{% highlight shell %}
hashcat -m0 <TARGET HASH TO CRACK> /usr/share/wordlists/rockyou.txt --force
{% endhighlight %}



# Reporting

## Lessons learned from 
earned a few things to consider the next time we see sites with CGI.


## Annotated Bibliography


\[1\]  Brotherston, Lee and Amanda Berlin.  Defensive Security Handbook:  Best Practices for Securing Infrastructure.  O'Reilly, April 2017. pp. 6-9. ISBN 978-1-491-96038-7. 

A description of Lockheed-Martin's Intusion Kill Chain.  Brotherston and Berlin applied their use case of an overview of a ransomware attack to the phases of Lockheed's kill chain.  Their definitions and example serves as guidance for the contstruction of our own, here.


\[2\]  INTERNET: [`https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-115.pdf`](https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-115.pdf)
NIST documents includes descriptions of vulnerability scanning, penetration testing, password cracking, and other critical terms of applicable techniques.

[//]: # (Hyperlinks)
[1]:  ``
[2]: https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-115.pdf




### Analysis Check
<table>
    <caption>Discovery and Attack Cycle 2</caption>
    <thead>
       <tr>
            <th>Action</th>
            <th>Observation</th>
        </tr> 
    </thead>
    <tbody>
        <tr>
            <td>msfconsole mysql schema dump</td>
            <td>Missing INFORMATION_SCHEMA was influential; nonstandard table names</td>
        </tr>
        <tr>
            <td>msf mysql sql</td>
            <td>Simple calls yielded necessary information to impersonate any account</td>
        </tr>
        <tr>
            <td>hashcat</td>
            <td>Password cracking with wordlists in a reasonable time allowed use of accounts uncovered in previous steps.</td>
        </tr>
        <tr>
            <td>SQLi</td>
            <td>Injectable controls in almost all inputs and file upload dialogs; other inputs showed CGI filtering of URL encoded values</td>
        </tr>
    </tbody>
</table>


<table>
    <caption>Command to Kill Chain Step 2</caption>
    <thead>
        <tr>
            <th>Kill Chain Step</th>
            <th>Attacking Action</th>
            <th>Observation</th>
        </tr> 
    </thead>
    <tbody>
    <tr>
        <th rowspan="2">Reconnaissance</th>
        <td>msfconsole mysql schema dump</td>
        <td>Get database schema</td>
    </tr>
    <tr>
        <td>msf mysql sql</td>
        <td>Obtain users table with passwords</td>
    </tr>
    <tr>
        <th rowspan="2">Weaponization</th>
        <td>hashcat</td>
        <td>Offline password cracking in bulk</td>
    </tr>
    <tr>
        <td>SQLi <CODE>' '1'='1' OR --</CODE> </td>
        <td>Injectable controls all over</td>
    </tr>
    </tbody>
</table>