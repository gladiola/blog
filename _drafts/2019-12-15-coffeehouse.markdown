---
title:  "FreeBSD Jailhost for VMs"
date:   2019-12-15 08:30:00
description: FreeBSD Jails with VNET Subnets for Virtual Machines
---

In this tutorial, we'll walkthrough the process of setting up a host for hacking targets.  Our host will be based on FreeBSD; we'll use jails to control the individual target accounts; we'll use virtual machines as the targets inside those accounts.  Our goal will be to turn a modest Dell i5 salvaged desktop into a machine that we can hack and reset at will.  

As part of building our target host, we'll want to include some isolating and stabilizing features.  We'll want each target account to be contained in a resettable jail; this way, if there are severe problems with the VM because of the hacking attacks, then we can reset the target by restarting the jail.  Each target will need its own VNET so that we can focus over-the-network attacks on the desired targets without accidentally spilling over into other areas.  We'll want each VM hypervisor to be able to start up about half a dozen virtual machines.  This way we could host them all, or change out targets for variety.  

We'll build up our jails, one by one, step by step.  We'll controll them manually.  We'll operate each virtual machine in a headless fashion using a simple script.  Along the way, we'll cover some of the major topics in virtualization for hosts and networks.  

We couldn't build this project without the advice we found through some important publications.  No hacker's work is their own, and this tutorial is no exception.  For learning about FreeBSD jails, we turned to Michael Lucas' books.  For learning about the headless operation of the VMs, we used Andrea Fortuna's blog posts about Virtualbox.  As general background material on the GreeBSD operating system, we have turned many, many times to the FreeBSD Handbook.  Our desire for hacking targets like these has been fueled by some great labs experienced to be found with Offensive Security's PWK:  Penetration Testing with Kali Linux course.  Citations for these resources and more can be found in the annotated bibliography section.    

Let's get started.

# Preliminary Planning
## Hardware
Our target machine will be built on a salvaged Dell Optiplex 990.  Purchased at a government auction for about $50, this salvaged machine has been cleaned, tested, and fitted with some small SSDs.  For storage, we'll be using two refurbished Kingston 120GB SSDs that sold for about $12/each.  We've outfitted the box with 8G of 10600U RAM, a dual port network card, and a graphics port that does Display and DVI formats.  The power supply has been tested; there's a DVD drive we'll use for installing the base system ISO.

For our tutorial, we'll rig up our main internet connection to port 0 on the network card.  We'll have to hook up our monitor to the graphics card to meet the BIOS requirements.  We've set the BIOS boot order to let us boot from CD/DVD disk, USB drive, and then the SSD.  

## Phases of Install
Based on a previous prototyping run, we're going to break up our project into some phases so that we can gradually build up our jailhost into a target machine.

- Host base install
- Host ZFS drive partitioning 
- Basic jail install
- VNET setup
- Jail template for VMs
- Jail replication and provisioning

A lot of the decisions we're about to make will be based on our intentions for the system.  For example, when we get to VNET, we'll see that we've already laid out a plan for subnetting and explicitly assigning many IP addresses.  Likewise, we have a corresponding plan for setting up multiple ZFS features.  Combined, the VNET and ZFS will give each VM its own contained communications and storage.  

![Asset plan for SALVAGE13B-COFFEEHOUSE]({{ site.url }}/assets/images/SALVAGE13B-COFFEEHOUSE/assetPlan.png)

Making these decisions can be a lot easier when we're designing a system with a clear task and purpose.  Oftentimes, when we build systems on our own we may not be too sure what we'll do with that machine in the future.  If we were to stumble and flouder our way to concluding this is the type of system we wanted, then we could have a very messy and problematic install.  We're sure to run into problems, but having a clear idea is a great help to us.  We can set goals that will let us build out the box we want.

## Building Our VNET Plan
We will want to have four main jails on the VNET.  Each will carry its own subnet.  Inside those jails, we'll want 6 hosts per net.  After some experimenting with an online CIDR subnet calculator \[[5]\], we've decided on a plan based on 192.168.1.192/27/29.  Robert Eisele's subnet calculator can help us lay out every IP in that net and confirm the relationship of each IP to the network.  What subnet masks we need, which are the first and last hosts in each part of the net, and what addresses are used for the nets and their broadcasts:  those topics are all instantly described with that online tool.  

![Subnet plan for SALVAGE13B-COFFEEHOUSE]({{ site.url }}/assets/images/SALVAGE13B-COFFEEHOUSE/subnetPlan.png)

## Choosing an OS Release for the Host and the Jails
With jails, we must run the jailhost with the youngest version of the operating system.  We can run jails with older versions of the OS, but they can never be younger than what's running on the host.  With this rule in mind, we'll run 12.1 on the jailhost.  We simply go to the website at FreeBSD.org, download 12.1, and burn it to DVD disc using another working computer connected to the Internet.  

Based on past experiments, I found it was helpful to choose the "DVD" version of the FreeBSD installer.  While others may work, the DVD version holds the most data on the installer disk.  When troubleshooting and hunting around for resources, I felt it was most comfortable and familiar.  I think that it's possible that USB memstick and memstick-mini installer versions will work just as well; however, to cut down on the nagging feeling I might be missing something critical during troubleshooting, we chose our most familiar installer:  the dvd.  \[[6]\] \[[7]\] \[[8]\] \[[9]\]

# Host Base Install
## Installing FreeBSD Onto the Hardware
With our disk of a FreeBSD ISO on hand, we'll drop it into the target machine and let the installer take the lead.  We'll make choices that allow for the download of base, src, and ports files.  We'll allow the installer to make the usual choices for our machine  \[[10]\].  We'll choose a ZFS encrypted install, to accept every secure option in the installer dialog for hardening, and to download a copy of the FreeBSD handbook.  We'll name our machine SALVAGE13B-COFFEEHOUSE.  

All of these are mostly personal choices that won't have a bearing on the features we'll add on later.  However, we're going to make careful note of account names and passphrases that are decided during our install.  We'll make a root account and a user account called "barista".  We'll add barista to our wheel and operator groups.  This will give us an alternate to root that we can use with ssh later.  With the ZFS GELI encryption for the disk, we'll emerge from this phase with three critical passwords recorded.  

## Initial checks
With the system installed, one of the first things we'll do is check those passphrases.  Make sure each account works.  Any problems with those could complicate or confound anything we do after this.

