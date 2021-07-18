---
layout: post
title: Password Cracking With Hashcat
subtitle: Cracking BitLocker & iOS iTunes Encrypted Backups
cover-img: /assets/img/computer-screen-monitor-text.jpg
thumbnail-img: /assets/img/hashcat.PNG
share-img: /assets/img/computer-screen-monitor-text.jpg
tags: [Hashcat, Password Cracking]
---

Encryption and password locked application's are quite common to see in an investigation, and it is quite commonly dismissed as something that cannot be bypassed or accessed, I wanted to look at how some of most common encryption methods can be cracked using a password recovery tool.

### Password Hashes

Hashing is a one-way function to scramble data — it takes readable text and transforms it into a completely different string of characters with a set length.

However, unlike other encryption algorithms that transform data, hashing is difficult to revert, making cracking a time consuming process.

<a href="https://ibb.co/fG8qXdf"><img src="https://i.ibb.co/4gsKFRz/image.png" alt="image" border="0" /></a>

The biggest problem with password hashing is that if you run a specific word like 'green' through a hashing algorithm, the hashed outcome for that word will always be the same. 
Meaning you could in theory guess millions of passwords, run them through the same algorithm, and then see what the hash for a specific password is.

If you recover the hash of a password your trying to crack, using this method and comparing this against the hash will allow you to try find a matching hash, giving you the password.

<a href="https://imgbb.com/"><img src="https://i.ibb.co/6YJ7Rbk/image.png" alt="image" border="0" /></a>

**Dictionary** and **Brute-Force** attacks are the most common ways of guessing passwords. These techniques make use of a file that contains words, phrases, common passwords and other strings that are likely to be used as a viable password. 

_**Dictionary attacks**_ work by trying a list of words in a dictionary, often previously used passwords, often from lists obtained from past security breaches._

_**Brute-Force attacks**_ work by trial-and-error to find the correct password, working through all possible combinations hoping to guess correctly. **This is only suggested for a numeric password**

### Cracking BitLocker

**If the device was encrypted using the User Password authentication method this will be possible.**

**If using a smart code PIN to unlock the Bitlocker container, this method will ONLY return $2 and $3 hashes which are the recovery keys, and are far too complex to crack meaning it is highly unlikely to crack - (48 digits long).**

Initially I started by enabling BitLocker on a USB with a basic **.txt** file.

<a href="https://ibb.co/KVXrqHK"><img src="https://i.ibb.co/BTtCZRn/Capture.png" alt="Capture" border="0" /></a>

I then set it up using the User Password authentication method, with a password of **'Saturn_1'**

<a href="https://imgbb.com/"><img src="https://i.ibb.co/n0khmgr/Capture1.png" alt="Capture1" border="0" /></a>

I then dismounted the USB, and imaged the USB creating a **RAW DD** file of the USB's contents in **FTKImager**.

_A RAW file is an exact copy of the original data on a device._

<a href="https://ibb.co/gtCxjhx"><img src="https://i.ibb.co/CVFf1Df/Capture2.png" alt="Capture2" border="0" /></a>


   **bitlocker2john** should then print the following hashes:
   
   <a href="https://ibb.co/ncSyPd4"><img src="https://i.ibb.co/hWvNYQr/Capture3.png" alt="Capture3" border="0" /></a>
   
   **$bitlocker$0$**... : User Password fast attack mode
   
   **$bitlocker$1$**... : User Password attack mode with MAC verification (slower execution, no false positives)
   
   **$bitlocker$2$**... : Recovery Password fast attack mode
   
   **$bitlocker$3$**... : Recovery Password attack mode with MAC verification (slower execution, no false positives)

### Hashcat Breakdown
_If you ever need a full breakdown of hashcat attacks/attack modes you can always type ‘hashcat.exe --help’ in the command line while hashcat is open._

**Hash type [-m]**
Used to select the hash you want to attack. There are a lot of hashes supported by hashcat.  

A full list can be found in the link above or by simply typing **‘hashcat64.exe --help’** into the command line of hashcat. 

**Attack Modes [-a]**

| Attack Number   | Attack Type     | 
| --------------- | --------------- | 
| 0 | Straight                |
| 1 | Combination             |
| 3 | Brute-force + Mask      |
| 6 | Hybrid Wordlist + Mask  |
| 7 | Hybrid Mask + Wordlist  |

**Rule based attack [-r]**
A rule based attack takes every word in a **dictionary** file, then it will try variations of every word based on the rules you set. This means for every word in the dictionary, it will try 64 different variations of the word.

a _**dictionary**_ is a list of cracked passwords from previous data dumps of websites.

