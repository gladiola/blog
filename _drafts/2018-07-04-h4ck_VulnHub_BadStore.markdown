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
![Diagram showing Kali and Bad Store VM]({{ site.url }}/assets/images/KaliBadStore/Diagram_Kali_BadStore.png)


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
Since we don't have a need for stealth and do have direct access to the controls of both VMs, we'll start off with a simple commo check between the boxes.  By using 
{% highlight shell %}
ping -c 5 <DESTINATION IP ADDRESS>
{% endhighlight %}
and `ifconfig` on both VMs, we could reasonably see that the boxes were communicating.  

If communication among the boxes cannot be established, then a common point to check is the VM network adapter.  Another similar troubleshooting check will be to see if a command line or terminal on the host system can communicate with the VM, and vice versa.  

These commo checks may seem elementary; but, knowing IP addresses and having smooth commo among the boxes can be of help when exporting data.  Kali, in the configuration I was using, did not have an `ftp` command in the terminal.  Also, the FreeBSD version I was using did not have a simple `automount` So, to get files into and out of the attack box, I used FTP uploads through an intermediary website, `PuTTY`, `ssh`, and `scp`.  Since those details are particular to my setup, I won't cover most of those here.  Meanwhile, in most situations where I have had to use VMs on a variety of systems, being proficient with those kinds of file transfers was a handy skill to have.




## Kill Chains and Attacking Actions
Throughout our discussion, we'll try to relate the use of commands in Kali to analysis actions that use the Lockheed-Martin Intrusion Kill Chain.  Brotherston and Berlin, writing in the Defensive Security Handbook, presented an example use case in this format that allowed us to see all sides of the attack. \[5\] They included defensive actions and monitoring in correlations with phases of the kill chain.  Their chart headers looked similar to the one below.

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
To support a Reconnaissance phase, we conducted four kinds of vulnerability scans.  Two were Nessus scans (basic network and web applications); one was a collection on `nmap` scans of TCP and UDP protocol ports; another was a `nikto` scan.  We also did some manual observation of the website (directory traversal and simple injection probing), and a `sqlmap` scan; those will be covered separately.  It's obvious that this was very noisy reconnaissance; but, there are no points for stealth going against a home lab VM.  Let's look at what we can learn from these scans.

### Nessus scans for the site
Given some basic directions for running a Nessus scan,  \[2\] we ran a couple of them against the "Bad Store" VM.  A quick skimming of those results showed several OpenSSL-related vulnerabilities; that's when we began to see how many of the vulns might be historic.  Also listed was a vuln related to an old Apache version.   