We'll do a quick update of the FreeBSD installation and ports  \[[11]\] \[[12]\].
- freebsd-update fetch
- freebsd-update install
- portsnap fetch
- portsnap extract

We'll install nano, one of my favorite text editors.

{% highlight shell %}
cd /usr/ports/editors/nano
make -DBATCH install clean
{% endhighlight %}

For the sake of merciful administration, we'll also install bash.

{% highlight shell %}
cd /usr/ports/shells/bashcd
make -DBATCH install clean
{% endhighlight %}


The `-DBATCH` helps us avoid all of the pauses in the ncurses dialogs for configuring the downloads and compilations.  But, if there is difficulty, we'll have to delve into `make config`, see what options are presented; maybe we'll also have to hop into the ports that threw the errors, and then recompile those individually.  Sometimes we might have to do a `pkg install <PORTNAME>` to substitute the compiliation of a program with a strait binary install.  With those troubleshooting tasks under our belt, we'll continue with our project.

To check our current ZFS configuration:
{% highlight shell %}
zfs list
zpool list
{% endhighlight %}

To take a snapshot some datasets at this stage of installation:
{% highlight shell %}
zfs snapshot zroot/ROOT/default@<SOME_DATE_AND_TIME>
zfs snapshot zroot/usr@<SOME_DATE_AND_TIME>
{% endhighlight %}
\[[13]\]

# Host ZFS Drive Provisioning
## Carve Up Some ZFS Datasets and Directories
To later accomodate our jails, we'll want to create some ZFS datasets.  Combined with ZFS snapshot procedures, we'll then be able to make snapshots of our progress (or regress) when constructing the jails.  Since there's nothing really on the disk right now, we can declare the datasets with no worry.  We won't need to specify how much space each one takes up.  We just need to describe how we'll carve up the data.

To lay the foundation, we'll give:
{% highlight shell %}
zfs create -o canmount=off zroot/jail
{% endhighlight %}

To lay out the main divisions of the jail directories, we'll give:
{% highlight shell %}
zfs create zroot/jail/coffeehouse
zfs create zroot/jail/redeye
zfs create zroot/jail/cappucino
zfs create zroot/jail/latte
{% endhighlight %}

Within each of those, we'll lay out six subdirectories, for the VMs we'll install later.  Typically, we'll give:
{% highlight shell %}
zfs create zroot/jail/coffeehouse/vm1
zfs create zroot/jail/coffeehouse/vm2
{% endhighlight %}
... and so on, until we have given six subdirectories for each of the four jail directories.  

To see our progress, we can:
`zfs list` and it will show us all of the ZFS datasets.

We'll want to associate each of those datasets with a group, for user access control.  We can create groups for each of the four jail directories by using commands like:
{% highlight shell %}
pw groupadd coffeehouse
pw groupmod coffeehouse -m barista
{% endhighlight %}
... and so on, until we have given our barista account access to all four of the groups.  We can assign those groups as owners of the datasets with commands like:
{% highlight shell %}
chown -R barista:coffeehouse /zroot/jail/coffeehouse
{% endhighlight %}
We can see our system is unified with our barista account owning the four directories with each under its own group.  \[[14]\]\[[15]\]

### Limit Jail Size
We can limit the growth of ZFS directories by setting a quota property.  A simple script can help us limit our directories to a specified maxmimum size. \[[20]\]

## ZFS Snapshots, File Copies, and ZFS Rollbacks
The ZFS snapshot command is a little different from some other "snapshots."  The word snapshot implies that we are seeing the directory as it was at that moment.  That's true; but the ZFS snapshot command will begin tracking differences to the directory specified from the moment we command the snapshot to begin.  The ZFS snapshot is like the target moment for a future restoration.

We can apply the results in a couple ways.  We can send the snapshot to another directory, and then copy out what we need; or, we can use the `zfs rollback` command to undo all the changes to a directory since a given snapshot.  So, when we say we are taking a snapshot of a directory, we are really beginning the recording of differences to that directory from that moment on.  

## Prove Dataset and File Directory Association
To prove that a given ZFS dataset is associated with a directory, we'll conduct a simple test.  We'll write a text file to one of the directories owned by barista; we'll make a snapshot; we'll delete the file; then we'll restore the directory to its previous state using the snapshot.  We should be able to see the ZFS snapshot system working for us.

To test out the snapshot, we write a file to a known directory.
{% highlight shell %}
cd /zroot/jail/coffeehouse
nano test.txt
{% endhighlight %}
In that file file, we write some simple words so that it will have some content.  We take a snapshot of the directory.
{% highlight shell %}
zfs snapshot zroot/jail/coffeehouse@2019-12-05A
zfs list -t snapshot
{% endhighlight %}
Then we make a change to the directory.  We can delete that test.txt file.  The snapshot will show the change by showing an increase in Kb used.  
{% highlight shell %}
rm /zroot/jail/coffeehouse/test.txt
zfs list -t snapshot
{% endhighlight %}
### Restorations with File Copies
To practice restoring just that one file of the snapshot, we can send the snapshot to a temporary directory, list the contents, and copy it out to the desired destination.
{% highlight shell %}
mkdir /tmp/snapShot
cp -r /zroot/jail/coffeehouse/.zfs/snapshot/2019-12-05A /tmp/snapShot
ls /tmp/snapShot
cp /tmp/snapShot/test.txt /zroot/jail/coffeehouse/test.txt
{% endhighlight %}

### ZFS Rollbacks
Or we could try the rollback command:
{% highlight shell %}
zfs rollback zroot/jail/coffeehouse@2019-12-05A
{% endhighlight %}
Notice that when we are using the `zfs` commands, we have to leave off the `/` when referring to zroot.  

Once we're satisfied with the rehearsals of the rollbacks, we can remove the test file and delete the snapshot we made.
{% highlight shell %}
zfs destroy zroot/jail/coffeehouse@2019-12-05A
{% endhighlight %}
### Setup Snapshots Before Jail Installations
Our directories for the jails basically don't have anything in them besides the anticipated directory structure.  We'll make snapshots of each jail directory and each vm subdirectory to show our starting point.  We'll use commands like:
{% highlight shell %}
zfs snapshot zroot/jail/coffeehouse@2019-05-05-EMPTY
{% endhighlight %}
Until we've made all the snapshots we'd like.

 \[17\]\[18\]

