---
title:  "Building an Amnesiac FreeBSD Server: Blondie"
date:   2026-09-09 08:30:00
description: Designing a FreeBSD server that forgets itself on every boot, using a read-only live disc, GELI-encrypted ZFS, and jails.
---

We wanted a server that couldn't be permanently poisoned.  Not "hard to hack" -- amnesiac.  Every time it powers on, the operating system loads fresh off a read-only disc into RAM, and the working data lives on drives that can be unplugged and handed to a totally different machine.  This post walks through why we built it that way, how the hardware came together, and the actual FreeBSD commands we used to partition, encrypt, and pool the drives.

# Why Bother With "Amnesiac"?

Almost every destructive attack technique needs to write something to the target: a dropped payload, a modified registry key, a scheduled task, a poisoned config file, a wiped disk.  Looking through MITRE ATT&CK's Enterprise techniques, we counted around 60 technique groups -- things like Create Account, Data Destruction, Disable or Modify Tools, Modify Registry, Rootkit, and Scheduled Task/Job -- that all boil down to an attacker needing to write to the system at some point in the chain.  We're not claiming this list is exhaustive or independently verified; it's our own read of the techniques, done without a red team to check our work.  But the pattern is strong enough to be a decent design principle: if the core operating system can't be written to at all, a whole category of attacks loses its footing.

That gives us four things we wanted to test, even informally:

- **H1** -- Attackers can't destructively interact with a system without writing to it.
- **H2** -- An immutable OS is more resistant to destructive attack.
- **H3** -- An immutable OS promotes system integrity.
- **H4** -- An immutable OS promotes rapid recovery.

To be clear up front: this is a design-level case study, not a controlled experiment.  We haven't run a red-team campaign against Blondie, we haven't benchmarked its recovery time against a normal backup-and-restore setup, and we haven't compared physically-enforced immutability (a disc you have to walk over and re-burn) against the software-only immutability used by things like Tails or Fedora Silverblue.  Those are exactly the follow-up experiments we'd want to run next. What we do have is a working build, the reasoning behind it, and the commands to reproduce it.

# What "Immutable" Actually Means Here

Reading around on the subject, we liked a three-way split of immutability into **hard**, **soft**, and **operational** flavors: hard immutability means changes are technically infeasible; soft immutability means changes are possible but gated behind privileged credentials; operational immutability means nothing stops you technically, but a process or policy does.

Blondie ends up using all three:

- **Hard immutability** for the core OS: it's burned onto a read-only optical disc.  Changing it means building a new disc and physically walking it over to the server.
- **Operational immutability** as a companion control: the server itself requires physical access to swap discs, and there's no remote way to push a new OS image.
- **Soft immutability** for data on the attached drives, which isn't loaded automatically at boot and requires deliberate action (and a passphrase) to mount.

