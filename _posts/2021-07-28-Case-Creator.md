---
layout: post
title: Case Creation Script
cover-img: /assets/img/computer-screen-monitor-text.jpg
thumbnail-img: /assets/img/Folder.PNG
share-img: /assets/img/computer-screen-monitor-text.jpg
tags: [Python, Scripting, Automation]
---

Creating a case file is something done on a very regular basis with Digital Forensics, I wondered how easily this process could be automated and streamlined, so I had a go at writing a script for the entire process!


Within my current / previous workplaces, the case creation has normally involved just manually copying a template folder or running a batch script and changing what is necessary, as well as creating each set of folders for each exhibit type manually. My goal would be to make this as user friendly as possible, creating all of the files needed and structure without having to copy in or run multiple scripts, while also offering the ability to add some template keywords and some fail-safes to stop any overwritting of data.

### The Aim
I had considered how far this could potentially go, and what requirements would be needed to allow for a robust script:
- Select where the case file is saved from numerous defined locations
- Select if the exhibit is a Phone or Computer with different templates depending on exhibit type
- Option to add more than one exhibit per run.
- Add keywords from existing templates
- Handle inputs correctly and offer alternatives rather than crashing the program
- Add a fail-safe to make sure this cannot overwrite existing case files with the same filename / exhibit filename