## Record Basic Facts About Config Files
Since we're about to build some jails and adjust the configuration of the jailhost, we're likely to make some mistakes soon.  We can use some of these same techniques to gather files we're likely to edit and copy them to a directory for archiving.

We'll be interested in editing:
- /boot/boot.conf
- /etc/devfs.conf
- /etc/rc.conf
- /etc/jail.conf

We'll also make some copies of outputs of common jail and zfs commands.

# Prototyping Lessons Learned for What's Ahead
## Understanding How We Can Get to Our Goals
In order to set up the jails, we had to experiment a little.  Understanding from the FreeBSD Handbook, Lucas' books, and several blogs was an essential part of forecasting how we might get to where we'll go.  \[[21]\]\[[22]\]\[2\]\[3\]\[[24]\]\[[25]\]  These experiments and references led to a few small discoveries for us.  I'd like to share these now.  We will see their influence later.  Some of these ideas came up as we were stitching together parts from various references to make our prototype work.  

### Where Will The Base System Come From?
When we read over several sets of directions about setting up jails, it was very common to run across a mention like, "just grab a copy of base.txz."  Well, where is that supposed to come from?  \[[24]\]  We're going to pull ours down with a `fetch` like so:
{% highlight shell %}
fetch ftp://ftp.freebsd.org/pub/FreeBSD/releases/amd64/12.1-RELEASE/base.txz
{% endhighlight %}

To see what we'll have available to choose from for our hardware, we can visit with a browser:
{% highlight shell %}
ftp://ftp.freebsd.org/pub/FreeBSD/releases/amd64
{% endhighlight %}

For our template jail we'll call:
{% highlight shell %}
fetch ftp://ftp.freebsd.org/pub/FreeBSD/releases/amd64/12.1-RELEASE/base.txz
fetch ftp://ftp.freebsd.org/pub/FreeBSD/releases/amd64/12.1-RELEASE/lib32.txz
fetch ftp://ftp.freebsd.org/pub/FreeBSD/releases/amd64/12.1-RELEASE/src.txz
{% endhighlight %}

### Options, Options, Everywhere
In every set of directions, we'll notice variance over where to put different kinds of configuration information.  We'll follow Lucas' example for setting up a jail.conf file.  That will cover ideas like jail state changes, interfaces, and virtual networking for the jail.  How does a jail operate and how does it relate to resources?  We'll put those in jail.conf. \[2\]\[3\]

We'll add /etc/rc.conf items into individual jails for doing things like controlling services that run inside the jail.  \[[24]\]  

When we're preparing to compile from ports and set up the jails, we'll use /etc/make.conf variables for directing the compilation process towards jail-related ports.  \[[24]\]

On the host's /etc/rc.conf, we'll put values that enable jails and VMs.  

### A Major Pitfall
When we were running an earlier experiment, we ran into a real hazard with some of the kernel modules for VirtualBox.  We will need to be very careful to test out kernel modules produced during the vbox compilation process with `kldload`.  If we load up a kernel module that won't work well onto a GELI-encrypted ZFS drive, then we'll have a very difficult recovery.  So, we'll need to pick carefully before installing the kernel modules into the host's boot.conf.  This hazard won't present itself until later, but it'll be related to editing the jails and the host when we begin installing applications.

### We Get netcat For Free
On the plus side, `nc` is a default command for FreeBSD base installs.  We'll experiment with this some later.  As we have already mentioned, `fetch` is also automatically included; it's FreeBSD's sister to `wget`.  

### netcat Doesn't Work The Same
For those of us who've been training for pentests with Kali Linux \[4\], it's common to see the following uses of netcat:
{% highlight shell %}
nc -nvlp 192.168.1.100 4444 -e /bin/bash
{% endhighlight %}

However, this presents a problem:  FreeBSD uses a different flavor of `nc`.  The `-e` argument will not trigger the execution of a program.  Instead, we will have to use `fifo` and pipes to set up something similar:

{% highlight shell %}
        rm -f /tmp/f; mkfifo /tmp/f
        cat /tmp/f | /bin/sh -i 2>&1 | nc -l 127.0.0.1 1234 > /tmp/f
{% endhighlight %}
\[[25]\]\[[26]\]

`fifo`s are temporary channels in FreeBSD.  Then man pages tell us:

>One possible application of memory channels created by memchan or fifo is as temporary storage device to collect data coming in over a pipe or a socket.
\[[27]\]

In the code above, we read:
- remove any temp file named f
- make a fifo channel at temp f

Then:
- concatenate temp f and send it to shell, interactive
- pipe that shell to netcat listening on localhost to port 1234
- and send its output to temp f.

Notice how the last line sets up a circular reference.  The data will flow out of the fifo, into shell, into the netcat channel, and into the fifo.  \[[27]\]\[[28]\]  We will be using the memory channel, the `fifo`, to weld together the interactive shell and the netcat socket connection using pipes and redirection.  

Why is this a continuous, circular reference?  Because `cat` keeps reading until it hits EOF, or "end of file."  
>If file is a UNIX domain socket, cat connects to it and then reads it until EOF.  \[[29]\]

So, it'll be continuously reading out into the shell and the connection will be continuously redirecting into the `fifo` through a pipe. Given these ideas, we can build our usual bind and reverse shells in FreeBSD.  Later, we'll use them as a communications mechanism among our jails. 

#Basic Jail Install
Let's set up some jails.  When we're establishing jails, there are many potential paths we can follow.  We can see that there are a few ways we can get the OS into the jail, set up critical configuration facts, and populate the jails with programs like user applications.  If you're hunting around through tutorials, the variety of these paths can seem a little confusing.  We recommend reading and experimenting until you understand each hacker's choices.  
We'll start by mostly following Lucas' FreeBSD Mastery:  Jails.  \[30\]However, instead of just wrapping up our worlds into a temp file as he does in Chapter 2, we'll also fetch our base zipfiles from the FreeBSD Foundation over the Internet.

Logged on as root, we'll setup coffeehouse as our first, template, jail.  We'll put our txz files, extracted, in there.  As Lucas suggested, we'll keep a copy in a /jail/media subdirectory.  Many of the examples which follow are derived from Lucas' examples. \[30\]

### Fetching a Base System With the Same Version as The Jailhost
For our first trials, let's start with using the same kind as we have on hand.  For this example, that's 12.1-RELEASE.  

