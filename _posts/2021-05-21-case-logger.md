---
layout: post
title: Case Logging & Case Verifying Script 
cover-img: /assets/img/computer-screen-monitor-text.jpg
thumbnail-img: /assets/img/python.png
share-img: /assets/img/computer-screen-monitor-text.jpg
tags: [Python, Scripting, Automation, Validation]
---

Logging information on cases within Digital Forensics is important, ensuring as little tampering as possible of a case file after completion is part of the process of ensuring the integrity of evidence. Verifying information from a log in the case is also just as important, so I did my best to achieve both with a script.

### The Background
My intention with this script was to create a way of verifying if a case has been edited or changed in any form after completion, and create an audit trail of every file present from the root of a case directory. 

This could then be used in conjunction with another script I plan on creating which will then compare two log files and highlight any differences between them, which could then be used as part of a peer review process to ensure integrity between the case file at different times.

### The Aim
I had considered how far this could potentially go, and what requirements would be needed to allow for robust scripts:

**Case Logger Script**
- Log all files within directories from the location of the script
- Pull Filename, Last Accessed Time and Directory Path
- Hyperlink the Directory Path, meaning someone can click on it to jump to the file
- Present this information in a **.xlsx** file

**Case Verify Script**
- Compare two outputs of **Case Logger**
- Determine deleted, modified or created data from comparing the outputs

### Case Logger Script

### The Code

Initially I started by using the **logging** module, this would provide context in the command prompt and telling the user what is being processed. 
**DFO_NAME** uses the **getpass** module, this pulls the name of the PC user, which will be used in a filename in **EXCEL_FILENAME**
This uses the **datetime** module to pull the current time, and append it to the filename. 

Following this, I set values for the width of the columns, as well as two hard limits for row's and hyperlinks per sheet, this would mean if the amount of entries exceed these values, it will know to carry over to a new sheet.

_**logging**_ module defines functions and classes which implement a flexible event logging system for applications and libraries. 

_**getpass**_ module returns the “login name” of the user.

_**datetime**_ module supplies classes for manipulating dates and times.

{% highlight javascript linenos %}
logging.getLogger().setLevel(logging.INFO)

INPUT_DIRECTORY = os.getcwd()  # Can be changed
DFO_NAME = getpass.getuser() 
EXCEL_FILENAME = f"{DFO_NAME}_Case File Index_{datetime.now()
.strftime('%d-%m-%Y_%H-%M-%S')}.xlsx"

# Dict with column names (in order) and column widths. "None" means the
# width will be the max length of a value in that column
COLUMN_NAMES = {
    'Filename': 60,
    'Last Accessed Time': None,
    'Full Path': 145,
}

EXCEL_ROW_LIMIT = 1_048_576
HYPERLINK_LIMIT = 65_530
{% endhighlight %}

I then wrote a function to obtain the last access time of a file, this would convert it into a readable timestamp, and handle an incorrect / missing date.

{% highlight javascript linenos %}
def file_last_accessed(filepath):
    """Get the last time a file was accessed as a python datetime object"""
    timestamp = os.path.getmtime(filepath)
    try:
        date = datetime.fromtimestamp(timestamp)
        date = date.strftime("%d/%m/%Y, %H:%M:%S")
    except OSError:
        date = datetime(1970, 1, 1) + timedelta(seconds=timestamp)
    return date
{% endhighlight %}

{% highlight javascript linenos %}
def list_files(directory, last_access=file_last_accessed):
    """Recursively list all files in given directory and it's subdirectories."""
    logging.info("Starting file search...")
    progressbar = tqdm.tqdm(desc="Listing files", unit=' files')
    for currentpath, folders, files in os.walk(directory):
        for file_basename in files:
            # Line to try convert any UTF8 entries found.
            file_basename = file_basename.encode('utf8', 'replace').decode('utf8')
            file_path = f"{currentpath}{os.sep}{file_basename}"

            try:
                last_accessed = last_access(file_path)
            except Exception as err:
                last_accessed = err

            yield file_basename, last_accessed, file_path
            progressbar.update()
    progressbar.close()
{% endhighlight %}
