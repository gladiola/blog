---
title:  "Debricking WRT54GS Routers with Bus Pirate v.3.6 and JTAG"
date:   2019-12-15 08:30:00
description: Debricking WRT54GS Routers with Bus Pirate v.3.6 and JTAG
---

Some time ago, I was able to purchase an $8 router on eBay that was bricked during a DD-WRT install.  I knew it was bricked, and I believed I could revive it.  This blog post is about that journey.  

# Trouble with This Warthog
The router was probably bricked due to a problem with the model.  Most of the WRT54G routers, hereafter known affectionately as "Warthog", are able to undergo a "hard reset" process DD-WRT hackers know as 30-30-30.  This process involves holding down the reset button for 30 seconds with power, 30 seconds without power, and 30 seconds with power again.  It's a procedure that will reset most of the SOHO routers of this type. 

Unfortunately, a small number of the models can't survive this process.  This warthog is one of them.  My guess is that the hacker I bought it off of probably tried to put DD-WRT on it and ran into a problem because of this idiosyncrasy.  

We checked it out, and as expected, it's bricked.  To attempt to revive this router, we'll attempt to reflash the firmware through JTAG.  To carry this out, we'll use a Windows 10 home computer, a Bus Pirate serial interface device, and the victimized router.  If everything works, we'll be able to load DD-WRT on there, and the router will live again.

# Background with DD-WRT
Before working on this project, we'd already had some experience with using DD-WRT.  It's a great piece of firmware.  It's as good, in some cases much better, than firmware I've seen on other SOHO routers.  I know I have purchased routers for hundreds of dollars whose firmware UI and performance were not this good.  However, there are some quirks.

The number one quirk with using DD-WRT is a rushing, inexperienced operator who did not read the documentation.  And, that documentation is largely horrible.  To get through it, you have to follow conversations in old forum threads many times.  You can expect to see ancient, deprecated references along the way.  You'll have to sort through what's applicable to your router.  Most of all, it pays to read ahead.  

Firmware is fussy.  The small computers accepting these programs don't have the huge built-in layers of programming so many of us are used to these days.  When flashing firmware, we're putting it on the chip.  When we're writing normal programs for applications, we're layers above where we're going now.  By getting close to the bits, we're also going beyond the protections that would be normally provided.  Notice how we're flashing a bricked router.  It probably got bricked for these very reasons:  one false move for this equipment that would have worked just fine on many others.  

What can we do about this situation?  Get some experience with DD-WRT.  Read the documentation.  Be sure about the model number you're working on.  Don't overlook questions that arise.  Take care with the technical details.  They count here.  Really, the technical details are more than half the ball game.  

Documents that we need to have read and be familar with include:
- The "Peacock" Thread
- Model-specific How To Flash pages from the DD-WRT wiki
- Wiki Installation page on "Choosing the Correct Firmware"
- And any forum threads about DD-WRT that come up in model research.

Watch a few videos of people putting DD-WRT on their routers.  Give a scrap router a try.  Don't use your only SOHO router and computer for this experiment.  Build up and gain.  We knew when we were ready to step up to trying to debrick a router that had problems with undergoing these flashing procedures.  

# Existing Features on Bus Pirate
The Bus Pirate is a small computer that converts signal to serial output of different kinds.  At its heart is a PIC24 microcontroller.  There's a console interface built in.  We can connect the Bus Pirate to a computer with a USB cable.  Software can help us connect to the device.  Since we're hooked up to a Windows 10 computer, we will get in to Bus Pirate with PuTTY.  

We can determine which COM port the USB device is assigned by using Device Manager.  We'll use COM7, 115200, 8/1/n settings in PuTTY.  Once connected, we'll hit ENTER and a HiZ prompt will be displayed.  From there, we can explore the menu and some of the options by following along with some tutorials in the documentation.  

The script menu can even run some good ole fashioned BASIC, just like in 7th grade.  However, if we get stuck in an infinte loop, we'll have to power down the Pirate.  There isn't an interface sophisiticated enough to accept and respond to common keyboard interrupts.  Any program that we provided would either have to go without those or accept ones we build in.  For the most part, the BASIC language interpreter is provided for the sake of automating repetitive tasks when working with the Pirate.  

To make sure our hardware is working alright, we can run the self-test.  A simple tilde "~" and RETURN will initiate the test.  To get a successful test, we'll use the jumper wires on the Pirate's pin block to run some simple connections.  The menu prompt tells us which pins to connect.  It's just two jumps.

# Fixing up the Bus Pirate to Carry Out JTAG Messaging
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
