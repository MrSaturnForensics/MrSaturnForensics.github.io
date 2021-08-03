---
layout: post
title: Automatic Imaging Script **WIP**
subtitle: Using FTKImager and Tableau Imager
cover-img: /assets/img/computer-screen-monitor-text.jpg
thumbnail-img: /assets/img/ftk.png
share-img: /assets/img/computer-screen-monitor-text.jpg
streamableId: hy4157
tags: [Python, Scripting, Automation]
---



I wanted to see how easy automation would be within Digital Forensics and what level of automatic processing can be achieved, so I looked into a repetitive task carried out on a regular basis in cases, Imaging.

### Video Demonstration

{% include streamablePlayer.html id=page.streamableId %}

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

### Exhibit Type

This function will store the value of what the user inputs, this will be used for a different filename, and a potential different path for imaging that exhibit type, it will loop infinitely until the user selects an input displayed.

{% highlight javascript linenos %}
def get_source_type():
    """Ask the user for the type of what is being imaged."""

    while True:
        print("Press 'M' for Memory Card, 'U' for USB, 'H' for Hard Drive")
        logging.info("Press 'M' for Memory Card, 'U' for USB, 'H' for Hard Drive")
        exhibit_type = input()
        logging.info(exhibit_type)

        # Memory Card Processing Starts
        if exhibit_type in 'mM':
            return 'M'

        # USB Processing Starts
        elif exhibit_type in 'uU':
            return 'U'

        # Hard Drive Processing Starts
        elif exhibit_type in 'hH':
            return 'H'
        

        # User did not input any of these, loop again until they do
        else:
            print("Invalid input, please select again")
            logging.info("Invalid input, please select again")
{% endhighlight %}

### Fixing a limitation of PyAutoGUI!

The following function is used to swap between the data types depending on the state of the CAPS LOCK being ON or OFF. 
This was necessary as **PyAutoGUI** was putting the input's into text fields in either upper case or lowercase, using this function ensures that no matter the state of CAPS LOCK, the input will always be in capital letters. This will be used by appending **fix_case** to the variable.

{% highlight javascript linenos %}
def fix_case(to_swap):
    """
    Swaps case of letters in pathlib path or in a string IF CAPSLOCK IS ON
    Output type stays as Pathlib object if input was a Pathlib object.
    Otherwise returns a string.
    """
    if GetKeyState(VK_CAPITAL) == 1:
        if isinstance(to_swap, Path):
            return Path(str(to_swap).swapcase())
        else:
            return str(to_swap).swapcase()
    return to_swap
 {% endhighlight %}

### Filenames, Directories and Capital Letters Fix

I then created a class which would deal with the possible outputs for filenames and file paths depending on the **exhibit_type**, depending on the input, it will build a filename based on a different set of paths, as the Computer path has one less directory between the case reference and image location.

This is also where the option for an SSD image is, if selected it gives the user a more frequent warning to turn off the connection as soon as imaging completes as SSD's use trimming and garbage collection, which can potentially delete cached data.

This is also where the **fix_case** function is applied to the variables that contain the strings to be written into FTKImager.

A _**Class**_  is a way of grouping functions.

