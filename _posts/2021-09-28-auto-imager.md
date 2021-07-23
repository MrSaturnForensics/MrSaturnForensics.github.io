---
layout: post
title: Automatic Imaging Script **WIP**
subtitle: Using FTKImager and Tableau Imager
cover-img: /assets/img/computer-screen-monitor-text.jpg
thumbnail-img: /assets/img/ftk.png
share-img: /assets/img/computer-screen-monitor-text.jpg
tags: [Python, Scripting, Automation]
---

Inital text

As a start to this blog, I intended to make my life slightly easier, via creating a simple Python script which would allow the renaming of files based on the parent directories present within a folder structure. In laymans terms, case structures tend to include both the case reference and exhibit reference, with the Photographs being included within this folder chain. Based on this I can assume a structure for a a filename of a photograph, and create a script which can fill in this information. 

### The Aim
I had considered how far this could potentially go, and what requirements would be needed to allow for a robust script:
- Rename image files within the directory in the structure **'{case_reference}_{exhibit_reference} (number)'**
- Only rename .JPG files 
- Add a fail-safe to make sure this can only be ran inside a "Photographs" folder, so it cannot rename files by mistake!

### The Code
