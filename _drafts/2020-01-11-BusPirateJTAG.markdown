---
title:  "Debricking WRT54GS Routers with Bus Pirate v.3.6 and JTAG"
date:   2019-12-15 08:30:00
description: Debricking WRT54GS Routers with Bus Pirate v.3.6 and JTAG
---

Some time ago, we were able to purchase an $8 router on eBay that was bricked during a DD-WRT install.  We knew it was bricked, and we believed we could revive it.  This blog post is about that journey.  

# Trouble with This Warthog
The router was probably bricked due to a problem with the model.  Most of the WRT54G routers, hereafter known affectionately as "Warthog", are able to undergo a "hard reset" process DD-WRT hackers known as 30-30-30. \[[5]\] This process involves holding down the reset button for 30 seconds with power, 30 seconds without power, and 30 seconds with power again.  It's a procedure that will reset most of the SOHO routers of this type. 

Unfortunately, a small number of the models can't survive this process.  This warthog is one of them.  My guess is that the hacker I bought it off of probably tried to put DD-WRT on it and ran into a problem because of this idiosyncrasy.  

We checked it out, and as expected, it's bricked. \[[4]\] To attempt to revive this router, we'll attempt to reflash the firmware through JTAG.  To carry this out, we'll use a Windows 10 home computer, a Bus Pirate serial interface device, and the victimized router.  If everything works, we'll be able to load DD-WRT on there, and the router will live again.

# Background with DD-WRT
Before working on this project, we'd already had some experience with using DD-WRT.  It's a great piece of firmware.  It's as good, in some cases much better, than firmware I've seen on other SOHO routers.  I know I have purchased routers for hundreds of dollars whose firmware UI and performance were not this good.  However, there are some quirks.

The number one quirk and source of faults with using DD-WRT is a rushing, inexperienced operator who did not read the documentation.  And that documentation is largely horrible.  To get through it, you have to follow conversations in old forum threads many times.  You can expect to see ancient, deprecated references along the way.  You'll have to sort through what's applicable to your router.  The wiki is a rat's nest of interconnected postings.  There will be times when the reader can't be sure if the information is even applicable; sometimes the directions can't be readily imagined.  It's tough for a newbie to get through.  There will be mistakes, but, it pays to read ahead.  

Firmware is fussy.  The small computers accepting these programs don't have the huge built-in layers of programming so many of us are used to these days.  When flashing firmware, we're putting it on the chip.  Other times, When we're writing normal programs for applications, we're layers above where we're going with firmware flashes.  By getting close to the bits, we're also going beyond the protections that would be normally provided.  Notice how we're flashing a bricked router.  It probably got bricked for these very reasons:  one false move for this equipment that would have worked just fine on many others.  

What can we do about this situation?  Get some experience with DD-WRT.  Read the documentation.  Be sure about the model number you're working on.  Don't overlook questions that arise.  Take care with the technical details.  They count here.  Really, the technical details are more than half the ball game.  

Documents that we need to have read and be familar with include:
- The "Peacock" Thread \[[1]\]
- Model-specific How To Flash pages from the DD-WRT wiki \[[3]\]
- Wiki Installation page on "Choosing the Correct Firmware" \[[2]\]
- And any forum threads about DD-WRT that come up in model research.

Watch a few videos of people putting DD-WRT on their routers.  Give a scrap router a try.  Don't use your only SOHO router and computer for this experiment.  Build up and gain.  We knew when we were ready to step up to trying to debrick a router that had problems with undergoing these flashing procedures.  

# Existing Features on Bus Pirate
The Bus Pirate is a small computer that converts signal to serial output of different kinds.  At its heart is a PIC24 microcontroller.  There's a console interface built in.  We can connect the Bus Pirate to a computer with a USB cable.  Software can help us connect to the device.  Since we're hooked up to a Windows 10 computer, we will get in to Bus Pirate with PuTTY.  

We can determine which COM port the USB device is assigned by using Device Manager.  We'll use COM7, 115200, 8/1/n settings in PuTTY.  Once connected, we'll hit ENTER and a HiZ prompt will be displayed.  From there, we can explore the menu and some of the options by following along with some tutorials in the documentation.  \[[9]\]\[[10]\]\[[11]\]\[[12]\]\[[13]\]

