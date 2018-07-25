---
title:  "h4cking Vulnhub's \"Bad Store\""
date:   2018-07-23 08:30:00
description: Notes on Hacking Vulnhub's "Bad Store" VM
---

In this post, we'll cover adapting some of the recon techniques outlined in Georgia Weidman's book, *Pentration Testing* \[1\]\[2\]\[3\] to an unknown set of problems found in Kurt R. Roemer's "Bad Store" ISO on VulnHub. \[[4]\]  Our goal will be to use the VulnHub VM as a target.  We'll find what's possible by doing some scanning and enumeration. From there, we'll look at some exploits.  

### Spoilers
<blockquote>
    Throughout this post, we'll reveal some aspects of the "Bad Store" target.  Please note that by looking at some of these details, the VM's instructional quality as a target of unknown composition might be diminished to hackers who will want use the target for technical growth and professional development.  Experience recommends thorough use of the target VM before referring to resources such as these.
</blockquote>



Published in 2004, "Bad Store" \[[4]\] is one of the oldest VMs on Vulnhub available for these kinds of penetrations.  The VM target contains a poorly secured storefront website running on a Linux system with a MySQL database and some CGI programming.  The age of the target is significant in that, commercially, we see much less CGI programming than we used to; also, we will see that the passing of time has revealed new vulnerabilities in the technologies used in the VM that may not have been a part of the initial exercises.

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
        <td>192.168.1.10</td>
        <td><code>ifconfig</code> OK</td>
    </tr>
</table>

![Kali VM]({{ site.url }}/assets/images/KaliBadStore/2018-07-07-182516_1366x768_scrot.png)

## Proving COMMO
Since we don't have a need for stealth and do have direct access to the controls of both VMs, we'll start off with a simple commo check between the boxes.  By using 
{% highlight shell %}
ping -c 5 <DESTINATION IP ADDRESS>
{% endhighlight %}
and `ifconfig` on both VMs, we could reasonably see that the boxes were communicating.  

If communication among the boxes cannot be established, then a common point to check is the VM network adapter.  Another similar troubleshooting check will be to see if a command line or terminal on the host system can communicate with the VM, and vice versa.  

These commo checks may seem elementary; but, knowing IP addresses and having smooth commo among the boxes can be of help when exporting data. To get files into and out of the attack box, I used FTP uploads through an intermediary website, `PuTTY`, `ssh`, and `scp`.  In most situations where I have had to use VMs on a variety of systems, being proficient with those kinds of file transfers was a handy skill to have.

## ISO Reset
When working with the "Bad Store" VM, it was noted in some of the early directions that the VM would reset itself if restarted.  This became more of a consideration, later, when thinking up what type of exploits could be used inside the target.  Meanwhile, we decided not to turn the VM completely off; instead, when it was time to end a study session the VMs for both the attacker and the target would have their states saved before the computers were shut down.  This, in conjunction with giving no real commands on the target box, was one of the few rules of this exercise.

## Kill Chains and Attacking Actions
Throughout our discussion, we'll try to relate the use of commands in Kali to analysis actions that use the Lockheed-Martin Intrusion Kill Chain.  Brotherston and Berlin, writing in the Defensive Security Handbook, presented an example use case in this format that allowed us to see all sides of the attack. \[5\] They included defensive actions and monitoring in correlations with phases of the kill chain.  Their chart headers looked similar to the one below.

<table>
    <caption>Brotherston-Berlin Use Case Format [5]</caption>
    <thead>
       <tr>
        <th>Kill Chain Step</th>
        <th>Malicious Action</th>
        <th>Defensive Mitigation</th>
        <th>Potential Monitoring</th>
    </tr> 
    </thead>
</table>

Our needs here are a little different, so we'll modify our chart while following along from their example.  Since this is not a real attack; and since our view is mainly from the penetrator's perspective, we'll abbreviate a summary of some of the steps by relating an attacking action to a step in the kill chain and an observation, using tables like the one below.  As we can see, this format is similar to descriptions we might find about the phases of pentesting \[[26]\]

<table>
    <caption>Command to Kill Chain Step Summary</caption>
    <thead>
       <tr>
        <th>Kill Chain Step</th>
        <th>Attacking Action</th>
        <th>Observation</th>
    </tr> 
    </thead>
    <tbody>
    <tr>
        <th>Reconnaissance</th>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <th>Weaponization</th>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <th>Delivery</th>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <th>Exploitation</th>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <th>Installation</th>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <th>Command and Control</th>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <th>Actions and Objectives</th>
        <td></td>
        <td></td>
    </tr>
    </tbody>
</table>

![NIST 4 phases]({{ site.url }}/assets/images/KaliBadStore/NISTfourPhases.png)

Adapting a vernacular of different phases is a common practice.  As we can see from NIST publications like 800-115, similar analysis mechansims might divide up an operation into four phases; \[[36]\] some other evaluators might use five or more. \[[26]\]  So long as we can be clear with our recipients, the breaking up of the anaylsis into phases may not matter.

As we go through some of the phases of the hack, we'll cover some of the NIST-style Discovery and Attack cycles.  In retrospect, this fit a chronology of our actual actions better than the Kill Chain style of analysis.  So, as some of the Discovery and Attack cycles build up information, we'll see them grouped together in some subsections of the Kill Chains.  

The Planning phase of the penetration testing methodology was almost nonexistent.  We decided not to consult any in-depth directions because in the past those seemed to spoil the discovery.  The plans were boiled down to three simple rules:
- Do not read the directions.
- Always read the references.
- Wait two days before deciding to give in and read the answer.

Those rules held.  Likewise, our reporting phase is just this blog entry.  By the time we got through the exercise, we had about five distinct Discovery and Attack cycles that were naturally following the suggested pattern of discovery, attack, and additional discovery.

Early on, we wanted to see if we could actually make modifications to repair, modify, or defend that VM, after our penetrations.  In the beginning, that goal had to remain part of our ambitions, as we worked through getting in to the box and learning about the website that's covered in the VM.  This first step of vulnerability scanning began our first Discovery and Attack cycle.