To make that directory, we'll use commands like:
{% highlight shell %}
mkdir /zroot/jail/media/amd64/12.1-RELEASE/
{% endhighlight %}\[30\]

To get the files, from inside the media subdirectory that will hold the files, we'll tell the computer:
{% highlight shell %}
fetch ftp://ftp.freebsd.org/pub/FreeBSD/releases/amd64/12.1-RELEASE/base.txz
fetch ftp://ftp.freebsd.org/pub/FreeBSD/releases/amd64/12.1-RELEASE/lib32.txz
fetch ftp://ftp.freebsd.org/pub/FreeBSD/releases/amd64/12.1-RELEASE/src.txz
{% endhighlight %}\[30\]

Then, we'll tell the computer to extract those files with tape archive and copy them to the part of memory where we'll put the jail.
{% highlight shell %}
tar -xpf /zroot/jail/media/amd64/12.1-RELEASE/base.txz -C /zroot/jail/coffeehouse
tar -xpf /zroot/jail/media/amd64/12.1-RELEASE/lib32.txz -C /zroot/jail/coffeehouse
tar -xpf /zroot/jail/media/amd64/12.1-RELEASE/src.txz -C /zroot/jail/coffeehouse
{% endhighlight %}\[30\]

In the above commands, we see that we're telling the computer:
- use tape archive
- -xpf:
    + extract
    + preserve permissions
    + of files at the following directory:  /zroot/jail/media/...
- -C:  create an archive at the specified directory /zroot/jail/coffeehouse
\[31\]

## Building From an Imported System of Another Version

We could also fetch the distribution files in for later use.  This technique will allow us to build jails of a different release of FreeBSD.  We can't use a release that's younger than the jailhost's OS, but we can use a version that's older.  Jails are operating system virtualization; they're containers; so, if we want to preserve a system running an older configuration of the OS, importing a system of an earlier release could be the basis of such a jail.  Or, we could establish the jail, import the user and application files, and then stand it up inside the more recent version of the OS on the jailhost.  

We can anticipate that there will be problems with mixing versions.  For example, what about running a much older and vulnerable application in an older jail version that's on top of a newer jailhost OS version?  We could be in a situation where we can get the older applications to run; but, that could be part of our problem, too.  We can't stereotype an answer that will always be safe, stable, and perfect; but, we can see that it will be possible to set up another release of FreeBSD inside our jailhost.  

## Alternative Method:  Building From the Host's System
We came across an alternative method that looks promising, but we chose not to use:  building from the host's own system.  In our this approach, we'll use the host's own system to make the tarfiles we need.  Later, we'll `fetch` some files in.  

To make the base, using our current installation:
{% highlight shell %}
mkdir /tmp/jail
cd /usr/src
make cleanworld
make -j4 buildworld 
make installworld DESTDIR=/tmp/jail
make distribution DESTDIR=/tmp/jail
{% endhighlight %}
 \[[30]\]\[32\]

 I'd like to note that some of those `make` commands can take awhile.  I usually do the `make buildworld` overnight.  `installworld` also takes awhile, but not as long.  `distribution` took over an hour.  These were some of the most time consuming commands of the process.  

 From there, we'll zip it up in a tarfile.
{% highlight shell %}
tar -cd /jail/media/\[FILENAME\].txz --xz -C /tmp/jail/
{% endhighlight %}
 \[[30]\]
 
The above command would provide me with an error; I had no luck with this method.  It seemed as though there were a circular reference somewhere in the process.  Given my own inability to really understand the output of these processes, we chose not to follow this path.

### Possible Alternative:  Using `nanobsd.sh` to Build a Read-Only Embedded System
Quietly, for about the past ten years, FreeBSD has been providing a tool from Paol Henning-Kamp that build a small read-only version of FreeBSD called nanoBSD.  There are few references about this system, but we're interested because, by default, it puts everything into a read-only state.  That, and it's small size; nanobsd is small enough to fit on a compact flash.  \[[33]\]\[[34]\]\[[35]\]\[[36]\]

Unfortunately, there is a critical function in the scripts that does not work with the ZFS file system.  When the script approaches create_diskimage in legacy.sh, it will have already failed.  As we exit the script, we should find a \_.disk* directories that are not there.  Without them, we won't be able to transfer the products to the target directories using a `cp`.  

As we trace our way through the nanobsd scripts, we come to some critical points:

{% highlight shell %}
    echo "/dev/${NANO_DRIVE}${NANO_ROOT} / ufs ro 1 1" > etc/fstab
    echo "/dev/${NANO_DRIVE}${NANO_SLICE_CFG} /cfg ufs rw,noauto 2 2" >>  
{% endhighlight %}
\[[37]\]

We can plainly see that the UFS file system is what's used with nanobsd.  This wasn't mentioned in the man pages, but there it is.  Given that troubleshooting this script is kind of outside of the scope of this project, we'll stick to the "Fetching a Base System" procedure, mentioned above.  Perhaps we'll come back to edit this later for use with ZFS.  It doesn't look like that has been contributed to FreeBSD yet.

### Provisioning the Jail with the Operating System
With a copy of the OS staged, it's time for us to set it up in a part of the file system that we'll use for jails.  We'll load up a jail with just a base system.  We'll set a copy of that aside.

From our home directory for root:
{% highlight shell %}
tar -zpf /jail/media/12.1-RELEASE/base.txz -C /zroot/jail/coffeehouse
tar -zpf /jail/media/12.1-RELEASE/lib32.txz -C /zroot/jail/coffeehouse
tar -zpf /jail/media/12.1-RELEASE/src.txz -C /zroot/jail/coffeehouse
{% endhighlight %}

We can verify and protect what we did with:
{% highlight shell %}
ls /zroot/jail/coffeehouse
zfs snapshot zroot/jail/coffeehouse@<DATE_TIME_IDENTIFIER>
{% endhighlight %}

### Defining the Jail Directory as a Jail
So far, we've just loaded what's needed into a designated spot in the file system.  We are about to give the commands which will help that directory stand up and perform as its own jail on command.  

First, we have to edit the host's `/etc/rc.conf` to permit jails to be used:
{% highlight shell %}
nano /etc/rc.conf

jail_enable="YES"
jail_parallel_start="YES"
{% endhighlight %}

Next, we'll have to tell the system about each jail as we build it.  We provide this in `/etc/jail.conf`.  Later, we'll have a more complicated file.  For now we'll just try to establish a jail and start it.