The script menu can even run some good ole fashioned BASIC, just like in 7th grade.  However, if we get stuck in an infinte loop, we'll have to power down the Pirate.  There isn't an interface sophisiticated enough to accept and respond to common keyboard interrupts.  Any program that we provided would either have to go without those or accept ones we build in.  For the most part, the BASIC language interpreter is provided for the sake of automating repetitive tasks when working with the Pirate.  \[[8]\]

To make sure our hardware is working alright, we can run the self-test.  A simple tilde "~" and RETURN will initiate the test.  To get a successful test, we'll use the jumper wires on the Pirate's pin block to run some simple connections.  The menu prompt tells us which pins to connect.  It's just two jumps.\[[7]\]

# Fixing up the Bus Pirate to Carry Out JTAG Messaging
Before we can get started, we need to adjust the firmware on our Bus Pirate.  This little handy device is good for a lot of things; however, the firmware loaded on this particular one can't do JTAG.  To dress that up, we'll need to reflash the firmware onto the Bus Pirate.  \[[14]\]\[[6]\]

We could be afraid.  We could live in fear of zapping this little device.  But, when we think back to the great nothing we're accomplishing already, we can rest assured that whatever nomimal risk we face in this endeavor will not impede our already negligible productivity.  Let's get on with flashing some firmware on Bus Pirate.

We had to download two programs.  One, MPLABX IDE \[[17]\], is a NetBeans derivative (NetBeans can be used as a framework for building other programs like it) that compiles and runs the code.  The second was a compiler that prepares code for the Bus Pirate. \[[16]\]  We downloaded the code from the GitHub repository.  \[[14]\] 

There were two small hiccups.  Both were covered by the documentation on GitHub.  We had to use a jumper wire on the BusPirate (for PGC and PGD) in order to put it in the right mode to accept the firmware flash.  Second, the hex file produced by the IDE needed to have its letters replaced with capitals so that the checksum would work out. \[19]\] With some careful command crafting, the program ran.  It ended in an error message; but, as with the others, the documentation pointed out that this was actually a case of successful completion.  

{% highlight shell %}
pirate-loader.exe  --dev=COM7 --hex=../../Firmware/busPirate.X/dist/BusPirate_v3/production/busPirate.X.production.hex
{% endhighlight %}

With Bus Pirate v.3.6 hardware flashed with the latest firmware, we still did not see a JTAG option in the menu.  So, we will still need to experiment to see if it can be used well.  Turning back to the resources from DD-WRT, we'll probably try to use a Bus Pirate program that's in the collection for "HairyDairyMaid's" code. \[[15]\]


{% highlight shell %}
{% endhighlight %}
 \[[]\]
## Annotated Bibliography


