---
title:  "Debricking WRT54GS Routers with Bus Pirate v.3.6 and JTAG"
date:   2019-12-15 08:30:00
description: Debricking WRT54GS Routers with Bus Pirate v.3.6 and JTAG
---

Some time ago, I was able to purchase an $8 router on eBay that was bricked during a DD-WRT install.  I knew it was bricked, and I believed I could revive it.  This blog post is about that journey.  

The router was probably bricked due to a problem with the model.  Most of the WRT54G routers, hereafter known affectionately as "WARTHOG", are able to undergo a "hard reset" process DD-WRT hackers know as 30-30-30.  This process involves holding down the reset button for 30 seconds with power, 30 seconds without power, and 30 seconds with power again.  It's a procedure that will reset most of the SOHO routers of this type. 

Unfortunately, a small number of the models can't survive this process.  This warthog is one of them.  My guess is that the hacker I bought it off of probably tried to put DD-WRT on it and ran into a problem because of this idiosyncrasy.  

We checked it out, and as expected, it's bricked.  To attempt to revive this router, we'll attempt to reflash the firmware through JTAG.  To carry this out, we'll use a Windows 10 home computer, a Bus Pirate serial interface device, and the victimized router.  If everything works, we'll be able to load DD-WRT on there, and the router will live again.

# Fixing up the Bus Pirate
Before we can get started, we need to adjust the firmware on our Bus Pirate.  This little handy device is good for a lot of things; however, the firmware loaded on this particular one can't do JTAG.  To dress that up, we'll need to reflash the firmware onto the Bus Pirate.  

We could be afraid.  We could live in fear of zapping this little device.  But, when we think back to the great nothing we're accomplishing already, we can rest assured that whatever nomimal risk we face in this endeavor will not impede our already negligible productivity.  Let's get on with flashing some firmware on Bus Pirate.





{% highlight shell %}
{% endhighlight %}
 \[[]\]
## Annotated Bibliography
\[1\]  


\[\] _____.  INTERNET:   [``]()

\[\] _____.  INTERNET:   [``]()

\[\] _____.  INTERNET:   [``]()

\[\] _____.  INTERNET:   [``]()

\[\] _____.  INTERNET:   [``]()

\[\] _____.  INTERNET:   [``]()

\[\] _____.  INTERNET:   [``]()

\[\] _____.  INTERNET:   [``]()

\[\] _____.  INTERNET:   [``]()

\[\] _____.  INTERNET:   [``]()

\[\] _____.  INTERNET:   [``]()

[//]: # (Hyperlinks)
[1]: https://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/
