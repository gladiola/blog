---
title:  "$5 Cloud Node to Bastion Host"
date:   2020-03-20 08:30:00
description: Hardening a cloud node to withstand normal Internet traffic.
---

Cloud providers hook us in with cheap teaser rates; and then they slowly offer us feature after feature, at $10 each.  Before you know it, the $5 a month budget has grown to $50, $60, or $70.  We'll tell the story of how we can took a bare bones FreeBSD VM from a major cloud hosting company and built it up to be a standalone bastion host that works as a nameserver and more.  

# Our Goals
For this project, we wanted to buy a domain name, and then do everything ourselves.  Aside from needing the registrar to get the name, we didn't want to have to use any of the provided services.  Registrars can do a great job of offering web hosting, site building, and other services; but, we shouldn't have to depend upon them.  If we took the lowest cost deal they offered for hosting, what could we do?

For a projected cost of $10 a month, we could get two VMs.  These nodes would each have their own public IP address.  They'd be bare bones FreeBSD systems.  Being "out there on the Internet" would be to our advantage; sites acting as nameservers would serve us better away from our main node.  Since our main demarc only gets one IP, these two VMs might be a good fit.

# Domain Name Purchasing
We bought a pair of names that we like.  We didn't buy any extras.  We didn't get hosting, certificates, email:  none of that.  Just get a name.

# Before We Build Anything
With an account created at the hosting provider, we could quickly see that they offered us just enough power to get ourselves in trouble.  First off, they wanted us to set either a root password or upload an SSH key.  Choose the key.  Building an SSH key is a little more involved; but, we don't want to arrive at the node, in the root account via SSH, with a simplistic password.

That's right:  the hosting provider, by default, chose to provide SSH into root, first thing.  Now, after a while, we'd all do what we could to plus up the node and improve things.  However, the Internet is the Wild West.  Already out there will be scanning nodes looking for new accounts like ours.  It'd be best to arrive tough enough to repel common attacks.  For this reason, we recommend setting up an SSH key if you have the choice.

Since we chose the SSH key, we were not able to use the provided web console to look into our host.  You guessed it:  if you pick a public key, then you have to use it.  So, we did.  

REF:

# Establishing a BIND DNS Server with MX to Encrypted EMail Provider
We quickly established a BIND9 DNS server, using this tutorial:

Our domain name provider had a set of dialogs that allowed us, through our account, to direct our domain name to our own nameserver.  Armed with the IPs from the VMs and some progress through the DNS server tutorial, we were able to adapt and fill out the records wtih the registrar.

We stopped right before the "DNSSEC" portion of the tutorial and added our MX records.  For our email server, we chose a free online email provider that offers storage in Switzerland.  As part of our subscription with them, we were able to use custom domains.  To get that to work, we had to follow a set of dialogs the email provider had for setting up critical DNS records for SPF, DKIM, and DMARC.  It took a little bit for our changes to propagate, but it worked with little fuss.

Once our nameserver was found and our emails were arriving at the desired account, we walked through DNSSEC.  This was one advantage we had over what the registrar offered:  they don't do DNSSEC setups.  With the tutorial, we got it working in less than an hour.  By carefully following the directions, we were able to get DNSSEC working right away.  It was actually one of the easiest aspects of the system deployment.

# Dynamic DNS and Our Node
With two nodes out on the Internet, and one nearby, there was always the chance that a service provider might change our address.  In practice, we were able to get stable addresses just by asking our local muni fiber company to provide one.  Still, they could change at any time.  So, we used an existing subscription to a dynamic DNS provider to address the hosts.  

To use a dynamic DNS with FreeBSD, we install ddclient.  Our service provider gave us example configuration files that worked on the first try.  After a while, we began to realize that, if our nameserver was working well, then there was no reason why we couldn't run a dynamic DNS service of our own.  ddclient will work with RFC compliant procedures.  We found tutorials to support that innovation on FreeBSD, too.

It was just easier to have some addresses with simple names, through dynamic DNS.  Time and again, during configurations like these, we would need to call up the computer.  From one to the other and back again, we'd type in calls.  It can be helpful and convenient during the confusing moments of configuration to be able to call in to your desired host easily.  

In the case of whitelisting hosts in firewall rules, it can be helpful to also list some hosts using their dynamic dns domain name.  If the IPs change, and firewall rules hold hardcoded IP addresses, then how will you get back in to the host you've just secured?  Dynamic DNS addresses in the firewall rules offer a saftey during the confusing moments of configuration.

# Firewall Rules with pf
Pf comes installed as part of the base operating system in FreeBSD.  There are also two others we could choose from.  It's mostly a matter of preference.  We picked pf.  

To set up the rules, we consulted some tutorials.  Early on, we wrote pass in rules for our favorite IPs and those dynamic DNS host and domain names, as mentioned above.  

We soon found out we were locking out critical services.  By using searches of /etc/service for keywords, we often found ports that needed to be kept open to keep our machine running smoothely.  Since we were drafting some of these rules for the first time, this took some expriementation.  

# External Monitoring with Shodan
Shodan.io offers a monitoring service.  It's free with the developer's account, up to about a dozen hosts.  Around 14 or 15, they start to require a paid Enterprise account.  Since we have few computers to look after, we snapped up Shodan Monitor.  

With Shodan, we learned the painful history of the past users of our IP address.  

# Proxy:  Providing Standoff Distance and a Planned Access Approach
With the DNS server up and running, we had a lot of space left over. The minimal, bare-bones host provided by the cloud service had used barely 10% of its capacity.  We wondered what else we could do with the machine.  We could screen.

With a firewall already running (and repelling unwanted calls) on the master and slave DNS servers, wouldn't it be nice to use them as standoff entrances for our site?  With a demarc to safeguard, it would be best if we could require traffic to enter at the distant hosts and then walk down to our site along a path we could specify.  This seemed to be the best way to build in "standoff distance" into the acces plan.  

With standoff distance, we have a link between our data center demarc and the general public that we can monitor, secure, and modify.  What if we have a DoS attack?  If our front door is our only door to the Internet, we won't have any room to move or leverage to create a response.  Again, having a VM on the Internet in a distant place is to our advantage.  

Now, as traffic arrives at our data center demarc, it'll hit a firewall right away.  Later, as it moves down the line to our desired server, it'll hit some auth.  So, inside our data center we could react to traffic; but, the proxies on the distant hosts will help us control traffic that arrives to the server that's our goal.  At the data center demarc, we can have a firewall rule that will refuse traffic for our desired server unless it walks down the path from our proxy.

# Installing HAProxy
Installing the program was a snap.  We had to read some documentation, and follow some suggestions; but, it all worked without a hitch.  There are several techniques we can choose from with HAProxy.  Ultimately, we chose 







{% highlight shell %}
{% endhighlight %}
 \[[]\]
## Annotated Bibliography


\[1\] _____.  INTERNET:   [`https://forum.dd-wrt.com/phpBB2/viewtopic.php?t=51486`](https://forum.dd-wrt.com/phpBB2/viewtopic.php?t=51486)