{% highlight javascript linenos %}
class Filenames:
    """
    A class that stores filename parts and full filenames and filepaths
    Variable :case_ref: can be accessed through :Pathstore.case_ref:
    and :save_dir: through :Pathstore.save_dir:, for example.
    """
    DFO_name = input("Please Enter DFO Name: ")
    logging.info(DFO_name)
    source_type = get_source_type()
    exhibit_type = source_type

    case_ref = save_dir.parents[3].name  # Case Reference from dir name
    exhibit_ref = save_dir.parents[2].name  # Exhibit Reference from dir name

    # Changing for "PC" case folder structure, as it differs
    if exhibit_type == "H" or exhibit_type == "U":
        case_ref = save_dir.parents[2].name  # Case Reference from dir name
        exhibit_ref = save_dir.parents[1].name  # Exhibit Reference from dir name
        
    # Ask if its a hard drive or SSD if "H" is selected then store this for use later
    if exhibit_type == "H":
        while True:
            print("Press 'H' for Hard Drive or 'S' for SSD")
            logging.info("Press 'H' for Hard Drive or 'S' for SSD")
            global pc_type
            pc_type = input()
            logging.info(pc_type)

            # Store as hard drive
            if pc_type == "h" or pc_type == "H":
                break

            # Store as SSD
            elif pc_type == "s" or pc_type == "S":
                break

            # User did not input any of these, loop again until they do
            else:
                print("Invalid input, please select again")
                logging.info("Invalid input, please select again")
                
        
    # Check for phone structure, if not change memory card filename for PC structure
    image_files = Path(save_dir)
    phone_structure_check = image_files.parents[1].stem    
    if not phone_structure_check == "Memory Card":
        case_ref = save_dir.parents[2].name  # Case Reference from dir name
        exhibit_ref = save_dir.parents[1].name  # Exhibit Reference from dir name    
         
    # Find first unreserved suffix for new reports
    suffix = 1
    _log_filename = f"{case_ref}_{exhibit_ref}_{source_type}{{suffix}}.E01.txt"
    while True:
        FTK_log_filename = Path(_log_filename.format(suffix=suffix))
        if save_dir.joinpath(FTK_log_filename).exists():  # Add one to suffix and test again
            suffix += 1
        else:  # Unreserved log filename is now stored in the variable :log_filename:
            break

    FTK_filename = Path(f"{case_ref}_{exhibit_ref}_{source_type}{suffix}")
    FTK_filename_txt = Path(f"{case_ref}_{exhibit_ref}_{source_type}{suffix}.E01.txt")
    FTK_filepath = save_dir / FTK_filename
    FTK_filepath_txt = save_dir / FTK_filename_txt

    Tableau_filename = Path(f"{case_ref}_{exhibit_ref}_{source_type}{suffix}_Tableau.txt")
    Tableau_filepath = save_dir / Tableau_filename

    exhibit_and_suffix = f"{exhibit_ref}_{source_type}{suffix}"

    # Pyautogui types strings using capslock.
    # If capslock is on, the typed letters case is inverted.
    # When giving a string for pyautogui to write, the string should be case
    # swapped if capslock is on. Function :fix_case: will do this automatically.
    case_ref_capsfix = fix_case(case_ref)
    exhibit_and_suffix_capsfix = fix_case(exhibit_and_suffix)
    DFO_name_capsfix = fix_case(DFO_name)
    FTK_filename_capsfix = fix_case(FTK_filename)
    Tableau_filename_capsfix = fix_case(Tableau_filename)
    save_dir_capsfix = fix_case(save_dir)
  {% endhighlight %}

### Navigating Tableau Imager and FTKImager
This is the bit I cannot show as it's very specific to my current workplace, however the idea behind it is **PyAutoGUI** provides the ability to use hotkeys and shortcuts, using a combination of these and some screen searching methods, This navigates the software! 

**If you would like more specific details around this please contact me and I can help**.


### HPA & DCO Check

To test for HPA & DCO being present on a drive was a fairly straightforward process, **Tableau Imager** will include this within the .txt file created when the device details are looked at, so before **FTKImager** is opened, the .txt is checked to see if **HPA in Use: Yes', 'DCO in Use: Yes** is present inside the file, this was done using the **regex** module, using a pattern search of that string, if it is found - the script will stop and warn the examiner of the issue.

The _**regex**_ module lets you check if a particular string matches a given regular expression.

A _**regular expression**_ (shortened as "regex") are special strings representing a pattern to be matched in a search operation. 

{% highlight javascript linenos %}
hard_drive_patterns = 'HPA in Use: Yes', 'DCO in Use: Yes'

with open(str(Filenames.Tableau_filepath)) as file:
    HPA_DCO_check = file.read()

for pattern in hard_drive_patterns:
    # If match, issue exit out program and warn DFO on screen
    # Terminate program / place a warning in folder
    if re.search(pattern, HPA_DCO_check):
        open('HPA OR DCO ACTIVE.txt', 'w').close()
        print('HPA OR DCO IN USE PLEASE EXAMINE MANUALLY')
        logging.info('HPA OR DCO IN USE PLEASE EXAMINE MANUALLY')
        input()
        sys.exit()

    else:  # Carry on if no issues
        print()

print('HPA / DCO not in use -  No issues')
logging.info('HPA / DCO not in use -  No issues')
{% endhighlight %}
