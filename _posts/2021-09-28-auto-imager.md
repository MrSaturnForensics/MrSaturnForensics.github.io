---
layout: post
title: Automatic Imaging Script **WIP**
subtitle: Using FTKImager and Tableau Imager
cover-img: /assets/img/computer-screen-monitor-text.jpg
thumbnail-img: /assets/img/ftk.png
share-img: /assets/img/computer-screen-monitor-text.jpg
tags: [Python, Scripting, Automation]
---

I wanted to see how easy automation would be within Digital Forensics and what level of automatic processing can be achieved., so I looked into a repetitive task carried out on a regular basis, Imaging.

### The Background
Within my current and previous workplaces, the case creation has normally involved just manually copying a template folder or running a batch script and changing what is necessary, as well as creating each set of folders for each exhibit type manually. My goal would be to make this as user friendly as possible, creating all of the files needed and structure without having to copy in or run multiple scripts, while also offering the ability to add some template keywords and some fail-safes to stop any overwritting of data.

### The Aim
I had considered how far this could potentially go, and what requirements would be needed to allow for a robust script:
- Select where the case file is saved from numerous defined locations

