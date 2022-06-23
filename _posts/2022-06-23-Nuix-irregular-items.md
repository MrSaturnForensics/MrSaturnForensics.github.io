---
layout: post
title: Nuix Irregular Item Script
cover-img: /assets/img/computer-screen-monitor-text.jpg
thumbnail-img: /assets/img/nuix-logo-grey-1024x566-1702408471.PNG
share-img: /assets/img/computer-screen-monitor-text.jpg
tags: [NUIX, Scripting, Perl]
---


Irregular Items are a standard part of any ingestion into a NUIX case, these items present issues and require further action or consideration by the examiner. 
The idea of this script is to conveniently provide all of these items under the tagged section in NUIX for review, this could be used both during the processing stage as
well as during a peer review process in order to verify all Irregular items have been dealt with sufficiently.

### The Aim
I had considered what would be ideal for this script:
- Break down each Irregular type into a Sub-tag under a Main "QA" tag
- Account for all Irregular type(s) in order to make this work on all types of data
- Make the script lightweight so that on a large dataset, processing time will not be impacted heavily

### The Code
