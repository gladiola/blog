---
title:  "$5 Cloud Node to Bastion Host"
date:   2020-03-20 08:30:00
description: Hardening a cloud node to withstand normal Internet traffic.
---

Cloud providers hook us in with cheap teaser rates; and then they slowly offer us feature after feature, at $10 each.  Before you know it, the $5 a month budget has grown to $50, $60, or $70.  We'll look at how we can take a bare bones FreeBSD VM from a major cloud hosting company and build it up to be a standalone bastion host that works as a nameserver.  

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







{% highlight shell %}
{% endhighlight %}
 \[[]\]
## Annotated Bibliography


\[1\] _____.  INTERNET:   [`https://forum.dd-wrt.com/phpBB2/viewtopic.php?t=51486`](https://forum.dd-wrt.com/phpBB2/viewtopic.php?t=51486)