---
layout: post
title: Case Logger Script 
cover-img: /assets/img/computer-screen-monitor-text.jpg
thumbnail-img: /assets/img/logger.PNG
share-img: /assets/img/computer-screen-monitor-text.jpg
tags: [Python, Scripting, Automation, Validation]
---

Logging information on cases within Digital Forensics is important, ensuring as little tampering as possible of a case file after completion is part of the process of ensuring the integrity of evidence.

### The Background
My intention with this script was to create a way of verifying if a case has been edited or changed in any form after completion, and create an audit trail of every file present from the root of a case directory. This could then be used in conjunction with another script I plan on creating which will then compare two log files and highlight any differences between them, which could then be used as part of a peer review process to ensure integrity between the case file at different times.

### The Aim
I had considered how far this could potentially go, and what requirements would be needed to allow for a robust script:
- Select where the case file is saved from numerous defined locations


### The Code
