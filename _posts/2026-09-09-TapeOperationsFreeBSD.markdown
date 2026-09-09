---
title:  "Basic Tape Operations with FreeBSD"
date:   2026-09-09 09:00:00
description: Learning LTO tape backups from scratch on FreeBSD -- mt, tar, and the mistakes that taught us to respect the write-once nature of tape.
---

We wanted to see if we could make use of an LTO tape drive that came bundled with one of our salvaged computers. We'd heard plenty about tape's reputation for backups, and we'd seen enough war stories about the bravery required to touch old mainframe tape scripts. What we hadn't seen were many solid tutorials or comprehensive directions for actually doing tape operations ourselves. Almost nobody uses tape as part of personal computing anymore -- the businesses that still use it mostly do so as part of larger commercial data center operations. For most of us, the closest thing to tape operations was cassette tapes for storage in the 1980s. LTO tape gets talked about a lot and used rarely, so we wanted to break through the mystery.

# The Plan

Nothing fancy: write a file to tape, eject the tape, and read the file back. Once that works, do a simple sequence of writes and reads without damaging anything already on the tape.

# The Hardware

We used a Dell T610 tower with an onboard LTO3 single-deck tape machine, along with some LTO3 tapes to test with. Tape drivers and the common command-line tools are already built into FreeBSD, so there was nothing extra to install.

One early warning worth calling out: use the `nsa` driver, not `sa`. There is an `sa` driver, but it has real limitations -- when we tried it, we weren't able to properly advance down the tape through multiple file markers. `nsa` is the one to reach for.

# Code Notes

1. Put the files you want backed up into a directory you control first. Isolating them from wherever they normally live is safer than pointing tar directly at scattered locations.

2. Load the tape:
   ```
   mt -f /dev/nsa0 load
   ```

3. Rewind it to the beginning:
   ```
   mt -f /dev/nsa0 rewind
   ```

4. Check status to see where we are on the tape:
   ```
   mt -f /dev/nsa0 status
   ```
   If this is the first use of the tape, we can just start writing.

5. If there are already files on the tape, we need to either scan ahead and log what's there, or consult our own notes about where to go next. It's easy to overwrite an existing file and damage the sequence, so use the write-protect notch whenever possible.

6. Skip to the end of the existing records:
   ```
   mt -f /dev/nsa0 eod
   ```

7. Archive files once we're at the right spot on the tape:
   ```
   tar cvf /dev/nsa0 [FILE_TO_ARCHIVE]
   ```

8. Write an end-of-file marker:
   ```
   mt -f /dev/nsa0 weof 1
   ```

9. List what's in the archive on tape:
   ```
   tar -tf /dev/nsa0
   ```
   Or pull it back off:
   ```
   tar xf /dev/nsa0
   ```

10. Write notes. Log what was put on the tape and where.

11. For the next record, rewind and step forward through the file markers until reaching the next blank space to write.

# A Scripted Demo

To make the whole sequence rehearsable, we wrote a little interactive shell script that walks through loading, rewinding, checking status, and then hunting for three different files on the tape by name and position, pausing at each step for us to read the output before continuing:

```sh
#!/usr/bin/env sh
clear
echo "scriptedDemo"
echo "Press ENTER to continue."
read INPUT

## Check tape
echo "Check if tape is staged in machine."
echo "Press ENTER to continue."
read INPUT
clear

## Load the tape into the machine
echo "Here we go."
echo
echo "Loading tape into machine."
echo "mt -f /dev/nsa0 load"
mt -f /dev/nsa0 load

## Rewind tape
echo "Rewinding tape"
echo "mt -f /dev/nsa0 rewind"
mt -f /dev/nsa0 rewind
echo "Press ENTER to continue."
read INPUT
clear

echo "Reading status from tape."
echo "mt -f /dev/nsa0 status"
mt -f /dev/nsa0 status
echo
echo "Press ENTER to continue."
read INPUT
clear

echo "Looking for file 0 on the tape."
echo "We should be at the beginning."
echo "So, no mt commands this time."
echo "tar t -f /dev/nsa0"
tar t -f /dev/nsa0
echo
echo "Press ENTER to continue."
read INPUT
clear

echo "Looking for file 1 on tape."
echo "Rewind to the beginning."
mt -f /dev/nsa0 rewind
echo "Go forward space count 1 file."
echo "mt -f /dev/nsa0 fsf 1"
mt -f /dev/nsa0 fsf 1
echo "tar t -f /dev/nsa0"
tar t -f /dev/nsa0
echo "Press ENTER to continue."
read INPUT
clear

echo "Looking for file 2 on tape."
echo "Back up to 0 and go forward space count 2 files."
echo "mt -f /dev/nsa0 rewind"
mt -f /dev/nsa0 rewind
echo "mt -f /dev/nsa0 fsf 2"
mt -f /dev/nsa0 fsf 2
echo "Show where we are with status."
echo "mt -f /dev/nsa0 status"
mt -f /dev/nsa0 status
echo
echo "Look in that spot with tar."
echo "tar t -f /dev/nsa0"
tar t -f /dev/nsa0
echo "Press ENTER to continue."
read INPUT
clear

echo "Looking for third archive on tape."
echo "Back up to 0 and go forward space count 3 files."
echo "mt -f /dev/nsa0 rewind"
mt -f /dev/nsa0 rewind
echo "mt -f /dev/nsa0 fsf 3"
mt -f /dev/nsa0 fsf 3
echo "See where we are with status."
echo "mt -f /dev/nsa0 status"
mt -f /dev/nsa0 status
echo
echo "See what is here with tar."
echo "tar t -f /dev/nsa0"
tar t -f /dev/nsa0
echo "Press ENTER to continue."
read INPUT
clear

echo "Take the tape offline."
echo "Rewind tape."
echo "mt -f /dev/nsa0 rewind"
mt -f /dev/nsa0 rewind
echo "mt -f /dev/nsa0 offline"
mt -f /dev/nsa0 offline
exit 0
```

The five sample files we used for testing were nothing exotic -- just short text files (`test.txt` through `test5.txt`) with sentences like "The quick brown fox jumped over the lazy dogs" and "The fox is just really fast," enough to confirm we were reading back exactly what we wrote, in the right order, at the right file marker.

# What We Learned

**Tape sequencing is fragile.** Unlike random access memory or even a spinning disk, overwriting on tape can damage end-of-file markers, which are the only navigational delimiters usable on the medium. In practice, it was very easy to accidentally overwrite a tape marker, and every subsequent file on the tape could become inaccessible as a result.

**Logging matters more than it sounds like it should.** With potentially many files stacked on a long tape, keeping a log and tracking file positions isn't optional busywork -- skipping it makes every subsequent use of the tape more hazardous.

**Encryption is on you.** Unlike a GELI-encrypted hard drive, the tape itself offers no protection scheme beyond whatever encryption we apply to the files ourselves. Tar has compression built in, but encryption is a separate step, and since backups may sit in storage for a long time, encrypting them before they ever hit tape is worth treating as a required step rather than a nice-to-have. That encryption would need to happen before the `tar` step above, not after.

# Testing Before It Matters

Writing to tape should be rehearsed, not learned live during a real recovery. Checking status and reading files back to confirm our location on the tape is worth doing every time. Any write operation should be backed by a rehearsed read script, and files being written should live in their own isolated directory first. Add an encryption step if the data warrants it, rehearse the tar and archive commands, and generally compartmentalize each step of the plan -- the goal is reducing the odds of a catastrophic, unrecoverable mistake.

# Why Bother With Tape At All

Tape has been recommended as a preferred backup medium for a long time, and off-site backup offers real advantages over anything living entirely on-premises. We've lost count of how many times backups get recommended in the abstract, yet practical tutorials and directions remain surprisingly rare.

Tape and hard drive backups can both be routed off-site fairly easily and securely through the mail -- mailed to yourself at a personal mailbox, a remote office, a P.O. box, or to someone you trust. It's worth remembering that anyone entrusted with that data also takes on some amount of risk or liability by holding it.

# References

- FreeBSD `mt` man page: [man.freebsd.org/cgi/man.cgi?query=mt](https://man.freebsd.org/cgi/man.cgi?query=mt)
- FreeBSD `tar` man page: [man.freebsd.org/cgi/man.cgi?query=tar](https://man.freebsd.org/cgi/man.cgi?query=tar)
- As of our research, the FreeBSD Handbook didn't have dedicated coverage of tape operations, so this was mostly worked out from the man pages and hands-on testing.