One lesson from looking at fully-immutable systems (and from GDPR's "right to be forgotten," which flatly assumes some data has to be deletable) is that making *everything* immutable is a trap.  Personal data, logs that might need redaction, anything with a legal retention/erasure requirement -- none of that belongs baked into a read-only image.  So we deliberately kept only the core OS and jail configuration hard-immutable, and left everything else on mutable, disconnectable storage.

Systems like Tails already do something similar in spirit -- boot from removable media, run from RAM, leave nothing behind -- but Tails targets a single session on end-user hardware.  Immutable container node images (Flatcar, Fedora CoreOS) and desktop distros like Silverblue or NixOS enforce read-only behavior entirely in software with atomic image swaps.  Our twist is stacking a *physical* write barrier (the optical disc) underneath those same software-level ideas, on the theory that a software-only read-only mount can potentially be defeated by an attacker who escalates privileges, while a write-once physical disc cannot be rewritten by any software-only attack.  We haven't measured whether that actually buys us anything over a purely software-enforced approach -- that's on the future work list.

# The Hardware: Meet Blondie

We started with a Dell T610 pulled from a government e-waste auction -- 2011-era enterprise gear, dusty, and stripped of most of its good parts, which is typical for this kind of hardware.  After a physical cleaning, we built it out to:

- Two 80GB VelociRaptor drives, mirrored, for the OS
- Two refurbished 120GB SSDs, mirrored, as ZFS log ("SLOG") drives
- Four 500GB drives in a RAIDZ-2 array for jails
- A DVD-RW drive (the whole point) and an LTO-III tape backup unit
- Dual Xeon 5550 quad-cores (swapped in for the stock CPU) with an added cooling tower
- 196GB of RAM
- Extra NICs: four for jails, two for the OS, and the onboard pair left mostly idle except for the iDRAC management interface

No GPU, no sound card, no wireless -- there's nothing here that isn't earning its keep. Because the dual 500W power supplies pull real power, this box got its own circuit.

The built-in hardware RAID controller couldn't be fully bypassed, and physically unplugging it caused malfunctions, so we made peace with it: every physical drive gets its own RAID-0 VDEV, meaning the hardware controller thinks it's doing RAID while ZFS does all the actual work above it. We recorded every drive's serial number, model, and capacity as we racked it, because power-cycling mid-setup just to check that stuff again is no fun.

# From "Regular Install" to "Amnesiac"

Our first attempt didn't actually get to amnesiac -- we ended up scripting a fairly conventional FreeBSD install with most of the features we wanted, but with the OS still living on disk. That was useful groundwork: it let us work out the ZFS partitioning commands before adding the complexity of a live disc.

For that first pass, each drive gets a three-way split:

- 15% left as blank "free space" (headroom for future swap or resize needs)
- Swap sized as a fraction of RAM depending on the drive's role
- The remainder as one large ZFS partition

Once that pattern worked, we started over using a boot-only live CD instead of an installed OS, encrypting partitions individually instead of encrypting whole disks, and being deliberate about saving GELI metadata somewhere durable. The final Blondie feature list:

- Immutable OS booted from a live disc into RAM
- GELI-encrypted disc partitions
- Swap and ZFS logging built into the partition layout
- Metadata and encryption keys saved off to a separate USB drive
- A dedicated set of drives for FreeBSD jails
- Room to expand without a redesign

# No Root Password

We don't set a root password at all -- SSH key only, and eventually gated behind TOTP-based 2FA and IP restrictions once the firewall is configured. That closes off password-guessing entirely, for root and otherwise, since most automated login attacks are dictionary attacks against a password prompt that no longer exists.

The tradeoff is real: no root password also kills `su`/`sudo` as an escalation path, which is mostly fine since almost everything we care about happens inside jails anyway. The bigger risk is key hygiene -- if you copy your private key onto the box "for convenience," you've basically recreated a stored plaintext password. We keep keys on a small set of dedicated USB drives instead.

To generate the keys from the live disc's shell:

```
ssh-keygen -b 1024 -t rsa
```

# Getting a USB Drive Usable From the Live Disc's Read-Only Shell

The live CD's shell environment is read-only by default, so before you can save anything (like those SSH keys), you need a writable place to put it. First, find the USB drive:

```
camcontrol devlist
```

Then partition and format it:

```
gpart create -s GPT da0
gpart add -t freebsd-ufs -a 1M da0
newfs -U /dev/da0p1
```

Mount it, write a test file, unmount, unplug, replug into a different port, and remount to confirm it's readable from elsewhere:

```
mkdir /tmp/someDir
mount /dev/da0p1 /tmp/someDir
```

Once that round-trip works, generate the keys straight onto the drive (pointing `ssh-keygen` at `/tmp/otherDir/keys` when prompted), and don't forget to record the passphrase somewhere durable.

# Laying Out and Encrypting the Drives

With the USB sorted, the rest of the drives get partitioned per their role. A representative slice of the commands we used, run against the live disc's `mfid*` device names:

```
# Swap disk in bay 0
gpart create -s GPT mfid0
gpart add -a 5K -s 395G -t freebsd-swap -l ZFS-BLONDIE-0-SWAP-WCC2ERZ56512 mfid0

# ZFS cache in bay 1
gpart create -s GPT mfid1
gpart add -a 5K -s 63G -t freebsd-zfs -l ZFS-BLONDIE-1-CACHE-WXL1E40C1352 mfid1

# ZFS SLOG mirror in bays 2 and 3
gpart create -s GPT mfid2
gpart add -a 5K -s 63G -t freebsd-zfs -l ZFS-BLONDIE-2-SLOG-WXL1E4067503 mfid2
gpart create -s GPT mfid3
gpart add -a 5K -s 63G -t freebsd-zfs -l ZFS-BLONDIE-3-SLOG-WXC0CA9V8431 mfid3

# Jail work disks in bays 4-7 (swap + files partitions each)
gpart create -s GPT mfid4
gpart add -a 5K -s 50G -t freebsd-swap -l ZFS-BLONDIE-4-SWAP-WCC2EE134983 mfid4
gpart add -s 345G -t freebsd-zfs -l ZFS-BLONDIE-4-FILES-WCC2EE134983 mfid4
```

We labeled everything with a `Host-Bay-SerialNumber` naming pattern so that if a drive gets physically moved, we can still tell what it is from the label alone. Note that "bay 7" in a label is our own note about physical position -- the device name FreeBSD assigns (`mfid7`) is whatever the hardware RAID controller happens to hand back, and that mapping can drift if the controller has trouble recognizing a drive. Always double-check `gpart show` after a reboot.

With partitions in place, GELI encrypts each one individually:

```
geli init -l 256 mfid0p1
geli attach mfid0p1
```

...repeated per partition. Every `geli init` writes a metadata backup file to an ephemeral directory that disappears on power-down, so the very next step is copying those `.eli` files somewhere that survives a reboot -- in our case, the same USB drive holding the SSH keys, renamed to match the disk label rather than the auto-generated filename:

```
cp /var/backups/mfid0p1.eli /tmp/usb3/backups_27SEP2020/ZFS-BLONDIE-0-SWAP-WCC2ERZ56512.eli
```

Skip that copy and a later recovery of that partition is effectively impossible. It's also worth remembering that anyone with root access and knowledge of the GELI passphrase can re-export that metadata later, so losing the backup file isn't automatically fatal -- but relying on that path means your "immutable" box now has a very mutable dependency on someone remembering a passphrase.

If you make a labeling mistake (we did -- a typo'd serial number on one partition), `gpart modify` fixes it without having to redo the encryption:

```
gpart modify -i 1 -l ZFS-BLONDIE-4-SWAP-WCC2EE134983 mfid4
```

# Swap, Then More Swap

Turning on the first swap partition revealed a practical ceiling:

```
swapon mfid0p1.eli
swapinfo -k
```

64G turned out to be the largest single swap partition FreeBSD would accept in our setup, so instead of one giant partition we resized down and added several 64G partitions per disk:

```
gpart resize -i 1 -s 64G mfid0
gpart recover mfid0
gpart add -s 64G -t freebsd-swap -l ZFS-BLONDIE-0-SWAP-B-WCC2ERZ56512 mfid0
```

...each new partition getting its own `geli init` / `geli attach` pair. We also hit a kernel-level ceiling (`kern.maxswzone`) once total configured swap got too large, and rather than tune the kernel we just backed off from 384GB to 324GB of total swap.

# Building the ZFS Pool

With every disk partitioned, encrypted, and attached, the last step (before things get too deep into jail-specific configuration for one post) is standing up the actual ZFS pool:

```
mkdir /tmp/BLONDIE
zpool create BLONDIE-MAIN raidz2 \
  BLONDIE-4-WD12345/ZFS-BLONDIE-4-WD12345 \
  BLONDIE-5-WD12345/ZFS-BLONDIE-5-WD12345 \
  BLONDIE-6-WD12345/ZFS-BLONDIE-6-WD12345 \
  BLONDIE-7-WD12345/ZFS-BLONDIE-7-WD12345

zpool add BLONDIE-MAIN log mirror \
  BLONDIE-2-WD12345/ZLOG-BLONDIE-2-WD12345 \
  BLONDIE-3-WD12345/ZLOG-BLONDIE-3-WD12345
```

At that point we've got an operating system that boots read-only off a disc into RAM, a set of GELI-encrypted, ZFS-backed drives for swap and logging, and a RAIDZ-2 kit ready to host jails -- all of it built with commodity FreeBSD tooling rather than anything exotic.

# Where This Actually Stands

We want to be straightforward about scope: this chapter (and this post) is a single-instance case study, built and evaluated by the same people who designed it. There's no red-team validation, no non-amnesiac control system to compare against, and no measured recovery-time numbers. The MITRE ATT&CK mapping was done by hand, by us, without a second reviewer -- treat it as illustrative rather than authoritative.

If we come back to this, the next steps are: build a plain (non-amnesiac) FreeBSD box on comparable hardware as a control, run a defined slice of ATT&CK techniques against both under the same conditions, time recovery after a simulated ransomware event on each, and get someone else to sanity-check the technique-to-hypothesis mapping. Until then, consider this a reproducible engineering pattern and threat-model argument -- not proof.
