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

### The Aim
I had considered how far this could potentially go, and what requirements would be needed to allow for a robust script:
- Rename image files within the directory in the structure **'{case_reference}_{exhibit_reference} (number)'**
- Only rename .JPG files 
- Add a fail-safe to make sure this can only be ran inside a "Photographs" folder, so it cannot rename files by mistake!

### The Code