## Vulnerability Scanning "Bad Store"
To support a Reconnaissance phase, we conducted four kinds of vulnerability scans.  Two were Nessus scans (basic network and web applications); one was a collection on `nmap` scans of TCP and UDP protocol ports; another was a `nikto` scan.  We also did some manual observation of the website (directory traversal and simple injection probing), and a `sqlmap` scan; those will be covered separately.  It's obvious that this was very noisy reconnaissance; but, there are no points for stealth going against a home lab VM.  Let's look at what we can learn from these scans.

### `Nessus` scans for the site
Given some basic directions for running a Nessus scan,  \[2\] we ran a couple of them against the "Bad Store" VM.  A quick skimming of those results showed several OpenSSL-related vulnerabilities; that's when we began to see how many of the vulns might be historic.  Also listed was a vuln related to an old Apache version.   

Copies of the Nessus scan results that we ran against the VM are available through these links:
- [PDF of Nessus scan using the Basic Network scan type](https://github.com/gladiola/blackmagic/blob/Demo/agkistrodon/contortix/Penelope/BadStore/Nessus/salvage13_BasicNetwork_BadStore_ya6pkz.pdf)
- [PDF of Nessus scan using the Web Application scan type](https://github.com/gladiola/blackmagic/blob/Demo/agkistrodon/contortix/Penelope/BadStore/Nessus/salvage13_BadStore_Web_e1yrt4.pdf)

We clicked around some to look up those vulnerabilities on hyperlinks related to the Tenable website; while well supported with CVE documentation and the like, there were easier to exploit possibilties likely.  We continued with some of the other vuln scans like `nmap`.

### `nmap` scan for TCP, UDP, and versions
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

So, what does this tell us?  We can see that those three ports are open.  This will let us see that we will need to focus on programs that serve HTTP(S) and MySQL.  Trying to attack other applications will probably be fruitless.  

## `nikto` scanning
![nikto recommendations]({{ site.url }}/assets/images/KaliBadStore/2018-07-07-183641_1366x768_scrot.png)

Quick and easy, `nikto` scanning did a directory traverse that seemed easy to use.  It coughed up five or six directories to check into.  For brevity, We'll show what we eventually found with some of those.  

The supplier directory was referred to in robots.txt.  Looking up robots.txt showed that a user-agent of a certain name would not be disallowed.  This looked useful.  Also, robots.txt mentioned an `/upload` directory.  That, combined with a file upload dialog, implied an remote or local file inclusion vulnerability possibility.  We could also see how directory rules woudl cycle and recycle scanning bots of the wrong type.  To understand this, we had to compare robots.txt alongside a Javascript script.

The `cgi-bin/` showed a couple of things.  First, laying around was a `test.cgi` file.  Later, after finding some hashes, it would match up with a sysadmin's account.  Apparently, this was a leftover test laying around in the plain.  It showed clearly that there were base64 and MD5 hashes in use; but, that would be close to quickly recognized by the shape of the strings found later.

There was a `/supplier/accounts` file laying around.  This was a text file with about four lines associating a number with a base64 encoded string.  Recognizable by its trailing "=", the base64 was quickly decoded.  

{% highlight shell %}
echo <TARGET STRING> | base64 --decode
{% endhighlight %}

The output showed a pattern like:
`<100X>:<USER>/<PASSWORD>/<?OPTIONAL METAL WORD>/<IP ADDRESS>`
These later proved to be an association between item numbers, a supplier, their password, and IP.  However, the site used email addresses to run the logins, so these account rows did not get us into anyone's account.

## Manual directory checks and source code reading
One of the first things we can do is to look at the website.  
- Are there any forms?  
- Does it react to URL encoding?  
- Does the source code seem to pull down any outside scripts?  
- Where are other assets stored?  

Given those questions, we can go hunting for plenty of vulnerabilities without any scanning tools.  What turns up?

In summary, every possible page deserved a review of its visible source code.  What inputs were there?  What files were called as a part of making the website page possible?  Following these ideas also led us into a credit card validation routine in Javascript that might offer some opportunities later.

## Tickmark 1 equals 1
![sqli error from one equals one]({{ site.url }}/assets/images/KaliBadStore/2018-07-07-210740_1366x768_scrot.png)

Almost every single text input I tried showed some kind of vulnerability to `' OR '1'='1' `.  Often, it showed an error.  In the case of the CGI guestbook, it accepted the call as text and displayed it on th eweb page without throwing the usual errors.  When placed in the supplier login text input, the `'1=1` hack would make the file upload dialog pop up.  Maybe useful later.

## Some handy facts laying out in the plain
A look at the source code of each page revealed that a lot of form processing was being donw in CGI.  Much luck for me; I never got into CGI.  So, that lead would require more research to use.  

 ![Clues left lying around]({{ site.url }}/assets/images/KaliBadStore/CluesLeftLyingAround.png)

Meanwhile, it also turned up another script, `frmvrfy.js` that compares two password values.  Apparently part of the reset routine.  This was a program I felt we should exploit with our Javascript skills, but we set it aside for the time being.  

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
            <td>Nessus Basic Network scan</td>
            <td>Historic vulnerabilities uncovered</td>
        </tr>
        <tr>
            <td>Nessus Web scan</td>
            <td>Historic vulnerabilities uncovered</td>
        </tr>
        <tr>
            <td>nmap</td>
            <td>Age of server versions will affect later exploits</td>
        </tr>
        <tr>
            <td>nikto</td>
            <td>Stubbed-out directories and loose files with clues</td>
        </tr>
        <tr>
            <td>Manual probing</td>
            <td>Relationship of inputs to CGI program and initial input validation guesses</td>
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
        <td>Nessus Basic Network</td>
        <td>Apache 1.3 and OpenSSL < 0.9</td>
    </tr>
    <tr>
        <td>Nessus Web</td>
        <td>Apache 1.3 and OpenSSL < 0.9</td>
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
        <td>Manual probing</td>
        <td>Exposed files and injectable inputs</td>
    </tr>
    </tbody>
</table>

## Minor stump
At this point, I had a lot of information, but no real login.  I didn't want to give in and read the provided directions that are on the site.  Time to plink around a little more and see what I could do.  Overall, I felt that I should be getting a login and a chance to see veryone's account history from the site.  Not being able to do that was a little discouraging.  I would have to find a way to ge that somehow.

I tried a couple of Metasploit modules; but, really, a moment of success came by running some of the MySQL-related auxilliaries.

## We got the gold
![msf auxilliary schema dump]({{ site.url }}/assets/images/KaliBadStore/2018-07-07-184427_1366x768_scrot.png)

By running some MySQL auxilliaries with `msfconsole`, we were able to send SQL directly to the MySQL engine, dump schema, and `SELECT` a bunch of useful content.  One of the results was that we were able to get database records for usernames as email addresses (this particular website logs users in by email addy), passwords, bank account numbers, and detailed transaction information. 

By `SELECT`ing those email addresses and passwords from the userdb table, `msfconsole` was able to output a simple text file that could be trimmed and used as input for `hashcat`. 

![screenshot of user accounts and passwords]({{ site.url }}/assets/images/KaliBadStore/BadStorePasswords.png)

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


## Next goals
At this point, it was time to set some new goals.  Thoroughly scanned, with a database coughing up pretty much whatever we wanted, the site would yield whatever information simple account impersonation might provide.  That said, it was not enough.  A lot of what was done thus far was passive.  A malicious attacker would shape and control the box so that it could be used for her own ends.  

## Using MySQL to call the target db
At this point, we had just enough information to see if we could call up the database and poke around and see what we could find.  The Metasploit Framework auxilliary scripts had already shown us that our computer could get in.  How could we mimic that with our own skills?  Can we use our normal database administration skills to do some damage to this instance of MySQL on the target box?  Or, maybe, can we just set up things to run the way we want instead of the way they were supposed to?

![mysql error from calling select db zero]({{ site.url }}/assets/images/KaliBadStore/2018-07-07-195544_1366x768_scrot.png)

Even if we had only manually attempted to probe the website, we could still learn some database information from our past injection calls.  Given the example above, we can see that simple calls could allow us to plink away at database tables and eventually learn about the columns.  Compared to the schema dump commands that we saw through `msfconsole`, we can see that there'd come a point when we'd want to just get in there and start administering the box.  


For the first try, we can call up an instance of MySQL on the target machine using commands just as if we were normally administering one of our own.  
![mysql status from direct call to remote db engine]({{ site.url }}/assets/images/KaliBadStore/2018-07-07-192718_1366x768_scrot.png)


While on the machine, we will want to be able to do things like install another user \[[12]\], grant permissions \[[13]\], and maybe even change the password to our discovered "sysadmin" account on the target box.  Most of these user-installation activities will involve a `GRANT` command; when we try to give similar commands, we can see this warning about `--skip-grant-tables`.  For now, this basically foils our ability to tamper with the database engine accounts.  Meanwhile, if we were ambitious, we might find a way to modify the initialization files for the MySQL engine to hack our way past this constraint and reset root. \[[15]\] \[[16]\]  But, this brings us back around to what we still need to do:  exploit the box.  Perhaps we can try this in post-exploitation.
![mysql skip grant tables]({{ site.url }}/assets/images/KaliBadStore/2018-07-07-204857_1366x768_scrot.png)

## Preparing to use sqlmap
Perhaps a return to automated hacking tools will help us.  Earlier, default runs of `sqlmap` didn't have enough information to do anything effectuve.  However, now, we have learned some facts about the website so that we can use `sqlmap` in an effective way.  Namely, we have discovered on our own some injectable controls so that `sqlmap` can have a chance at exploiting them.  The pictures below show some of the details we were able to discover through simple observation of the forms with an ordinary browser.

![identifying form action url and injectable controls]({{ site.url }}/assets/images/KaliBadStore/2018-07-07-211524_1366x768_scrot.png)
In the picture above, we see finding the URL that the form uses for submission.  In the picture below, we see the discovery of the button name that sets off that form submission.  Since our form is submitted with a POST action, our steps are a little different from a common GET submission.  \[[9]\]\[[12]\]

![identifying form submit button name]({{ site.url }}/assets/images/KaliBadStore/2018-07-07-212240_1366x768_scrot.png)

To make our life easier, we can use Burp Suite's proxy readouts to show us what the submitted POST will look like. [12]  From there, we'll save a copy of that POST message to a file, alter it slightly, and use it alongside our `sqlmap` to give the script a way to get an exploit installed.  \[[9]\]\[[10]\]\[[11]\]\[[12]\]

![sqlmap confirms injectable control]({{ site.url }}/assets/images/KaliBadStore/2018-07-07-214245_1366x768_scrot.png)
In the picture above, we see that `sqlmap` has been able to identify the injectable control.  

![sqlmap displays suggested exploit calls]({{ site.url }}/assets/images/KaliBadStore/2018-07-07-214600_1366x768_scrot.png)
In the picture above, we see what `sqlmap` suggests for use as a command that would allow us to pawn the box.  The program determined that these commands were good choices after doing some methodical testing with repetitive calls.  It's a noisy process, as with most scans, but we can see that the program was able to recommend these prefabricated calls that it knows will work.

## Preparing to exploit the target box
From here, we would normally allow `sqlmap` to prepare a prefabricated exploit.  The program would ask us a simple quiz dialog about what kind of technology we think the target server might be using.  ASP, JSP, and PHP are common targets.  However, in this case, I think our target box is running an HTML/CGI config that probably uses Perl.  A quick look around didn't reveal any simple choices.  Instead, it seems to me that the thing to do will be to break into the box in a way that lets us use Perl to write and install our own Perl scripts; or, maybe do something similar with `sh` for UNIX scripts; run them; and then use the browser to navigate to predetermined places with a URL in order to read the output.  

Time to learn a little more about Perl.

## Using `sqlmap` to upload and download files 
Near this time, we entered a phase where we were basically exploiting the box.  By being able to upload and download files, we had a means of obtaining data and trying to plant data.  Throughout the exercise, I failed to write data to the directory as desired.  We were able to write to the database, but not the immediate Linux file structure.  Some sample techniques are below.

{% highlight shell %}
sqlmap -d `mysql://root:secret@192.168.1.9:3306/badstoredb --file-read="/usr/local/apache/cgi-bin/badstore.cgi"
{% endhighlight %}
An example of reading a file from the target box using `sqlmap` and a direct database connection.

{% highlight shell %}
sqlmap -d `mysql://root:secret@192.168.1.9:3306/badstoredb --file-dest="/usr/local/apache/cgi-bin/hello.cgi" --file-write="hello.cgi" --user-agent="badstore_webcrawler"
{% endhighlight %}
An example of attempting to write a file from the attacker box's current directory to a CGI-BIN directory in Apache 1.3 on the target box using `sqlmap` with a direct database connection.  We applied the `--user-agent` argument to match a value that turned up in analyzing the `robots.txt` file.  In there, it became plain that some User-Agents were permitted in some directories and others not.  Adjusting this was an attempt to avoid a denial.

{% highlight shell%}
sqlmap -r badStore_postEmail.txt -p email --threads=10 --dbms=mysql -D badstoredb -v 5
{% endhighlight %}
An example of using `sqlmap` to read a POST message and apply its contents to a parameter, using `-p <PARAMETER_NAME>`.  The POST message was previously intercepted with Burp Suite and then copied to a text file.  That text file was edited to adjust some parameters.  Using these techniques, a POST message can be edited and applied with `sqlmap` to try to gain access specific to certain user accounts, cookies, etc., based on the design of the site's input validation checks in the receiving program.  In this way, if we wanted to use common anonymous access to use information we had or suspected about an admin account, we might try to forge our way into an admin login.  

This kind of POST file read technique can be combine with the other `--file-dest` and `--file-write` arguments in `sqlmap` to try to upload a file as a given user.

### Analysis Check
<table>
    <caption>Discovery and Attack Cycle 3</caption>
    <thead>
        <tr>
            <th>Action</th>
            <th>Observation</th>
        </tr> 
    </thead>
    <tbody>
        <tr>
            <td>Burp and sqlmap</td>
            <td>Bypass POST requirement to mimic expected form actions</td>
        </tr>
        <tr>
            <td>File uploads and downloads</td>
            <td>Understanding what was where</td>
        </tr>
        <tr>
            <td>sqlmap scans to determine SQLi</td>
            <td>Much more complex attack than if derived manually</td>
        </tr>
    </tbody>
</table>

<table>
    <caption>Command to Kill Chain Step 3</caption>
    <thead>
       <tr>
        <th>Kill Chain Step</th>
        <th>Attacking Action</th>
        <th>Observation</th>
    </tr> 
    </thead>
    <tbody>
    <tr>
        <th rowspan="3">Weaponization</th>
        <td>sqlmap</td>
        <td>Automated discovery of effective SQLi</td>
    </tr>
    <tr>    
        <td>POST forms intercepted with Burp</td>
        <td>Changing expected information</td>
    </tr>
    <tr>
        <td>POST forms edited and staged</td>
        <td>Efficient reloading of altered data for repetitive attack attempts</td>
    </tr>
    <tr>
        <th rowspan="2">Delivery and Exploitation</th>
        <td>sqlmap</td>
        <td>Command line uploads of files</td>
    </tr>
    <tr>
        <td>sqlmap</td>
        <td>Command line downloads of files</td>
    </tr>
    <tr>
        <th>Reconnaissance</th>
        <td>sqlmap storage</td>
        <td>Simple observation of file transfers began to reveal write-permission problems </td>
    </tr>
    </tbody>
</table>

## Using direct connections with MySQL
In addition to being able to use `sqlmap`, at one point I noticed that what account I was using to smash my way in just didn't matter.  That is, once we could get it with SQLi on a known injectable control, what account we were using didn't have a bearing on what we could do.  For example, `msfconsole` had some interesting tools which worked just fine to reveal the schema and send some SQL commands to the database engine.

{% highlight shell %}
use auxiliary/scanner/mysql/mysql_schemadump
show options
exploit

use auxiliary/admin/mysql/mysql_sql
show options
set RHOST 192.168.1.9
set SQL "SELECT * FROM badstoredb.acctdb"
exploit

set SQL "show columns"
exploit
{% endhighlight %}
`msfconsole` commands like these came in handy to reveal most of what we wanted to know about the database supporting the dynamic website. \[[27]\]  Given these simple utilities, we were able to get any table we wanted and to see the entire schema.  Some of the prefabricated scripts for attacking this older version of MySQL database didn't work well because of the lack of an `INFORMATION_SCHEMA` database and standardized tables to cover the meta-structure of the databaases; but, that didn't stop our ability to glean the information we needed using the techniques above and some simple manual processes.  

Aroung this time we realized there was nothing to lose from just calling `mysql` with common remote commands.  Given an IP address, known port, and working username and password combination, it was no different than calling a remote instance of `mysql` normally.  This got us the usual command prompt and we could proceed without the fuss of even bothering with `msfconsole auxiliary`.

### Analysis Check
<table>
    <caption>Discovery and Attack Cycle 4</caption>
    <thead>
       <tr>
            <th>Action</th>
            <th>Observation</th>
        </tr> 
    </thead>
    <tbody>
        <tr>
            <td>Manual input</td>
            <td>Impersonate users and observe account differences</td>
        </tr>
        <tr>
            <td>MySQL console</td>
            <td>Questions arose about skip-grant-tables and db account permissions</td>
        </tr>
        <tr>
            <td>sqlmap uploads</td>
            <td>Began seeking alternatives to file installation</td>
        </tr>
    </tbody>
</table>

<table>
    <caption>Command to Kill Chain Step 4</caption>
    <thead>
       <tr>
        <th>Kill Chain Step</th>
        <th>Attacking Action</th>
        <th>Observation</th>
    </tr> 
    </thead>
    <tbody>
    <tr>
        <th rowspan="1">Command and Control</th>
        <td>sqlmap uploads</td>
        <td>Attempted uploads foiled; shell installation unlikely</td>
    </tr>
    <tr>
        <th rowspan="2">Actions on Objective</th>
        <td>User Accounts</td>
        <td>Decrypted passwords and known user accounts permit impersonation of any user on the site</td>
    </tr>
    <tr>
        <td>sqlmap</td>
        <td>Gain more knowledge of MySQL engine setup with console-style commands to the database engine</td>
    </tr>
    </tbody>
</table>

## Follow-on uploads and actions
We were able to try to upload a Perl script file to the CGI-BIN directory; and, judging by the error messages, the transmissions failed.  I attempted several file upload actions through `sqlmap` that ended in failures.  No matter where or how I attempted to write, we were denied permission by the target computer.  During these attempts, I realized just how valuable it could be to have database accounts that were totally foreign to the permissions required for file writing.  Several of the error messages that `sqlmap` returned showed us this would be the case.

![screenshot of denied file writes]({{ site.url }}/assets/images/KaliBadStore/403OnUploadedFiles.png)

By viewing 403s like this, I wondered if the files were written but just not provided with the correct permissions.  Every time I found that this was not the case.  It's likely that the permissions on the directory were set to completely deny the file writes altogether.  In those instances, I had expected to see a 404, but my requests returned a 403.  It's likely that the short-circuit on the status codes simply encountered a sooner reason for denial. \[[21]\]\[[22]\]\[[23]\]\[[24]\]\[[25]\]

At one point, our goals included the idea of writing scripts to add a user, modify an existing user, or otherwise misuse current user information. \[[35]\] \[[36]\]

![screenshot of failure to implement chmod through file upload dialog]({{ site.url }}/assets/images/KaliBadStore/chmodFailure.png)
In the example above, I failed to get a `chmod` command to run against a file I had uploaded through that dialog.

Still, I had a dream of being able to penetrate that box, pop a shell, and then begin updating it from the outside.  I really wanted to leave that box better off than when I found it.  

To that end, I attempted to gain more information about the box. With the entire database schema and selected tables downloaded, I still wanted more modifications.  

I began downloading selected files and programs in an attempt to gain more information. \[[31]\] First, I downloaded some of the CGI programs I had been poking against.  I was a little surprised at what I saw in the `badstore.cgi`.  There were a couple of surprises I hadn't touched.  Next, I began hunting for system files.  I never did find the desired boot file; I checked for cronjobs; I downloaded `/etc/passwd`, `/etc/fstab`, `/etc/hosts`, `/etc/profile` and others.  Generally, the empty files with just a null byte that came down would indicate that nothing was found at the address I requested.

![sqlmap output files]({{ site.url }}/assets/images/KaliBadStore/sqlmapOutputFiles.png)
`sqlmap` automatically catalogs downloaded files in its own hidden directory.  We navigated there and took a look.  It automatically converted directory slashes to underscores in the filenames.  To find out if there was anything in the file, all we had to do was to see if the file size was more than 4 bytes.  The four byte files simply held a null character.  In those cases, we had attempted to download a file that was either not there or not available. 

### Analysis Check
<table>
    <caption>Discovery and Attack Cycle 5</caption>
    <thead>
        <tr>
            <th>Action</th>
            <th>Observation</th>
        </tr> 
    </thead>
    <tbody>
        <tr>
            <td>sqlmap downloads</td>
            <td>Determined the two user accounts and generally observed system files</td>
        </tr>
        <tr>
            <td>sqlmap, msfconsole uploads</td>
            <td>Could not install a file or detect a running service besides the Perl CGI already there</td>
        </tr>
    </tbody>
</table>


<table>
    <caption>Command to Kill Chain Step 5</caption>
    <thead>
        <tr>
            <th>Kill Chain Step</th>
            <th>Attacking Action</th>
            <th>Observation</th>
        </tr> 
    </thead>
    <tbody>
        <tr>
            <th>Reconnaissance</th>
            <td>Analyze downloaded files</td>
            <td>Observe critical values; notice missing or empty files</td>
        </tr>
        <tr>
            <th>Exploitation</th>
            <td>sqlmap</td>
            <td>Fetch system files to learn about system accounts and file structure</td>
        </tr>
        <tr>
            <th>Installation</th>
            <td>sqlmap, msfconsole</td>
            <td>All attempts to upload shellcode failed</td>
        </tr>
        <tr>
            <th>Actions on Objective</th>
            <td>Linux File System</td>
            <td>Multiple attempts to discern possible foothold on the target box by downloading key files.</td>
        </tr>
    </tbody>
</table>

<table>
    <caption>Kill Chain Summary</caption>
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
            <td>Nessus Basic Network</td>
            <td>Apache 1.3 and OpenSSL < 0.9</td>
        </tr>
        <tr><td>Nessus Web</td><td>Apache 1.3 and OpenSSL less than 0.9</td></tr>
        <tr><td>nmap</td><td>Open ports, server versions</td></tr>
        <tr><td>nikto</td><td>Diretories and open ports</td></tr>
        <tr><td>Manual probing</td><td>Exposed files and injectable inputs</td>
        </tr>
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
            <td>SQLi </td>
            <td>Injectable controls all over</td>
        </tr>
        <tr>
            <th rowspan="3">Weaponization</th>
            <td>sqlmap</td>
            <td>Automated discovery of effective SQLi</td>
        </tr>
            <td>POST forms intercepted with Burp</td>
            <td>Changing expected information</td>
        </tr>
        <tr>
            <td>POST forms edited and staged</td>
            <td>Efficient reloading of altered data for repetitive attack attempts</td>
        </tr>
        <tr>
            <th rowspan="2">Delivery and Exploitation</th>
            <td>sqlmap</td>
            <td>Command line uploads of files</td>
        </tr>
        <tr>
            <td>sqlmap</td>
            <td>Command line downloads of files</td>
        </tr>
        <tr>
            <th>Reconnaissance</th>
            <td>sqlmap storage</td>
            <td>Simple observation of file transfers began to reveal write-permission problems </td>
        </tr>
        <tr>
            <th rowspan="1">Command and Control</th>
            <td>sqlmap uploads</td><td>Attempted uploads foiled; shell installation unlikely</td>
        </tr>
        <tr>
            <th rowspan="2">Actions on Objective</th>
            <td>User Accounts</td>
            <td>Decrypted passwords and known user accounts permit impersonation of any user on the site</td>
        </tr>
        <tr>
            <td>sqlmap</td>
            <td>Gain more knowledge of MySQL engine setup with console-style commands to the database engine</td>
        </tr>
        <tr>
            <th rowspan="1">Reconnaissance</th>
            <td>Analyze downloaded files</td>
            <td>Observe critical values; notice missing or empty files</td>
        </tr>
        <tr>
            <th rowspan="1">Exploitation</th>
            <td>sqlmap</td>
            <td>Fetch system files to learn about system accounts and file structure</td>
        </tr>
        <tr>
            <th rowspan="1">Installation</th>
            <td>sqlmap, msfconsole</td>
            <td>All attempts to upload shellcode failed</td>
        </tr>
        <tr>
            <th rowspan="1">Actions on Objective</th>
            <td>Linux File System</td>
            <td>Multiple attempts to discern possible foothold on the target box by downloading key files.</td>
        </tr>
    </tbody>
</table>


## Closing session

 ![badstore.cgi admin portal]({{ site.url }}/assets/images/KaliBadStore/Capture_secretAdminPortal.png)

 ![badstore.cgi log files]({{ site.url }}/assets/images/KaliBadStore/logs.png)
 
 ![badstore.cgi database connection]({{ site.url }}/assets/images/KaliBadStore/Capture_dbConnectionInCGI.png)
 

 Downloading the `badstore.cgi` file itself yielded a wealth of information.  Pretty much the heart of the entire exercise, this program showed us some other directories to check up on.  It also showed us some hardcoded password values and some program aspects that we didn't hit on by ourselves.  There were other features in there available for exploit.  However, since most of them were encompassed by the program, we chose not to follow those leads at the time.  

I continued looking for a mechanism to pop a shell with.  With no JSP, ASP, or PHP server-side programs enabled, there were slim pickings.  The MySQL was in a version below 5.0; so, there was no "INFORMATION_SCHEMA" to play with.  Metasploit modules for Heartbleed and Shellshock weren't working. \[[28]\] \[[29]\] \[[30]\] \[[33]\]  After some time, I had to recognize that it was fruitless to continue further exploitation attempts.  As far as I can tell, I wasn't going to get any further in on this one.  

## Lessons learned from "Bad Store"
My dominant impression was that Kurt Roemer built a pretty strong VM.  Despite its age, and subsequent vulnerabilities that emerged long after he published it, we couldn't break into the system itself.  The vulns that were there for our exploitation were really the only major holes in the system.  A big part of these exercises is learning about what makes a strong system.  Not being able to write files where we wanted was the most impressive factor about the system we were attacking.  

While we don't know for sure, let's suppose some of the qualities that the VM may have had that allowed for this strength to happen.  First, there was a clear separation of user accounts between the database engine and the file system.  Second, it's supposed that write permissions were removed from most directories.  It seems as if the site were built with a user account that was later removed.  There were few open ports and no common services.  Contemporary languages and programs that are commonly exploited just weren't there.  The Perl CGI made explicit checks for keywords; its monolithic structure hampered our attempts at finding assumed included components.  Despite certain vulnerabilities and poor coding practices like hard-coded passwords, the system didn't let us get much beyond the application's weakness.  That is, while we were able to break in the database and steal all the data; still, it showed the possibility of a strong site by rebuffing our attempts to break in to the underlying file system.  Overall, it was a good challenge.  Along the way, we learned a few things to consider the next time we see sites with CGI.


## Annotated Bibliography
\[1\]  Weidman, Georgia.  "Chapter 4:  Using the Metasploit Framework," Penetration Testing:  A Hands-On Introduction to Hacking.  No Starch Press.  pp.87-109.  ISBN 978-1-89327-564-8.

Review of techniques to implement a simple attack with default settings with a metasploit module using the <code>msfconsole</code>. 

\[2\]  Weidman, Georgia.  "Chapter 6:  Finding Vulnerabilities," Penetration Testing:  A Hands-On Introduction to Hacking.  No Starch Press.  pp.133-153.  ISBN 978-1-89327-564-8.

Review of techniques to implement Nessus scans, <code>nmap</code> scans, and <code>nikto</code> scans. 

\[3\] Weidman, Georgia.  "Chapter 14:  Web Application Testing," Penetration Testing:  A Hands-On Introduction to Hacking.  No Starch Press.  pp.313-338.  ISBN 978-1-89327-564-8.

Review of techniques to implement substitution of values sent with Burp Suite, using <code>sqlmap</code> for SQLi, and understanding local and remote file inclusion attacks.

\[4\]  Roemer, Kurt R.  "Bad Store 1.2.3," Vulnhub.  INTERNET:  [`https://www.vulnhub.com/entry/badstore-123,41/`](https://www.vulnhub.com/entry/badstore-123,41/) 

Website listing details of the VM on Vulnhub.  Includes screenshots and links to download URLs.

\[5\]  Brotherston, Lee and Amanda Berlin.  Defensive Security Handbook:  Best Practices for Securing Infrastructure.  O'Reilly, April 2017. pp. 6-9. ISBN 978-1-491-96038-7. 

A description of Lockheed-Martin's Intusion Kill Chain.  Brotherston and Berlin applied their use case of an overview of a ransomware attack to the phases of Lockheed's kill chain.  Their definitions and example serves as guidance for the contstruction of our own, here.

\[6\] _____.  "What is a potfile?" Hashcat Wiki.  INTERNET:  [`https://hashcat.net/wiki/doku.php?id=frequently_asked_questions#how_can_i_show_previously_cracked_passwords_and_output_them_in_a_specific_format_eg_emailpassword`](https://hashcat.net/wiki/doku.php?id=frequently_asked_questions#how_can_i_show_previously_cracked_passwords_and_output_them_in_a_specific_format_eg_emailpassword)

\[7\] _____.  "How can I show previously cracked passwords, and output them in a specific format (e.g. email:password)?" Hashcat Wiki.  INTERNET:  [`https://hashcat.net/wiki/doku.php?id=frequently_asked_questions#how_can_i_show_previously_cracked_passwords_and_output_them_in_a_specific_format_eg_emailpassword`](https://hashcat.net/wiki/doku.php?id=frequently_asked_questions#how_can_i_show_previously_cracked_passwords_and_output_them_in_a_specific_format_eg_emailpassword)

\[8\]  _____.  "Hashcat reports "Status: Cracked", but did not print the hash value, and the outfile is empty. What happened?" Hashcat Wiki.  INTERNET: [`https://hashcat.net/wiki/doku.php?id=frequently_asked_questions#hashcat_reports_statuscracked_but_did_not_print_the_hash_value_and_the_outfile_is_empty_what_happened`](https://hashcat.net/wiki/doku.php?id=frequently_asked_questions#hashcat_reports_statuscracked_but_did_not_print_the_hash_value_and_the_outfile_is_empty_what_happened)

\[9\] Abouzeid, Halim.  "From SQL Injection to WebShell"  RSA.  INTERNET:  [`https://community.rsa.com/community/products/netwitness/blog/2017/04/10/from-sql-injection-to-webshell`](https://community.rsa.com/community/products/netwitness/blog/2017/04/10/from-sql-injection-to-webshell)

Step by step tutorial on using `sqlmap` to pop a shell on a target box.  Shows both the use of automated exploits for JSP, ASP, and PHP, and also the use of uploading custom scripts.  

\[10\] _____.  INTERNET:  [`https://w00troot.blogspot.com/2017/05/getting-reverse-shell-from-web-shell.html`](https://w00troot.blogspot.com/2017/05/getting-reverse-shell-from-web-shell.html)
Some reverse shell script suggestions for various systems.  

\[11\] _____.  INTERNET:  [`https://www.darkmoreops.com/2014/08/28/use-sqlmap-sql-injection-hack-website-database/`](https://www.darkmoreops.com/2014/08/28/use-sqlmap-sql-injection-hack-website-database/)
A tutorial showing some `sqlmap` hacks by injection.  Includes another example of get a hashed password and crack it using a modified form of `hashcat`.

\[12\] _____.  INTERNET:  [`https://sneakerhax.com/tool-tips-sqlmap-with-post-requests/`](https://sneakerhax.com/tool-tips-sqlmap-with-post-requests/)
Tutorial showing some steps for using Burp Suite to pick up POST messages to injectable programs and modify them for use with `sqlmap`.

\[13\]  INTERNET:  [`https://dev.mysql.com/doc/refman/5.5/en/create-user.html`](https://dev.mysql.com/doc/refman/5.5/en/create-user.html)
MySQL Handbook for a later version; commands similar to what's installed on the target box.  Commands for creating a new user with MySQL.

\[14\]  INTERNET:  [`https://dev.mysql.com/doc/refman/5.5/en/grant.html`](https://dev.mysql.com/doc/refman/5.5/en/grant.html)
Commands for GRANT with MySQL.  Critical basic commands for setting privileges on an account with MySQL.

\[15\]  INTERNET:  [`https://stackoverflow.com/questions/1708409/how-to-start-mysql-with-skip-grant-tables`](https://stackoverflow.com/questions/1708409/how-to-start-mysql-with-skip-grant-tables)
Stack Overflow post suggesting that skip grant tables might be overridden with reconfiguring some ini files.

\[16\]  INTERNET:  [`https://superuser.com/questions/1127299/how-to-restart-mysql-with-skip-grant-tables-if-you-cant-use-the-root-password`](https://superuser.com/questions/1127299/how-to-restart-mysql-with-skip-grant-tables-if-you-cant-use-the-root-password)
Suggestion that bypassing the skip grant tables arguments can be used in a risky operation to reset the ROOT account password with MySQL.

\[17\]  INTERNET:  [`http://seclists.org/metasploit/2012/q3/40`](http://seclists.org/metasploit/2012/q3/40)
Got turned on to `spool` for Metasploit commands.

\[18\]  INTERNET: [`http://www.guninski.com/modproxy1.html`](http://www.guninski.com/modproxy1.html)
Post on `mod_proxy`

\[19\]  INTERNET: [`https://hack2rule.wordpress.com/2017/02/25/sql-injection-to-meterpreter/`](https://hack2rule.wordpress.com/2017/02/25/sql-injection-to-meterpreter/)
Blog post on using `sqlmap`, `Meterpreter`, `Burp` and others.

\[20\]  INTERNET: [`https://github.com/sqlmapproject/sqlmap/wiki/Usage`](https://github.com/sqlmapproject/sqlmap/wiki/Usage)
`sqlmap` github site explaining the usage of the arguments; man pages.

\[21\]  INTERNET: [`https://www.tutorialspoint.com/perl/perl_cgi.htm`](https://www.tutorialspoint.com/perl/perl_cgi.htm)
Tutorial on simple Perl CGI scripts.  These would have been the basis for some of our initial attempts to write scripts to the box.  

\[22\]  Kamthan, Pankaj.  INTERNET: [`https://www.irt.org/articles/js184/#origins_consequences`](https://www.irt.org/articles/js184/#origins_consequences)
Article explaining some of the details of CGI security.  This looked like it would offer some good advice on setting up CGI scripts.

\[23\]  INTERNET: [`https://httpd.apache.org/docs/2.4/howto/cgi.html`](https://httpd.apache.org/docs/2.4/howto/cgi.html)
Apache documentation on setting up CGI with Apache 2.4.

\[24\]  INTERNET: [`https://www.techrepublic.com/article/get-it-done-installing-apache-web-server-on-linux/`](https://www.techrepublic.com/article/get-it-done-installing-apache-web-server-on-linux/)
An overview of instructions on how to install Apache 1.3.  This provided a quick reference on what might have been pertinent to the author of the VM when installing the web server we were attacking.

\[25\]  INTERNET: [`https://www.cybrary.it/forums/topic/how-to-bypass-error-403-forbidden/`](https://www.cybrary.it/forums/topic/how-to-bypass-error-403-forbidden/)
Forum posts on other people getting 403s.  No big surprises here; but some sound advice on a statement of the conditions.

\[26\]  INTERNET: [`https://www.cybrary.it/2015/05/summarizing-the-five-phases-of-penetration-testing/`](https://www.cybrary.it/2015/05/summarizing-the-five-phases-of-penetration-testing/)
Description of the five phases of pentesting.

\[27\]  INTERNET: [`https://www.offensive-security.com/metasploit-unleashed/scanner-http-auxiliary-modules/`](https://www.offensive-security.com/metasploit-unleashed/scanner-http-auxiliary-modules/)
Offensive Security's documentation on Metasploit's auxiliary modules.  Shows the modules in use, commonly adjusted settings, and a brief description of each.

\[28\]  INTERNET: [`https://www.rapid7.com/db/modules/auxiliary/scanner/ssl/openssl_heartbleed`](https://www.rapid7.com/db/modules/auxiliary/scanner/ssl/openssl_heartbleed)
Rapid7's description of an openssl_hearbleed scanner in Metasploit.

\[29\]  INTERNET: [`https://www.trojanhorsesecurity.com/shellshock`](https://www.trojanhorsesecurity.com/shellshock)
Trojan Horse Security's description of a Shellshock scanner and exploit in Metasploit.

\[30\]  INTERNET: [`https://www.rapid7.com/db/modules/exploit/multi/http/apache_mod_cgi_bash_env_exec`](https://www.rapid7.com/db/modules/exploit/multi/http/apache_mod_cgi_bash_env_exec)
Rapid7's description of a Shellshock exploit in Metasploit.

\[31\]  INTERNET: [`https://bytebin.wordpress.com/2011/04/25/how-to-check-debian-boot-log-messages/`](https://bytebin.wordpress.com/2011/04/25/how-to-check-debian-boot-log-messages/)
Blog post on where to find Debian boot logs.  Useful notes when we were trying to locate BadStore VM's boot-up information.

\[32\]  INTERNET: [`https://www.cvedetails.com/vulnerability-list/vendor_id-185/product_id-316/version_id-31799/Mysql-Mysql-4.0.25.html`](https://www.cvedetails.com/vulnerability-list/vendor_id-185/product_id-316/version_id-31799/Mysql-Mysql-4.0.25.html)
Web page on some CVEs related to early versions of MySQL.

\[33\]  INTERNET: [`https://packetstormsecurity.com/files/128447/Apache-mod_cgi-Bash-Environment-Variable-Code-Injection.html`](https://packetstormsecurity.com/files/128447/Apache-mod_cgi-Bash-Environment-Variable-Code-Injection.html)
Blog post outlining the source code for a custom Metasploit module to work against Apache mod_cgi Bash.

\[34\]  INTERNET: [`https://www.digitalocean.com/community/tutorials/how-to-use-passwd-and-adduser-to-manage-passwords-on-a-linux-vps`](https://www.digitalocean.com/community/tutorials/how-to-use-passwd-and-adduser-to-manage-passwords-on-a-linux-vps)
Tutorial on `/etc/passwd` that helped us understand what we saw when we downloaded a copy from BadStore.

\[35\]  INTERNET: [`https://www.tldp.org/LDP/sag/html/adduser.html`](https://www.tldp.org/LDP/sag/html/adduser.html)
Linux Documentation Project's page on how to add a user.

\[36\]  INTERNET: [`https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-115.pdf`](https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-115.pdf)
NIST documents includes descriptions of vulnerability scanning, penetration testing, password cracking, and other critical terms of applicable techniques.

[//]: # (Hyperlinks)
[1]:  ``
[2]: ``
[3]: ``
[4]:  https://www.vulnhub.com/entry/badstore-123,41/
[5]: ``
[6]: https://hashcat.net/wiki/doku.php?id=frequently_asked_questions#what_is_a_potfile
[7]: https://hashcat.net/wiki/doku.php?id=frequently_asked_questions#how_can_i_show_previously_cracked_passwords_and_output_them_in_a_specific_format_eg_emailpassword
[8]:  https://hashcat.net/wiki/doku.php?id=frequently_asked_questions#hashcat_reports_statuscracked_but_did_not_print_the_hash_value_and_the_outfile_is_empty_what_happened
[9]: https://community.rsa.com/community/products/netwitness/blog/2017/04/10/from-sql-injection-to-webshell
[10]: https://w00troot.blogspot.com/2017/05/getting-reverse-shell-from-web-shell.html
[11]: https://www.darkmoreops.com/2014/08/28/use-sqlmap-sql-injection-hack-website-database/
[12]: https://sneakerhax.com/tool-tips-sqlmap-with-post-requests/
[13]: https://dev.mysql.com/doc/refman/5.5/en/create-user.html
[14]: https://dev.mysql.com/doc/refman/5.5/en/grant.html
[15]: https://stackoverflow.com/questions/1708409/how-to-start-mysql-with-skip-grant-tables
[16]: https://superuser.com/questions/1127299/how-to-restart-mysql-with-skip-grant-tables-if-you-cant-use-the-root-password
[17]: http://seclists.org/metasploit/2012/q3/40
[18]: http://www.guninski.com/modproxy1.html
[19]: https://hack2rule.wordpress.com/2017/02/25/sql-injection-to-meterpreter/
[20]: https://github.com/sqlmapproject/sqlmap/wiki/Usage
[21]: https://www.tutorialspoint.com/perl/perl_cgi.htm
[22]: https://www.irt.org/articles/js184/#origins_consequences
[23]: https://httpd.apache.org/docs/2.4/howto/cgi.html
[24]: https://www.techrepublic.com/article/get-it-done-installing-apache-web-server-on-linux/
[25]: https://www.cybrary.it/forums/topic/how-to-bypass-error-403-forbidden/
[26]: https://www.cybrary.it/2015/05/summarizing-the-five-phases-of-penetration-testing/
[27]: https://www.offensive-security.com/metasploit-unleashed/scanner-http-auxiliary-modules/
[28]: https://www.rapid7.com/db/modules/auxiliary/scanner/ssl/openssl_heartbleed
[29]: https://www.trojanhorsesecurity.com/shellshock
[30]: https://www.rapid7.com/db/modules/exploit/multi/http/apache_mod_cgi_bash_env_exec
[31]: https://bytebin.wordpress.com/2011/04/25/how-to-check-debian-boot-log-messages/
[32]: https://www.cvedetails.com/vulnerability-list/vendor_id-185/product_id-316/version_id-31799/Mysql-Mysql-4.0.25.html
[33]: https://packetstormsecurity.com/files/128447/Apache-mod_cgi-Bash-Environment-Variable-Code-Injection.html
[34]: https://www.digitalocean.com/community/tutorials/how-to-use-passwd-and-adduser-to-manage-passwords-on-a-linux-vps
[35]: https://www.tldp.org/LDP/sag/html/adduser.html
[36]: https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-115.pdf