{% highlight shell %}
nano /etc/jail.conf

exec.start="/bin/sh /etc/rc";
exec.stop="/bin/sh /etc/rc.shutdown";
exec.clean;
mount.devfs;

coffeehouse{
    host.hostname="coffeehouse.salvage13.local";
    ip4.addr="192.168.1.192/27";
    path="/zroot/jail/coffeehouse";
}

{% endhighlight %}
\[[38]\]

We see if we can start up the jail:
{% highlight shell %}
jail /zroot/jail/coffeehouse/ coffeehouse 192.168.1.192/27 /bin/sh
{% endhighlight %}
\[[39]\]

Exiting out of that, we copy in a resolve:
{% highlight shell %}
cp /etc/resolv.conf /zroot/jail/coffeehouse/etc/resolv.conf
{% endhighlight %}

Back in, to follow Lucas' book better, we continue:
{% highlight shell %}
touch /etc/fstab
vi /etc/rc.conf
sshd_enable="YES"
exit
service jail start coffeehouse
jls
{% endhighlight %}
We should be able to see our coffeehouse jail up and running.
\[[39]\]

Let's preserve these small but valuable changes:
{% highlight shell %}
zfs snapshot zroot/jail/coffeehouse@<DATE_TIME_IDENTIFIER>
{% endhighlight %}

Our jail needs users.  Let's add a user.  From outside the jail, we give:
{% highlight shell %}
jexec coffeehouse adduser
{% endhighlight %}

We add the user "coffee" to `wheel` and `operator`.  We can take a snapshot.

### Copy Config Information to a Snapshot
Since most of our system configuration changes are in the base system of the host, we'll want to be able to make a separate copy of some files in an area that we can conserve with a `zfs snapshot`.  A simple script can help us grab copies of files that we're likely to change over the next little while.

{% highlight shell %}

\# If the base directory to hold the common configs is not present,
\# then build one.
if \[ ! -d /zroot/var \]
then
    mkdir /zroot/var
fi

if \[ ! -d /zroot/var/common \]
then
    mkdir /zroot/var/common
fi

\# Create a fresh directory to hold common config files
\# Remove the old one from earlier today, if necessary.
\# Record when we are taking the copy of the config files
someDate=$(date +%Y-%m-%d:%H:%M);
if \[ -d /zroot/var/common/$someDate \]
then
    rm -r /zroot/var/common/$someDate
fi
mkdir /zroot/var/common/$someDate
echo $someDate > /zroot/var/common/$someDate/_snap_$someDate.txt
hash=$(sha512 -q -s `echo $someDate` )
echo $hash >> /zroot/var/common/$someDate/_snap_$someDate.txt

\# Copy some common config files
if \[ -f /boot/boot.conf \]
then
    cp /boot/boot.conf /zoot/var/common/$someDate/
fi

if \[ -f /etc/rc.conf \]
then
    cp /etc/rc.conf /zroot/var/common/$someDate/
fi

if \[ -f /etc/devfs.conf \]
then
    cp /etc/devfs.conf /zroot/var/common/$someDate/
fi

if \[ -f /etc/jail.conf \]
then
    cp /etc/jail.conf /zroot/var/common/$someDate/
fi

\# Record available jails and zfs facts
jls > /zroot/var/common/$someDate/jlsOutput.txt
vmstat > /zroot/var/common/$someDate/vmstatOutput.txt
zfs list -t snapshot > /zroot/var/common/$someDate/snapshotsOutput.txt
zpool list > /zroot/var/common/$someDate/zpoolOutput.txt
zfs list > /zroot/var/common/$someDate/zfsListOutput.txt

\# Take a snapshot of common
zfs snapshot zroot/var/common@$someDate

exit

{% endhighlight %}


## VNET Setup
We'll give our jail a spot on a VNET.  WE'll follow Lucas' "FreeBSD Mastery: Jails" chapter 9 for most of this work.  From there, we'll expand.  Our goal will be to get the jails on the network in a way we can verify with another local computer.  As we build our other jails, we'll give each one a subnet that accommodates 6 hosts.  Using our plan for subnetting established earlier, we'll begin by calling this first jail 192.168.1.192/27.  

Our goal for subnetting was to be able to group VMs together in a subnet, organized by jails.  At this stage, we don't have all of that installed.  So, in order to have something testable and observable, we'll have to start by putting the jail coffeehouse as a host.  Later, we'll assign those host addresses to the VMs.  

To lay in VNET on the jailhost, we'll need to modify rc.conf and jail.conf.  In this example, we'll use a separate physical interface as the physical NIC for the jails.  

First, we had to comment out previously applied networking in jailhost's rc.conf.  Then we added an interface and gave it an explicit address and netmask.  Our netmask on this one ends in a e0, because we're giving a 255.255.2255.224 for a CIDR /27 at this level.

{% highlight shell %}
\#ifconfig_bce0="192.168.1.190"
\#ifconfig_bce0_ipv6="inet6 accept_rtadv"
\# Jail-Related Settings
jail_enable="YES"
jail_list="coffeehouse"
\#jail_parallel_start="YES"
\# VNET-Related Settings
ifconfig_bce1_name="jailether"
ifconfig_jailether="inet 192.168.1.192 netmask 0xffffffe0"
{% endhighlight %}

To help us with the VNET, and to follow along with Lucas, we've used jib.  It comes pre-installed with FreeBSD.  It can be found in `/usr/share/examples/jails/`.  That script does a lot of `ifconfig` functions for us, and it trims up the MAC address.  There is a similar one called `jng` right beside it in the examples folder.  

The jailhost's /etc/jail.conf gets set up for VNET.  We can see that we've attached the "B" end of the epair from jib to the jail.

{% highlight shell %}
\# JPO gladiola 02JAN2020
\# CONF to establish jail
\# REF:  Lucas, Michael.  FreeBSD Mastery:  Jails.  ISBN 978-1-64235-024-1
$j="/zroot/jail";
path="$j/$name";
host.hostname="$name.salvage13.local";
exec.clean;
exec.start="/bin/sh /etc/rc";
exec.stop="/bin/sh /etc/rc.shutdown";
mount.devfs;
exec.consolelog="/var/tmp/$name";

coffeehouse{
    vnet;
    vnet.interface="e0b_coffeehouse";
    exec.prestart="/usr/local/scripts/jib addm coffeehouse jailether";
    exec.poststop="/usr/local/scripts/jib destroy coffeehouse";
    devfs_ruleset=4;
}
{% endhighlight %}

