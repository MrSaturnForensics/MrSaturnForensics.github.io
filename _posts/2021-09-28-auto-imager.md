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

### Directory Path
This function will ask the user for the file path for the image files, it has a built in check to verify that the folder directory above is called "Image Files", prompting the user to select the right area, if not it displays an error and asks the user to select again. This uses the **tkinter** module to create an interface for the user to interact with.

_**tkinter**_ is the standard Python interface to the Tk GUI toolkit.

{% highlight javascript linenos %}
root = Tk()
root.eval('tk::PlaceWindow . center')
root.withdraw()

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

### Automation Log File & Verifying a new device

The following lines then build the filename of an automation log which will keep track of the input's present in the command prompt for audit trail purposes.
It appends the current date and time into the filename so it can be seen exactly when the script was ran, it then appends each line and input the user put's into the .txt file.

The way to deal with media being connected with no file system is through the use of a module called **win32com** this allows me to check on the amount of devices present inside "Device Manager" on the computer the script is ran on. This is stored under the variable **wmi.InstancesOf("Win32_USBHub)**.

I then carried out a **Len()** check on this, once a new device has been connected, the amount should increase by 1, giving us a way to determine if something new is recognised. I could then build in functionality to get the user to check if the amount has not increased.

_**win32com**_ is a module which provides access to many of the Windows APIs from Python.

_**len()**_ function returns the number of items in an object.


{% highlight javascript linenos %}
# Get the current date and time
now = datetime.datetime.now()

LOG_FILE_NAME = "Automation log_" + str(now.strftime("%d-%m-%Y_%H-%M-%S") + ".txt")

# Create and start a log file
logging.basicConfig(filename=save_dir / LOG_FILE_NAME,
                    format='%(asctime)s %(message)s',
                    level=logging.DEBUG)
logging.info('** PROCESSING STARTED **')

# Checking Device Manager for devices, this will help us determine if the
# connected media has a file system. Entries will be counted and compared
# to allow for if a new one is found

# Checking using a length check and prompting user to connect new device
wmi = win32com.client.GetObject("winmgmts:")
amount_of_devices = len(wmi.InstancesOf("Win32_USBHub"))
amount_of_devices = int(amount_of_devices)
amount_verification = input("Please connect device for imaging then press ENTER")
print("Finding new device connected...")

# Give time for it to be recognised, and compare if entry amount changed
time.sleep(20)
amount_of_devices_after = len(wmi.InstancesOf("Win32_USBHub"))
amount_of_devices_after = int(amount_of_devices_after)

if amount_of_devices >= amount_of_devices_after:

    # If amount is equal, no new drive has been detected
    print("No new device found, please check it is inserted correctly then press ENTER")
    logging.info("No new device found, please check it is inserted correctly then press ENTER")

    # If drive isnt detected after this time, give it longer and check again
    input()
    time.sleep(15)
    amount_of_devices_after = len(wmi.InstancesOf("Win32_USBHub"))
    amount_of_devices_after = int(amount_of_devices_after)

    if amount_of_devices >= amount_of_devices_after:
        print("Device still not found, review if file system is present / picked up on file explorer")
        print("If the device is showing in file explorer, disconnect and restart script")

        logging.info("Device still not found, review if file system is present / picked up on file explorer")
        logging.info("If the device is showing in file explorer, disconnect and restart script")
        logging.info('** PROCESSING UNSUCCESSFUL / UNABLE TO DETECT DEVICE **')
        input()
        quit()

    else:
        print("Device Found - No issues")
        logging.info("Device Found - No issues")
        print()

else:
    print("Device Found - No issues")
    logging.info("Device Found - No issues")
    print()

{% endhighlight %}
    
