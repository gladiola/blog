---
title:  "blackmagic"
date:   9999-06-04 08:30:00
description: gladiola/blackmagic repo
---

Our repo is up at: <a href="https://github.com/gladiola/blackmagic">gladiola/blackmagic</a>.

# Demo Programs
Listed below are some selected programs from the repo, by language or system.

### Program [JPO_02APR2018A](https://github.com/gladiola/blackmagic/tree/Demo/ASP.NETCore2.0/JPO_02APR2018A)
Selected files from an ASP.NET Core 2.0 program. References to Microsoft tutorials and reference docs are included in the source code.

#### Features
Main effect achieved here was to allow a scaffolded project to include:
- QR code 2 Factor Authentication that works with Microsoft Authenticator
- Login with Facebook
- Login with Twitter
- Identity with password standards
- Requires HTTPS
- Redirects HTTP to HTTPS
- Use of UserSecrets in development for holding critical values
[//]: # ()

### Program [SystemCheck.ps1](https://github.com/gladiola/blackmagic/blob/Demo/PowerShell/CPSC5700_Benchmarking/SystemCheck.ps1)
Abbreviated version of system checks used with benchmarking tests. Presented here as a simple time-saving utility for gathering data on a machine. Gathers data on the machine executing the script and places output in a text file.

#### Lists details about:
- Computer manufacturer and model
- Processor
- BIOS
- Operating sytem
- Current processes
[//]: # ()

### Program [greenXterm.sh](https://github.com/gladiola/blackmagic/blob/Demo/FreeBSD/greenXterm.sh)
Simple shell script to make green and black terminal windows in X11.

### Program [portListC.sh](https://github.com/gladiola/blackmagic/blob/Demo/FreeBSD/portListC.sh)
Shell program to install packages. Package names are placed into an array in shell and then iterated over. Inside the loop, the shell script will apply the port name to the package command. Used with FreeBSD 10.3 on a system that had a poudriere jail. This particular port list was based off a poudriere-determined list of port dependencies; the original list of ports wanted was much smaller.

Lists of ports like this, without the dependencies, were originally generated using shell scripts like discoverPorts.sh.

Having shell scripts like these was a helpful part of standing up a poudriere jail.