\[1\] _____.  INTERNET:   [`https://forum.dd-wrt.com/phpBB2/viewtopic.php?t=51486`](https://forum.dd-wrt.com/phpBB2/viewtopic.php?t=51486)

The DD-WRT "Peacock" thread.

\[2\] _____.  INTERNET:   [`https://wiki.dd-wrt.com/wiki/index.php/Installation`](https://wiki.dd-wrt.com/wiki/index.php/Installation)

\[3\] _____.  INTERNET:   [`https://wiki.dd-wrt.com/wiki/index.php/Hardware-specific`](https://wiki.dd-wrt.com/wiki/index.php/Hardware-specific)

A page of links to instructions for specific models of common SOHO routers.

\[4\] Using the Peacock Thread, point 6, "Is your router bricked," we were able to test the device and confirm that it was bricked.  Unresponsive to normal operation, failing these tests, and described by the previous owner as bricked:  this router was bricked.   

\[5\] _____.  INTERNET:   [`https://wiki.dd-wrt.com/wiki/index.php/Hard_reset_or_30/30/30`](https://wiki.dd-wrt.com/wiki/index.php/Hard_reset_or_30/30/30)

\[6\] _____.  INTERNET:   [`https://github.com/BusPirate/Bus_Pirate`](https://github.com/BusPirate/Bus_Pirate)

Bus Pirate GitHub page.

\[7\] _____.  INTERNET:   [`http://dangerousprototypes.com/docs/Self-test_guide`](http://dangerousprototypes.com/docs/Self-test_guide)

Bus Pirate self-test.

\[8\] _____.  INTERNET:   [`http://dangerousprototypes.com/docs/Bus_Pirate_BASIC_script_reference`](http://dangerousprototypes.com/docs/Bus_Pirate_BASIC_script_reference)

BASIC on Bus Pirate.

\[9\] _____.  INTERNET:   [`http://dangerousprototypes.com/docs/Bus_Pirate`](http://dangerousprototypes.com/docs/Bus_Pirate)

\[10\] _____.  INTERNET:   [`http://dangerousprototypes.com/docs/Bus_Pirate_user_interface`](http://dangerousprototypes.com/docs/Bus_Pirate_user_interface)

\[11\] _____.  INTERNET:   [`http://dangerousprototypes.com/docs/Bus_Pirate_menu_options_guide`](http://dangerousprototypes.com/docs/Bus_Pirate_menu_options_guide)

\[12\] _____.  INTERNET:   [`http://dangerousprototypes.com/docs/Bus_Pirate_101_tutorial`](http://dangerousprototypes.com/docs/Bus_Pirate_101_tutorial)

\[13\] _____.  INTERNET:   [`http://dangerousprototypes.com/docs/Bus_Pirate_102_tutorial`](http://dangerousprototypes.com/docs/Bus_Pirate_102_tutorial)

\[14\] _____.  INTERNET:   [`https://github.com/BusPirate/Bus_Pirate/blob/master/Documentation/building-and-flashing-firmware.md`](https://github.com/BusPirate/Bus_Pirate/blob/master/Documentation/building-and-flashing-firmware.md)

Bus Pirate general directions on how to flash the circuit board.

\[15\] _____.  INTERNET:   [`https://github.com/notch/bpjtag`](https://github.com/notch/bpjtag)

Hairy Dairy Maid's Debricker ported to use Bus Pirate and JTAG.

\[16\] _____.  INTERNET:   [`https://www.microchip.com/mplab/compilers`](https://www.microchip.com/mplab/compilers)

Required compiler for the PIC24.

\[17\] _____.  INTERNET:   [`https://www.microchip.com/mplab/mplab-x-ide`](https://www.microchip.com/mplab/mplab-x-ide)

MPLAB IDE used to run the build of the firmware projects.

\[18\] _____.  INTERNET:   [`https://github.com/BusPirate/Bus_Pirate/issues`](https://github.com/BusPirate/Bus_Pirate/issues)

Bus Pirate issues page from GitHub.

\[19\] _____.  INTERNET:   [`https://github.com/BusPirate/Bus_Pirate/issues/135`](https://github.com/BusPirate/Bus_Pirate/issues/135)

Small issue with a need to capitalize some letters in a hex file.

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
[1]: https://forum.dd-wrt.com/phpBB2/viewtopic.php?t=51486
[2]: https://wiki.dd-wrt.com/wiki/index.php/Installation
[3]: https://wiki.dd-wrt.com/wiki/index.php/Hardware-specific
[4]: ``
[5]: https://wiki.dd-wrt.com/wiki/index.php/Hard_reset_or_30/30/30
[6]: https://github.com/BusPirate/Bus_Pirate
[7]: http://dangerousprototypes.com/docs/Self-test_guide
[8]: http://dangerousprototypes.com/docs/Bus_Pirate_BASIC_script_reference
[9]: http://dangerousprototypes.com/docs/Bus_Pirate
[10]: http://dangerousprototypes.com/docs/Bus_Pirate_user_interface
[11]: http://dangerousprototypes.com/docs/Bus_Pirate_menu_options_guide
[12]: http://dangerousprototypes.com/docs/Bus_Pirate_101_tutorial
[13]: http://dangerousprototypes.com/docs/Bus_Pirate_102_tutorial
[14]: https://github.com/BusPirate/Bus_Pirate/blob/master/Documentation/building-and-flashing-firmware.md
[15]: https://github.com/notch/bpjtag
[16]: https://www.microchip.com/mplab/compilers
[17]: https://www.microchip.com/mplab/mplab-x-ide
[18]: https://github.com/BusPirate/Bus_Pirate/issues
[19]: https://github.com/BusPirate/Bus_Pirate/issues/135
[20]: 
