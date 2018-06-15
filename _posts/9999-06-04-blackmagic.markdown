---
title:  "blackmagic"
date:   9999-06-04 08:30:00
description: gladiola/blackmagic repo
---

Our repo is up at: <a href="https://github.com/gladiola/blackmagic">gladiola/blackmagic</a>.

# Demo Programs
Listed below are some selected programs from the repo, by language or system.

## Folder [ASP.NET Core 2.0](https://github.com/gladiola/blackmagic/tree/Demo/ASP.NETCore2.0)

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


## Folder [PowerShell](https://github.com/gladiola/blackmagic/tree/Demo/PowerShell)
### Program [SystemCheck.ps1](https://github.com/gladiola/blackmagic/blob/Demo/PowerShell/CPSC5700_Benchmarking/SystemCheck.ps1)
Abbreviated version of system checks used with benchmarking tests. Presented here as a simple time-saving utility for gathering data on a machine. Gathers data on the machine executing the script and places output in a text file.

#### Lists details about:
- Computer manufacturer and model
- Processor
- BIOS
- Operating sytem
- Current processes

### Program [JPO_runBenchmarks.ps1.txt](https://github.com/gladiola/blackmagic/blob/Demo/PowerShell/CPSC5700_Benchmarking/JPO_runBenchmarks.ps1.txt)
Runs a SystemCheck.ps1 code segment, and then executes some benchmark programs that would be expected in a nearby folder. Those benchmark programs are not included here.

#### Demonstrates:
- automated creation of files and directories
- automated creation of dynamic names of files based on time
- executing benchmark programs in batch
- routing output to specific files

### Program [JPO_pullKeyDataItemsFromBenchmarksB.ps1.txt](https://github.com/gladiola/blackmagic/blob/Demo/PowerShell/CPSC5700_Benchmarking/JPO_pullKeyDataItemsFromBenchmarksB.ps1.txt)
Program to navigate to benchmark output text file; extract critical data points from file; hold varying amounts of data, discovered before and after a pattern match, in an object; aggregate data about a given benchmark test; deposit summary in another file.

#### Demonstrates:
- object creation and property value assignment
- looping
- pattern searching a string with context