Be aware, some of the rule files contain thousands of rules and the time to crack a password will be unrealistic, unless you are cracking a terrible hash algorithm such as MD5.

**Recommended Rules**
Based on efficiency, the **‘Best64’** rule is always the best rule to start with. Followed by **‘InsidePro-PasswordsPro’**. 

From there, it really depends on the type of hash you are cracking, if it is a weak hash algorithm like MD5. Then use the **'OneRuleToRuleThemAll’** rule. 

**Using multiple dictionaries**
You can also use multiple dictionaries in one attack. After a dictionary, simply add another dictionary name to the command.

**Combination Attack [-a 1]**
This works by appending the words of two dictionaries, for example if one dictionary contains the word ‘test’ and the second dictionary contains the word ‘123’ it will append one word to the other to make ‘test123’.

**Mask attack (Advanced Brute Force) [-a 3]**
Mask attacks take advantage of setting customised attacks by using charsets. This method is far quicker than simply trying to brute force a password. The charsets can be found below:

| Charsets  | Data Type     | 
| --------- | ------------- | 
| l - abcdefghijklmnopqrstuvwxyz  | [Lowercase letters]   |
| u - ABCDEFGHIJKLMNOPQRSTUVWXYZ  | [Uppercase letters]   |
| d - 0123456789 	                | [Numbers]             |
| h - 0123456789abcdef 			      | [Lowercase Hex values]|
| H - 0123456789ABCDEF 			      | [Uppercase Hex values]|
| a - ?l?u?d?s                    | [Combination of the above, excluding hex]|
| b - 0x00 - 0xff	                | [Raw bytes]           |
  
These are used by entering ‘?’ Followed by the charset you desire. If you wanted all numbers with a password length of 5: 

**-a 3 ?d?d?d?d?d**

You can also set custom charsets in each space by using -1 -2 -3 etc:

**-a 3 -1 ?u?l?d   ?1?1?1?1?1**

The above will try a combination of all uppercase letters, lowercase letters and numbers in each of those 5 spaces. 1 is set to contain ?u?l?d and is used in every space.
You can go further than this.

**-a 3 -1 ?u?l -2 ?l?d ?1?2?2?2?2?2 -i --increment-min=5 --increment-max=8**

This takes advantage of two custom charsets -1 and -2 is you can REALLY narrow down cracking time. This assumes the first letter is either an uppercase or lowercase letter with the rest being either lowercase letters or numbers. What this also does is start as a 5 letter password, and increments up to an 8 letter password.

**Hybrid dictionary and mask attack [-a 6]**
Want to try every word in the English dictionary followed by 4 numbers? This is common due to humans being noobs/predictable and using words followed by a year. You can use a hybrid attack consisting of a dictionary and a mask.

**-a 6 OxfordEnglish.txt ?d?d?d?d**

Hybrid mask and dictionary attack [-a 7]
As above however, the mask will come before the dictionary.

**-a 7 ?d?d?d?d OxfordEnglish.txt** 

Recommended Dictionaries
Here is a list of the recommended dictionaries to use on iOS 9 backups (smallest to largest):
⦁	Rockyou.txt
⦁	Realhuman_phill.txt
⦁	Realuniq.lst
⦁	Nummer_DB.top
⦁	HashesOrg
⦁	DCHTPassv1.0.txt
⦁	Weakpass_2a

Recommended Dictionaries
Here is a list of the recommended dictionaries to use on iOS 10 backups (smallest to largest):
⦁	Rockyou.txt
⦁	Realhuman_phill.txt


### iTunes Encrypted Backup

The keys for the iTunes encrypted backup can be found in a file called **Manifest.plist** and are stored in a container called a **Keybag**. The iOS backup format looks like this:

_The **Keybag** contains a number of encryption 'class keys' that ultimately protect files in the file system._

**iOS => 10:**
_$itunes_backup$*<iOS version[10]>*<WPKY>*<ITERATIONS [10MILLION]>*<SALT>*<DPIC>*<DPSL>_
  
**iOS 9:**
_$itunes_backup$*< iOS version[9]>*<WPKY>*< ITERATIONS [1K]>*<SALT>**_

The problem with **iOS 10 backups and above** is that it adds an extra layer of protection using **PBKDF2 with 10 million iterations**. This massively slows down password cracking time.

_**PBKDF2** is a cryptographic key derivation function, which is resistant to dictionary attacks and rainbow table attacks._

*Due to the iOS 10+ backup encryption method being so strong, do not use any advanced attack modes on it, as it will just take too long. Stick to the rockyou.txt and realhuman_phill.txt dictionaries.
