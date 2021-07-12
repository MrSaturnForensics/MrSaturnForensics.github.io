---
layout: post
title: Password Cracking With Hashcat
subtitle: Cracking MD5 Hashes & iOS iTunes Encrypted Backups
cover-img: /assets/img/computer-screen-monitor-text.jpg
thumbnail-img: /assets/img/hashcat.PNG
share-img: /assets/img/computer-screen-monitor-text.jpg
tags: [Hashcat, Password Cracking]
---



### iTunes Encrypted Backup

The keys for the iTunes encrypted backup can be found in a file called **Manifest.plist** and are stored in a container called a **Keybag**. The iOS backup format looks like this:

_The **Keybag** contains a number of encryption 'class keys' that ultimately protect files in the file system._

**iOS => 10:**
_$itunes_backup$*<iOS version[10]>*<WPKY>*<ITERATIONS [10MILLION]>*<SALT>*<DPIC>*<DPSL>_
  
**iOS 9:**
_$itunes_backup$*< iOS version[9]>*<WPKY>*< ITERATIONS [1K]>*<SALT>**_

The problem with **iOS 10 backups and above** is that it adds an extra layer of protection using **PBKDF2 with 10 million iterations**. This massively slows down password cracking time.

_**PBKDF2** is a simple cryptographic key derivation function, which is resistant to dictionary attacks and rainbow table attacks._