In the jail's directories, we'll need to anchor the other end of the connection.  We'll give the e0b an address.  Above those lines, we can see how we'd setup ifconfig for the netmask needed to split into 6 hosts.  That will use a netmask ending in "f8" for a decimal netmask of 255.255.255.248 for a CIDR /29.  For now, it has the one address assigned because those others are not installed.

{% highlight shell %}
sshd_enable="YES"
inetd_enable="YES"
\#ifconfig_e0b_coffeehouse="inet 192.168.1.192 netmask 255.255.255.2";
\#ifconfig_e0b_coffeehouse="inet 192.168.1.192 netmask 0xfffffff8";
ifconfig_e0b_coffeehouse="192.168.1.193";
defaultrouter="192.168.1.1";
{% endhighlight %}

Reboot the machine.  We can get into the jail with
{% highlight shell %}
jexec -U root coffeehouse sh
{% endhighlight %}

From there, we can do some simple pings to google.com to see that the jail has connectivity with the internet.  A scan of my home network shows the .193 on the router.  If we try to ssh into .193 from another machine, not a part of that subnet, we'll find that we have a destination host unreachable.

Now's a good time to take a snapshot and record some notes on the settings.  Our next phase will be to stand up virtual machines using `bhyve`.

## Bhyve Hypervisor Installation
We chose bhyve because it is built in.  In an earlier prototype, I began with virtualbox-ose because that's what I was familiar with.  I made an error with kernel mods and locked up that prototype pretty bad.  Virtualbox gets cussed frequently, but it's one of the most widely used hypervisors at home.  We had found a great tutorial script to set up vbox VMs from a script.  

https://www.andreafortuna.org/2019/10/24/how-to-create-a-virtualbox-vm-from-command-line/

That one, in particular, was a tutorial good enough to help us realize that this might be possible.  Notice that it's about running vbox after it's installed; my prototype troubles had been with the kernel modules needed to get it all running in FreeBSD.  The script provided worked.   Also, Chapter 8 of the Virtualbox manual was worth reading because it covers many of the commands referred to in these scripts. \[[43]\] \[[44]\] \[[45]\] \[[46]\]

{% highlight shell %}
{% endhighlight %}
 \[[]\]