Copies of the Nessus scan results that we ran against the VM are available through these links:
- [PDF of Nessus scan using the Basic Network scan type](https://github.com/gladiola/blackmagic/blob/Demo/blog_support/salvage13_BasicNetwork_BadStore_ya6pkz.pdf)
- [PDF of Nessus scan using the Web Application scan type](https://github.com/gladiola/blackmagic/blob/Demo/blog_support/salvage13_BadStore_Web_e1yrt4.pdf)

We clicked around some to look up those vulnerabilities on hyperlinks related to the Tenable website; while well supported with CVE documentation and the like, there were easier to exploit possibilties likely.  For the time being, we turned our attention over to some of the other vuln scans like `nmap`.

### nmap scan for TCP, UDP, and versions
Our collection of `nmap` scans were done to find TCP and UDP ports that might be open.  The scans were run with code like:

{% highlight shell %}
nmap -sS -oA badStore_nmap 192.168.1.9 
nmap -sU -oA badStore_nmapU 192.168.1.9
nmap -sV -oA badStore_nmapV 192.168.1.9
{% endhighlight %}

And provided code output files like the blocks below.  The small difference among the scans was to have `nmap` look for slightly different protocol-related ports.  The default scans look for 1000 ports; one search did TCP, another UDP, and the verbose provided a little more output about the versions found. \[2\]

{% highlight shell %}
Nmap 7.25BETA1 scan initiated Thu Jun 28 21:58:25 2018 as: nmap -sS -oA badStore_nmap 192.168.1.9
Nmap scan report for 192.168.1.9
Host is up (0.0097s latency).
Not shown: 997 closed ports
PORT     STATE SERVICE
80/tcp   open  http
443/tcp  open  https
3306/tcp open  mysql
MAC Address: 00:26:C6:CC:BE:3A (Intel Corporate)

Nmap done at Thu Jun 28 21:58:26 2018 -- 1 IP address (1 host up) scanned in 0.91 seconds
{% endhighlight %}

{% highlight shell %}
Nmap 7.25BETA1 scan initiated Thu Jun 28 22:01:06 2018 as: nmap -sU -oA badStore_nmapU 192.168.1.9
Nmap scan report for 192.168.1.9
Host is up (0.014s latency).
All 1000 scanned ports on 192.168.1.9 are closed (957) or open|filtered (43)
MAC Address: 00:26:C6:CC:BE:3A (Intel Corporate)

Nmap done at Thu Jun 28 22:18:24 2018 -- 1 IP address (1 host up) scanned in 1037.61 seconds
{% endhighlight %}

{% highlight shell %}
Nmap 7.25BETA1 scan initiated Thu Jun 28 21:59:48 2018 as: nmap -sV -oA badStore_nmapV 192.168.1.9
Nmap scan report for 192.168.1.9
Host is up (0.0093s latency).
Not shown: 997 closed ports
PORT     STATE SERVICE  VERSION
80/tcp   open  http     Apache httpd 1.3.28 ((Unix) mod_ssl/2.8.15 OpenSSL/0.9.7c)
443/tcp  open  ssl/http Apache httpd 1.3.28 ((Unix) mod_ssl/2.8.15 OpenSSL/0.9.7c)
3306/tcp open  mysql    MySQL 4.1.7-standard
MAC Address: 00:26:C6:CC:BE:3A (Intel Corporate)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done at Thu Jun 28 22:00:04 2018 -- 1 IP address (1 host up) scanned in 16.55 seconds
{% endhighlight %}

So, what does this tell us?  We can see that those three ports are open and nothing else.  This will let us see that we will need to focus on programs that serve HTTP(S) and MySQL.  Trying to attack other applications will probably be fruitless.  

## sqlmap scanning
Since we do have a website up, and since it is running MySQL, it only takes a quick jump to suppose that we might have a chance at getting everything with a quick run of `sqlmap`.  What happens when we give that a try?

Not much.  That is, `sqlmap` ran its default scan against the target, but didn't find what it was usually looking for.  So, our friend from many commercial engagements, `sqlmap`, would not be of much help here.

## nikto scanning
Quick and easy, `nikto` scannig did a directory traverse that seemed easy to use.  It coughed up five or size directories to check into.  For brevity, We'll show what we eventually found with some of those.  

The supplier directory was referred to in robots.txt.  Looking up robots.txt showed that a user-agent of a certain name would not be disallowed.  This looked useful.  Also, robots.txt mentioned an `/upload` directory.  That, combined with a file upload dialog, implied an remote or local file inclusion vulnerability possibility.

The `cgi-bin/` showed a couple of things.  First, laying around was a `test.cgi` file.  Later, after finding some hashes, it would match up with a sysadmin's account.  Apparently, this was a leftover test laying around in the plain.  It showed clearly that there were base64 and MD5 hashes in use; but, that would be close to quickly recognized by the shape of the strings found later.

There was a `/supplier/accounts` laying around.  This was a text file with about four lines associating a number with a base64 encoded string.  Recognizable by its trailing "=", the base64 was quickly decoded.  

{% highlight shell %}
echo <TARGET STRING> | base64 --decode
{% endhighlight %}

The output showed a pattern like:
`<100X>:<USER>/<PASSWORD>/<?OPTIONAL METAL WORD>/<IP ADDRESS>`
These later proved to be an association between item numbers, a supplier, their password, and IP.  However, the site used email addresses to run the logins, so these account rows did nto get me into anyone's account.

## Manual directory checks and source code reading
One of the first things we can do is to look at the website.  
- Are there any forms?  
- Does it react to URL encoding?  
- Does the source code seem to pull down any outside scripts?  
- Where are other assets stored?  

Given those questions, we can go hunting for plenty of vulnerabilities without any scanning tools.  What turns up?

## Tickmark 1 equals 1
Almost every single text input I tried showed some kind of vulnerability to `' OR '1'='1' `.  Often, it showed an error.  In the case of the CGI guestbook, it accepted the call as text and displayed it on th eweb page without throwing the usual errors.  When placed in the supplier login text input, the `'1=1` hack would make the file upload dialog pop up.  Maybe useful later.

## Some handy facts laying out in the plain
A look at the source code of each page revealed that a lot of form processing was being donw in CGI.  Much luck for me; I never got into CGI.  So, that lead would require more research to use.  

Meanwhile, it also turned up a script, `frmvrfy.js` that compares two password values.  Apparently part of the reset routine.  

## Minor stump
At this point, I had a lot of information, but no real login.  I didn't want to give in and read the provided directions that are on the site.  Time to plink around a little more and see what I could do.  Overall, I felt that I should be getting a login and a chance to see veryone's account history from the site.  Not being able to do that was a little discouraging.  I would have to find a way to ge that somehow.

I tried a couple of Metasploit modules; but, really, a moment of success came by running some of the MySQL-realted auxilliaries.

## We got the gold
By running some MySQL auxilliaries with `msfconsole`, we were able to send SQL directly to the MySQL engine, dump schema, and `SELECT` a bunch of useful content.  One of the results was that we were able to get database records for usernames as email addresses (this particular website logs users in by email addy), passwords, bank account numbers, and detailed transaction information. 

By `SELECT`ing those email addresses and passwords from the userdb table, `msfconsole` was able to output a simple text file that could be trimmed and used as input for `hashcat`. 

## Cracking hashed passwords with hashcat
When the passwords were downloaded, by outputting the SQL to a text file, the passwords were hashed with MD5.  That simply wouldn't do.  In order to get a readily usable copy of those passwords, we'd need to crack the hash.  Hello, `hashcat`.

To crack one, commands go like:
{% highlight shell %}
hashcat -m0 <TARGET HASH TO CRACK> /usr/share/wordlists/rockyou.txt --force
{% endhighlight %}

A couple things happened here.  I had to use the `--force` to get past an error; I had to do some file transfers because my version of Kali wasn't up to date; and I had to turn on `rockyou.txt` for the first time on that machine.  

One more thing happened:  when feeding hashes in to `hashcat`, they would disappear from console output after the first crack.  The answers would turn up in the potfile, below.  It became clear that when feeding a big block of hashes to crack to `hashcat`, it was important to plan for an out-of-order extraction of answers.  For example, the fifth hash input would not necessarily be the fifth-from-last hash in the potfile.  So, in the future, we would need to prepare more to prevent fumbling when getting the cracked password back out.  This time, the result set was so small that simple comparison was still practical.

### Where is the potfile?
After the first console display of the hashes, it was hard to see the cracked passwords for a moment.  When `hashcat` cracks a hash, by design, it will probably not attempt to crack that hash again.  In this way, `hashcat` can run faster and faster; with only unresolved hashes remaining, it will have less and less search area to cover.  

Meanwhile, as noobs running `hashcat` for the first few times, we were wondering, "Where the hell are my cracked hashes?" \[[6]\]\[[7]\]\[[8]\] The documentation told us that it would be in the potfile.  That potfile would be in the directory where `hashcat` was run.  By looking at the directory, it didn't seem to be there.  

The *potfile* is a file that holds all of the cracked hashes.  The hashcat documentation refers to it repeatedly; the potfile is in a hidden directory, in the same directory that `hashcat` was run in.  To list it and see the contents, from the directory where `hashcat` was run, try:

{% highlight shell %}
ls -lista .hashcat/hashcat.pot
{% endhighlight %}

The potfile we had for this run had entries that looked like this:

{% highlight shell %}
5ebe2294ecd0e0f08eab7690d2a6ee69:secret
5f4dcc3b5aa765d61d8327deb882cf99:password
9726255eec083aa56dc0449a21b33190:money
...
{% endhighlight %}

As mentioned earlier, the entries in the `hashcat` potfile didn't seem to be in the expected order.  A careful comparison of the hashes put in and the hashes in the potfile showed that they were not printed in the expected order.  The sequence of the input file was not the same as the sequence of hashes in the potfile.  Since the hashes were there beside their cracked values, a common search or `grep` would yield the correct answer; just be careful about skimming down the file or jumping to the last hash plaintext value; the last value in the potfile does not necessarily represent the last hash provided to `hashcat` to crack.

As a noob, this might seem counterintuitive.  Why would `hashcat` not simply answer with the cracked hash in the order that it received one to crack?  Keep in mind, that this is kind of a tiny "toy" use case.  `hashcat` is meant for bigger work.  If you wanted to crack many, many hashes using some GPUs, then `hashcat` was built for you.  This simple set of block extractions was almost an edge case.

## Next goals
At this point, it was time to set some new goals.  Thoroughly scanned, with a database coughing up pretty much whatever we wanted, the site would yield whatever information simple account impersonation might provide.  That said, it was not enough.  A lot of what was done thus far was passive.  A malicious attacker would shape and control the box so that it could be used for her own ends.  

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

\[6\] _____.  "What is a potfile?" Hashcat Wiki.  INTERNET:  [`https://hashcat.net/wiki/doku.php?id=frequently_asked_questions#how_can_i_show_previously_cracked_passwords_and_output_them_in_a_specific_format_eg_emailpassword`](https://hashcat.net/wiki/doku.php?id=frequently_asked_questions#how_can_i_show_previously_cracked_passwords_and_output_them_in_a_specific_format_eg_emailpassword)

\[7\] _____.  "How can I show previously cracked passwords, and output them in a specific format (e.g. email:password)?" Hashcat Wiki.  INTERNET:  [`https://hashcat.net/wiki/doku.php?id=frequently_asked_questions#how_can_i_show_previously_cracked_passwords_and_output_them_in_a_specific_format_eg_emailpassword`](https://hashcat.net/wiki/doku.php?id=frequently_asked_questions#how_can_i_show_previously_cracked_passwords_and_output_them_in_a_specific_format_eg_emailpassword)

\[8\]  _____.  "Hashcat reports "Status: Cracked", but did not print the hash value, and the outfile is empty. What happened?" Hashcat Wiki.  INTERNET: [`https://hashcat.net/wiki/doku.php?id=frequently_asked_questions#hashcat_reports_statuscracked_but_did_not_print_the_hash_value_and_the_outfile_is_empty_what_happened`](https://hashcat.net/wiki/doku.php?id=frequently_asked_questions#hashcat_reports_statuscracked_but_did_not_print_the_hash_value_and_the_outfile_is_empty_what_happened)

## Spare Parts

{% highlight shell %}

{% endhighlight %}


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

## Photos

![ALT]({{ site.url }}/assets/images/KaliBadStore/Diagram_Kali_BadStore.png)
![ALT]({{ site.url }}/assets/images/KaliBadStore/2018-07-07-182516_1366x768_scrot.png)
![ALT]({{ site.url }}/assets/images/KaliBadStore/2018-07-07-183641_1366x768_scrot.png)
![ALT]({{ site.url }}/assets/images/KaliBadStore/2018-07-07-184427_1366x768_scrot.png)
![ALT]({{ site.url }}/assets/images/KaliBadStore/2018-07-07-192718_1366x768_scrot.png)
![ALT]({{ site.url }}/assets/images/KaliBadStore/2018-07-07-195544_1366x768_scrot.png)
![ALT]({{ site.url }}/assets/images/KaliBadStore/2018-07-07-204857_1366x768_scrot.png)
![ALT]({{ site.url }}/assets/images/KaliBadStore/2018-07-07-210740_1366x768_scrot.png)
![ALT]({{ site.url }}/assets/images/KaliBadStore/2018-07-07-211524_1366x768_scrot.png)
![ALT]({{ site.url }}/assets/images/KaliBadStore/2018-07-07-212240_1366x768_scrot.png)
![ALT]({{ site.url }}/assets/images/KaliBadStore/2018-07-07-214245_1366x768_scrot.png)
![ALT]({{ site.url }}/assets/images/KaliBadStore/2018-07-07-214600_1366x768_scrot.png)
[//]: # (Hyperlinks)
[1]:  ``
[2]: ``
[3]: ``
[4]:  https://www.vulnhub.com/entry/badstore-123,41/
[5]: ``
[6]: https://hashcat.net/wiki/doku.php?id=frequently_asked_questions#what_is_a_potfile
[7]: https://hashcat.net/wiki/doku.php?id=frequently_asked_questions#how_can_i_show_previously_cracked_passwords_and_output_them_in_a_specific_format_eg_emailpassword
[8]:  https://hashcat.net/wiki/doku.php?id=frequently_asked_questions#hashcat_reports_statuscracked_but_did_not_print_the_hash_value_and_the_outfile_is_empty_what_happened
[9]: 
