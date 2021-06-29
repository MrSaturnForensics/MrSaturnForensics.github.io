---
layout: post
title: Automating Repetition - File Renaming
cover-img: /assets/img/computer-screen-monitor-text.jpg
thumbnail-img: /assets/img/files photo.PNG
share-img: /assets/img/computer-screen-monitor-text.jpg
tags: [Python, Scripting, Automation]
---

### Every Day Task
Some of the work we carry out within Digital Forensics is often repetitive, one of the every day tasks is renaming photographs taken of an exhibit as it's recieved, signed and triaged.

As a start to this blog, I intended to make my life slightly easier, via creating a simple Python script which would allow the renaming of files based on the parent directories present within a folder structure. In laymans terms, case structures tend to include both the case reference and exhibit reference, with the Photographs being included within this folder chain. Based on this I can assume a structure for a a filename of a photograph, and create a script which can fill in this information. 

### The Aim
I had considered how far this could potentially go, and what requirements would be needed to allow for a robust script:
- Rename image files within the directory in the structure **'{case_reference}_{exhibit_reference} (number)'**
- Only rename .JPG files 
- Add a fail-safe to make sure this can only be ran inside a "Photographs" folder, so it cannot rename files by mistake!

### The Code
Initially I tackled the fail-safe aim via the use of a module called **Tkinter**, in this situation, I was looking for a constant between all case files, I had found that 4 directories up was always **Case Files**, so it can be assumed if this is found then the script is being ran at the correct location, and if not it should not run. I had defined the initial directory as the **Path.cwd()** this is just the path in which the script is ran in. I then used **parent.stem** to obtain the folder name 4 directories up.

{% highlight javascript linenos %}
working_dir_check = Path.cwd()
working_dir_check_structure = working_dir_check.parents[3].stem
if not working_dir_check_structure == "Case Files":
   tkinter.messagebox.showerror(title="Incorrect Location Selected", message="Error, please ONLY try run in Photographs folder!",)
   quit()
{% endhighlight %}