## Annotated Bibliography
\[1\]  FreeBSD Foundation.  INTERNET: [`https://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/`](https://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/)

\[2\]  Lucas, Michael W.  Absolute FreeBSD:  The Complete Guide to FreeBSD.  No Starch Press.  ISBN 978-1-59327-892-2.

\[3\]  Lucas, Michael W.  FreeBSD Mastery:  Jails.  Tilted Windmill Press.  ISBN 978-1-64235-024-1.

\[4\]  Offensive Security.  Penetration Testing with Kali Linux.  Laboratory Manual, v. 1.1.6.  Controlled distribution, proprietary manual for commercial certification testing, made available to registered students with Offensive Security course for Penetration Testing with Kali Linux.

\[5\]  Eisele, Robert.  "CIDR Subnet Calculator"  INTERNET: [`https://www.xarg.org/tools/subnet-calculator/?q=192.168.1.192%2F27%2F29`](https://www.xarg.org/tools/subnet-calculator/?q=192.168.1.192%2F27%2F29)

A free online subnet calculator that uses CIDR.

\[6\]  FreeBSD Foundation.  INTERNET: [`https://www.freebsd.org/releases/12.1R/announce.html`](https://www.freebsd.org/releases/12.1R/announce.html)

Information about the release.
    
\[7\]  FreeBSD Foundation.  INTERNET: [`https://www.freebsd.org/where.html`](https://www.freebsd.org/where.html)

\[8\]  FreeBSD Foundation.  INTERNET: [`https://download.freebsd.org/ftp/releases/amd64/amd64/ISO-IMAGES/12.1`](https://download.freebsd.org/ftp/releases/amd64/amd64/ISO-IMAGES/12.1)
   
Page that allows us to choose the type of image to download.   

\[9\]  FreeBSD Foundation.  INTERNET: [`https://download.freebsd.org/ftp/releases/amd64/amd64/ISO-IMAGES/12.1/FreeBSD-12.1-RELEASE-amd64-dvd1.iso`](https://download.freebsd.org/ftp/releases/amd64/amd64/ISO-IMAGES/12.1/FreeBSD-12.1-RELEASE-amd64-dvd1.iso)

Chosen download for our host install.

\[10\]  FreeBSD Foundation.  INTERNET: [`https://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/using-bsdinstall.html`](https://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/using-bsdinstall.html)

Directions for using bsdinstall.

\[11\]  FreeBSD Foundation.  INTERNET: [`https://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/updating-upgrading-freebsdupdate.html`](https://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/updating-upgrading-freebsdupdate.htmll)

Using freebsd-update and related commands to install security updates.

\[12\]  FreeBSD Foundation.  INTERNET: [`https://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/ports-using.html`](https://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/ports-using.html)

Using portsnap fetch and related commands to install fresh scripts for ports.  This may help ensure that the latest security patches are available to programs compiled from ports.

\[13\] Lucas.  \[2\], pp. 258, 264, 271-272.

\[14\] Lucas.  \[2\], pp. 262.

\[15\] FreeBSD Foundation.  INTERNET:  [`https://www.freebsd.org/cgi/man.cgi?query=pw&sektion=8&manpath=freebsd-release-ports`](https://www.freebsd.org/cgi/man.cgi?query=pw&sektion=8&manpath=freebsd-release-ports)

Online manpage for pw.

\[16\] _____.  INTERNET:   [`https://www.pngtube.com/viewm/TRRmb_png-transparent-download-bonfire-clipart-smoke-fire-png/`](https://www.pngtube.com/viewm/TRRmb_png-transparent-download-bonfire-clipart-smoke-fire-png/)

Clipart flames download.

\[17\] Lucas.  \[2\], pp. 271-273.

\[18\] _____.  INTERNET:   [`https://www.thegeekdiary.com/zfs-tutorials-creating-zfs-snapshot-and-clones/`](https://www.thegeekdiary.com/zfs-tutorials-creating-zfs-snapshot-and-clones/)

\[19\] _____.  INTERNET:   [`https://www.unix.com/shell-programming-and-scripting/174496-how-check-if-file-exists-directory.html`](https://www.unix.com/shell-programming-and-scripting/174496-how-check-if-file-exists-directory.html)

sh syntax for checking if a file exists.

\[20\] Lucas.  \[2\], pp. 260.

\[21\] _____.  INTERNET:   [`https://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/jails-build.html`](https://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/jails-build.html)

Handbook page on the basics of building jails.

\[22\] _____.  INTERNET:   [`https://www.freebsd.org/doc/handbook/ports-using.html`](https://www.freebsd.org/doc/handbook/ports-using.html)

Handbook page on using ports.  Notice the variables for WKDIRPREFIX and PREFIX.

\[23\] _____.  INTERNET:   [`https://www.cyberciti.biz/faq/how-to-configure-a-freebsd-jail-with-vnet-and-zfs/`](https://www.cyberciti.biz/faq/how-to-configure-a-freebsd-jail-with-vnet-and-zfs/)

Blog post on setting up jails with VNET and ZFS.

\[24\] _____.  INTERNET:   [`https://savagedlight.me/2014/03/14/freebsd-jail-server-with-zfs-clone-and-jail-conf/`](https://savagedlight.me/2014/03/14/freebsd-jail-server-with-zfs-clone-and-jail-conf/)

Blog post on setting up jails with ZFS.

\[25\] _____.  INTERNET:   [`https://superuser.com/questions/691008/why-is-the-e-option-missing-from-netcat-openbsd#691043`](https://superuser.com/questions/691008/why-is-the-e-option-missing-from-netcat-openbsd#691043)

\[26\] _____.  INTERNET:   [`https://www.freebsd.org/cgi/man.cgi?query=nc&apropos=0&sektion=0&manpath=FreeBSD+12.1-RELEASE+and+Ports&arch=default&format=html`](https://www.freebsd.org/cgi/man.cgi?query=nc&apropos=0&sektion=0&manpath=FreeBSD+12.1-RELEASE+and+Ports&arch=default&format=html)

\[27\] _____.  INTERNET:   [`https://www.freebsd.org/cgi/man.cgi?query=fifo&apropos=0&sektion=0&manpath=FreeBSD+12.1-RELEASE+and+Ports&arch=default&format=html`](https://www.freebsd.org/cgi/man.cgi?query=fifo&apropos=0&sektion=0&manpath=FreeBSD+12.1-RELEASE+and+Ports&arch=default&format=html)

What's a `fifo`?

\[28\] _____.  INTERNET:   [`https://www.freebsd.org/cgi/man.cgi?query=mkfifo&apropos=0&sektion=0&manpath=FreeBSD+12.1-RELEASE+and+Ports&arch=default&format=html`](https://www.freebsd.org/cgi/man.cgi?query=mkfifo&apropos=0&sektion=0&manpath=FreeBSD+12.1-RELEASE+and+Ports&arch=default&format=html)

What does `mkfifo` do?

\[29\] _____.  INTERNET:   [`https://www.freebsd.org/cgi/man.cgi?query=cat&apropos=0&sektion=0&manpath=FreeBSD+12.1-RELEASE+and+Ports&arch=default&format=html`](https://www.freebsd.org/cgi/man.cgi?query=cat&apropos=0&sektion=0&manpath=FreeBSD+12.1-RELEASE+and+Ports&arch=default&format=html)

\[30\]  Lucas.  \[3\] pp.27-48.

This is the main effort of his chapter on "Jail Essentials."  It outlines a method for setting up a standard jail.conf to hold a default pattern for all the jails on machine.  It also describes the setup of a basic jail.  It covers the importation of the operating system base files and other similar basic aspects of getting a jail functioning.

\[31\] _____.  INTERNET:   [`https://www.freebsd.org/cgi/man.cgi?query=tar&apropos=0&sektion=0&manpath=FreeBSD+12.1-RELEASE+and+Ports&arch=default&format=html`](https://www.freebsd.org/cgi/man.cgi?query=tar&apropos=0&sektion=0&manpath=FreeBSD+12.1-RELEASE+and+Ports&arch=default&format=html)

Man pages for tar, tape archive.

\[32\] _____.  INTERNET:   [`https://www.freebsd.org/doc/handbook/makeworld.html`](https://www.freebsd.org/doc/handbook/makeworld.html)

FreeBSD Handbook page showing some examples of `make buildworld` and similar commands.

\[33\] _____.  INTERNET:   [`https://www.freebsd.org/doc/en_US.ISO8859-1/articles/nanobsd/howto.html`](https://www.freebsd.org/doc/en_US.ISO8859-1/articles/nanobsd/howto.html)

FreeBSD article and walkthrough of setting up a nanoBSD system on a flash drive.

\[34\] _____.  INTERNET:   [`https://www.freebsd.org/cgi/man.cgi?query=nanobsd&apropos=0&sektion=8&manpath=FreeBSD+8.1-RELEASE&arch=default&format=html`](https://www.freebsd.org/cgi/man.cgi?query=nanobsd&apropos=0&sektion=8&manpath=FreeBSD+8.1-RELEASE&arch=default&format=html)

nanoBSD man page.

\[35\] _____.  INTERNET:   [`https://www.youtube.com/watch?v=5qCaOMQ3ZnQ&t=13s`](https://www.youtube.com/watch?v=5qCaOMQ3ZnQ&t=13s)

YouTube video showing Warner Losh, a FreeBSD maintainer for nanobsd, speaking about nanobsd at BSDCan 2011.

\[36\] _____.  INTERNET:   [`https://www.youtube.com/watch?v=RHLRW88AJLE`](https://www.youtube.com/watch?v=RHLRW88AJLE)

YouTube video showing a talk from EuroBSDcon, "FreeBSD as a Hosting Platform, Revisited - Patrick M. Hausen."  Hausen's company uses nanobsd to establish images for hosting accounts with his hosting company.


\[37\] _____.  FreeBSD 12.1-RELEASE:   [`/usr/src/tools/tools/nanobsd`]

Source code directory for nanobsd in FreeBSD 12.1.  NanoBSD has been included with FreeBSD efforts since near 2006.  

\[38\] Lucas.  \[2\], pp. 568-572.

We're following Lucas' configuration verbatim, with slight adjustments.

\[39\] Lucas.  \[2\], pp. 573-574.

We provide some basic config and commands, following Lucas.

\[40\] _____.  INTERNET:   [`https://www.subnetonline.com/pages/subnet-calculators/dec-to-hex-calculator.php`](https://www.subnetonline.com/pages/subnet-calculators/dec-to-hex-calculator.php)

Helped us with the hex subnet masks when we were tired.

\[41\] Lucas.  \[3\], all of Chapter 9.

Lucas' chapter on jails and networking, which built on some of his earlier demonstrations in FreeBSD Mastery:  Jails, is our guide for most of this process.

\[42\] _____.  INTERNET:   [`https://forums.freebsd.org/threads/jails-vnet-freebsd-mastery-multiple-interfaces.70356/`](https://forums.freebsd.org/threads/jails-vnet-freebsd-mastery-multiple-interfaces.70356/)

A FreeBSD forum post that builds off of Lucas' Chapter 9 and shows some hackers drafting some scripts to do some of the same tasks that jib does.  This path might be a viable course of action for more complex hierarchical jails.  Notice in some of their examples that the epair interfaces are explicitly defined.  We can't make all of those adjustments with jib.  So, with jib, we get something that works; but, with the scripts we would get more control over how the interface was implemented.  For now, we'll stick with jib.

\[43\] _____.  INTERNET:   [`https://www.andreafortuna.org/2019/10/24/how-to-create-a-virtualbox-vm-from-command-line/`](https://www.andreafortuna.org/2019/10/24/how-to-create-a-virtualbox-vm-from-command-line/)

\[44\] _____.  INTERNET:   [`https://www.techrepublic.com/article/how-to-run-virtualbox-virtual-machines-from-the-command-line/`](https://www.techrepublic.com/article/how-to-run-virtualbox-virtual-machines-from-the-command-line/)

\[45\] _____.  INTERNET:   [`https://www.linuxtechi.com/manage-virtualbox-virtual-machines-command-line/`](https://www.linuxtechi.com/manage-virtualbox-virtual-machines-command-line/)

\[46\] _____.  INTERNET:   [`https://www.virtualbox.org/manual/ch08.html`](https://www.virtualbox.org/manual/ch08.html)

Manual pages for virtualbox worth reading if you are going to script out a headless run of vbox.

\[\] _____.  INTERNET:   [``]()

\[\] _____.  INTERNET:   [``]()

\[\] _____.  INTERNET:   [``]()

\[\] _____.  INTERNET:   [``]()

\[\] _____.  INTERNET:   [``]()

[//]: # (Hyperlinks)
[1]: https://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/
[2]: ``
[3]: ``
[4]: ``
[5]: https://www.xarg.org/tools/subnet-calculator/?q=192.168.1.192%2F27%2F29
[6]: https://www.freebsd.org/releases/12.1R/announce.html
[7]: https://www.freebsd.org/where.html
[8]: https://download.freebsd.org/ftp/releases/amd64/amd64/ISO-IMAGES/12.1/
[9]: https://download.freebsd.org/ftp/releases/amd64/amd64/ISO-IMAGES/12.1/FreeBSD-12.1-RELEASE-amd64-dvd1.iso
[10]: https://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/using-bsdinstall.html
[11]: https://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/updating-upgrading-freebsdupdate.html
[12]: https://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/ports-using.html
[13]: ``
[14]: ``
[15]: https://www.freebsd.org/cgi/man.cgi?query=pw&sektion=8&manpath=freebsd-release-ports
[16]: https://www.pngtube.com/viewm/TRRmb_png-transparent-download-bonfire-clipart-smoke-fire-png/
[17]: ``
[18]: https://www.thegeekdiary.com/zfs-tutorials-creating-zfs-snapshot-and-clones/
[19]: https://www.unix.com/shell-programming-and-scripting/174496-how-check-if-file-exists-directory.html
[20]: ``
[21]: https://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/jails-build.html
[22]: https://www.freebsd.org/doc/handbook/ports-using.html
[23]: https://www.cyberciti.biz/faq/how-to-configure-a-freebsd-jail-with-vnet-and-zfs/
[24]: https://savagedlight.me/2014/03/14/freebsd-jail-server-with-zfs-clone-and-jail-conf/
[25]: https://superuser.com/questions/691008/why-is-the-e-option-missing-from-netcat-openbsd#691043
[26]: https://www.freebsd.org/cgi/man.cgi?query=nc&apropos=0&sektion=0&manpath=FreeBSD+12.1-RELEASE+and+Ports&arch=default&format=html
[27]: https://www.freebsd.org/cgi/man.cgi?query=fifo&apropos=0&sektion=0&manpath=FreeBSD+12.1-RELEASE+and+Ports&arch=default&format=html
[28]: https://www.freebsd.org/cgi/man.cgi?query=mkfifo&apropos=0&sektion=0&manpath=FreeBSD+12.1-RELEASE+and+Ports&arch=default&format=html
[29]: https://www.freebsd.org/cgi/man.cgi?query=cat&apropos=0&sektion=0&manpath=FreeBSD+12.1-RELEASE+and+Ports&arch=default&format=html
[30]: ``
[31]: https://www.freebsd.org/cgi/man.cgi?query=tar&apropos=0&sektion=0&manpath=FreeBSD+12.1-RELEASE+and+Ports&arch=default&format=html
[32]: https://www.freebsd.org/doc/handbook/makeworld.html
[33]: https://www.freebsd.org/doc/en_US.ISO8859-1/articles/nanobsd/howto.html
[34]: https://www.freebsd.org/cgi/man.cgi?query=nanobsd&apropos=0&sektion=8&manpath=FreeBSD+8.1-RELEASE&arch=default&format=html
[35]: https://www.youtube.com/watch?v=5qCaOMQ3ZnQ&t=13s
[36]: https://www.youtube.com/watch?v=RHLRW88AJLE
[37]: ``
[38]: ``
[39]: ``
[40]: https://www.subnetonline.com/pages/subnet-calculators/dec-to-hex-calculator.php
[41]: ``
[42]: https://forums.freebsd.org/threads/jails-vnet-freebsd-mastery-multiple-interfaces.70356/
[43]: https://www.andreafortuna.org/2019/10/24/how-to-create-a-virtualbox-vm-from-command-line/
[44]: https://www.techrepublic.com/article/how-to-run-virtualbox-virtual-machines-from-the-command-line/
[45]: https://www.linuxtechi.com/manage-virtualbox-virtual-machines-command-line/
[46]: https://www.virtualbox.org/manual/ch08.html
[47]: 




