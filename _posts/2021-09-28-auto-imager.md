---
layout: post
title: Automatic Imaging Script **WIP**
subtitle: Using FTKImager and Tableau Imager
cover-img: /assets/img/computer-screen-monitor-text.jpg
thumbnail-img: /assets/img/ftk.png
share-img: /assets/img/computer-screen-monitor-text.jpg
tags: [Python, Scripting, Automation]
---

I wanted to see how easy automation would be within Digital Forensics and what level of automatic processing can be achieved, so I looked into a repetitive task carried out on a regular basis in cases, Imaging.

### The Background
Within my current and previous workplaces, imaging has been done through the use of **FTKImager** which has been seen as a standard tool, with FTK being mainly used to image memory card's, USB's and hard drives. The process of imaging tends to be very repetitive, with little change in the options selected, because of this I felt as though potentially automating the imaging process would be ideal.

### The Aim
I had considered how far this could potentially go, and what requirements would be needed to allow for a robust script:
- Detect a new device connected after the script is ran, acknowledging something new is connected for imaging
- Support imaging for memory card's, USB's, mechanical hard drives and SSD's
- Carry out checks for HPO and DCA upon imaging, and warn the user
- Carry out checks for hash errors or bad sectors, and warn the user
- Automatically fill in case details from a given directory path to get case data
- Create a log file of all user input's and the processes done
- Alert the user processing has finished

### The Code
{% highlight javascript linenos %}
root = Tk()
root.eval('tk::PlaceWindow . center')
root.withdraw()

### Directory Path
def get_save_directory():
    """Ask user for the directory where image files will be saved to."""
    while True:
        working_directory = Path.cwd().absolute()

        save_directory = Path(
            filedialog.askdirectory(
                initialdir=str(working_directory),
                title="Save under Exhibit Imaging Folder"
            )
        )
        # Check to make sure 2 directories up is called "Image Files"
        image_files = Path(save_directory)
        image_file_check = image_files.parents[0].stem

        if not image_file_check == "Image Files":
            tkinter.messagebox.showerror(title="Incorrect Location Selected", message="Error, please select specific exhibit imaging folder",) 
        else:            
            break
    
    if not save_directory:
        sys.exit()  # Exit if cancel is pressed
    else:
        return Path(save_directory)
        
save_dir = get_save_directory()
{% endhighlight %}
