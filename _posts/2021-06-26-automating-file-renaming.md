---
layout: post
title: Automating Repetition - File Renaming
cover-img: /assets/img/computer-screen-monitor-text.jpg
thumbnail-img: /assets/img/files photo.PNG
share-img: /assets/img/computer-screen-monitor-text.jpg
tags: [Python, Scripting, Automation]
---

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

I then moved onto building in where the script can pull the details from to build the filenames. In a similar way, I had defined **working_dir** as the path the script is ran from. I then created some variables containing the folder names of the case reference / exhibit reference, finally I then assigned the filename structure using an F string containing the assigned varaibles, which would now be **photograph_filename**.

{% highlight javascript linenos %}
working_dir = Path.cwd()
case_ref = working_dir.parents[1].name  
exhibit_ref = working_dir.parents[0].name 

photograph_filename = f"{case_ref}_{exhibit_ref}"
{% endhighlight %}

### Building it further
I then decided to do the brunt of this in a function called **main()**, the idea being that for every file this process would loop over, meaning it can build unique filenames, I set **i = 1** as the initial count, which would increase value by 1 every loop. this would then later be built into **my_dest** for the final filename. I then used **glob** to only obtain **.jpg**, as well as using **natsort,  os_sorted** to structure these files in a "natural" way, i.e the way you would see it in file explorer. This would then be appied to every file present within the directory that is a .jpg - finally this was then all called at the end of the script with the **main()** function. 

{% highlight javascript linenos %}
def main():
   i = 1
   for current_path in os_sorted(working_dir.glob("*.jpg")):
      my_dest = current_path.with_name(f'{photograph_filename} ({i}).JPG')
      current_path.rename(my_dest)
      
      i += 1
      
if __name__ == '__main__':
   main()
{% endhighlight %}

### The Result
<a href="https://ibb.co/PzxYv0Z"><img src="https://i.ibb.co/x8hstdS/before.png" alt="before" border="0" /></a>


### Final Code
{% highlight javascript linenos %}
# Check to make sure 5 directories up is called "Case Files" or do not run.
working_dir_check = Path.cwd()
working_dir_check_structure = working_dir_check.parents[3].stem
if not working_dir_check_structure == "Case Files":
   tkinter.messagebox.showerror(title="Incorrect Location Selected", message="Error, please ONLY try run in Photographs folder!",)
   quit()

# Specify that the current folder is the directory to look up from
working_dir = Path.cwd()
case_ref = working_dir.parents[1].name  # Case Reference from photo dir
exhibit_ref = working_dir.parents[0].name # Exhibit Reference from photo dir

# Building Filename data that needs to be pulled
photograph_filename = f"{case_ref}_{exhibit_ref}"

# Function to rename multiple files
def main():
   i = 1
   # Only change .jpg files / os.sorted keeps file explorer like order
   for current_path in os_sorted(working_dir.glob("*.jpg")):
      # Build full filename
      my_dest = current_path.with_name(f'{photograph_filename} ({i}).JPG')
      # Rename all files
      current_path.rename(my_dest)

      # Itterate by 1 for each file present
      i += 1
      
if __name__ == '__main__':
   
   # Calling main() function
   main()
{% endhighlight %}
