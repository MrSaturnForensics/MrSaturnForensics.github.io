---
layout: post
title: Case Creation Script
cover-img: /assets/img/computer-screen-monitor-text.jpg
thumbnail-img: /assets/img/Folder.PNG
share-img: /assets/img/computer-screen-monitor-text.jpg
tags: [Python, Scripting, Automation]
---

Creating a case file is something done on a very regular basis with Digital Forensics, I wondered how easily this process could be automated and streamlined, so I had a go at writing a script for the entire process!

### The Background
Within my current and previous workplaces, the case creation has normally involved just manually copying a template folder or running a batch script and changing what is necessary, as well as creating each set of folders for each exhibit type manually. My goal would be to make this as user friendly as possible, creating all of the files needed and structure without having to copy in or run multiple scripts, while also offering the ability to add some template keywords and some fail-safes to stop any overwritting of data.

### The Aim
I had considered how far this could potentially go, and what requirements would be needed to allow for a robust script:
- Select where the case file is saved from numerous defined locations
- Select if the exhibit is a Phone or Computer with different templates depending on exhibit type
- Option to add more than one exhibit per run.
- Add keywords from existing templates
- Handle inputs correctly and offer alternatives rather than crashing the program
- Add a fail-safe to make sure this cannot overwrite existing case files with the same filename / exhibit filename

### The Code

### Template Files & Locations
I started off by tackling some of the directories for templates and location of cases. 

**TEMPLATE_FOLDER** represents anything which should be copied into the root of the case folder, such as an empty template keyword txt file for use later. 

**CASE_DIRECTORIES** represents any location of which the case folder can be saved to, and the input defined at the start is linked to the directory. 

**KEYWORD_LISTS_DIR** represents the file path to the folder containing all of the template keyword files.

{% highlight javascript linenos %}
TEMPLATE_FOLDER = r"C:\Template" 
CASE_DIRECTORIES = {   
    'C': r'C:\Case Files\C',
    'D': r'C:\Case Files\D',
    'E': r'C:\Case Files\E',
    'F': r'C:\Case Files\F',
    'G': r'C:\Case Files\G',
    'H': r'C:\Case Files\H',
    'N': r'C:\Case Files\N',
    'S': r'C:\Case Files\S',
    'V': r'C:\Case Files\V',
    'W': r'C:\Case Files\W',
}
EXHIBIT_DIRECTORIES = {
    'C': r'C:\COMPUTER',  
    'P': r'C:\PHONE' 
}
KEYWORD_LISTS_DIR = Path(
    r"C:\KEYWORD LISTS" 
)
{% endhighlight %}

The rest of the code I have included with Functions which I then plan on building into a final main() function. 

**_Function_** is a block of code which only runs when it's called.

### Keyword Selection
This section which would deal with the keywords for the case folder, I initially used a sorted **glob** search for any file within **KEYWORD_LISTS_DIR** that is a **.txt**, this is then placed into a new variable **keyword_list_files**. 

**_Glob module_** finds all the pathnames matching a specified pattern according to the rules used by the Unix shell.

Following this, I added a **len** check, which checks the amount of files found in **keyword_list_files**, if this returns 0 this means no keyword files have been found, which will terminate the program and provide an error.

**_len()_** function returns the number of items in an object.

{% highlight javascript linenos %}
def ask_for_keyword_files():
    """Returns a string that has all the contents of the files that the user selected."""
    # List of all txt files present within directory
    keyword_list_files = sorted(KEYWORD_LISTS_DIR.glob(r'*.txt'))
    
    if len(keyword_list_files) == 0:
        print(f"Couldn't find keyword list files: \"{KEYWORD_LISTS_DIR}\"")
        input('Press enter to exit.')
        sys.exit()
{% endhighlight %}

After this I used **print** to provide text instructions on the screen of how to select which keyword file is present, as well as hard coding the names of the txt files, making it easier for the user to see which value represents which keyword file. 

**_print()_** function prints the specified message to the screen.

The user is then instructed to input the numbers representing each keyword file they need, this input allows for single or multiple keyword files to be included. The input is then striped of "**,**" to allow for multiple files inputs to be understood.

{% highlight javascript linenos %}
    # Print a list of files to the user to select from
    print("Please select one or more keyword lists for examination by\n"
          "giving one or more numbers from the list below.\n"
          'Examples: "1,2,3" or "5"')
    print()

    # Get the index of the file the user wants to use
    while True:
        print("0) No Keyword List Required")
        
        print("1) Cats")
        print("2) Dogs")
        print("3) Birds")
        keyword_answer = input('> ')
        selections_strings = keyword_answer.split(',')
{% endhighlight %}

The input from this is then stripped of any spaces, and tested to see if it's an integer. If this fails then it loops for the user to input again and displays an error message.

After this it will then also verify if the input matches the amount of files found within the keyword folder, and will only allow an input integer that is equal or less.
I also added in the functionality to not include any keywords, of which the user can skip with "0".
{% highlight javascript linenos %}
        # Try to convert input list to python integers:
        selections = []
        for string in selections_strings:
            # strip in case user entered spaces
            string = string.strip()

            # Test if user gave a valid integer
            try:
                integer = int(string)
            except ValueError:
                print(f"This input only accepts numbers, \"{string}\" is not a number! Please try again!")
                print()
                break  # Back to start

            # No keyword list required, escape out
            if keyword_answer == "0":
                continue

            # Test that the integer points to a file:
            if not (1 <= integer <= len(keyword_list_files)):
                print(f"This keyword list does not exist, \"{integer}\" Please only select from the list shown!")
                print()
                break  # Back to start

            # Integer was valid
            selections.append(integer)
        else:
            # Whole input list was processed successfully!
            break
{% endhighlight %}

Finally this returns a variable which contains all of the keywords selected by the user, which will be used later to append to the empty keyword file. 

{% highlight javascript linenos %}
    # Create a variable that will contain all the keyword files selected
    concatenated_output = ''
    for selection_int in selections:
        # user gave non-zero indexed values, so -1
        selected_file_content = keyword_list_files[selection_int - 1].read_text()
        concatenated_output += selected_file_content + '\n'

    return concatenated_output
 {% endhighlight %}



### Safe Inputs

{% highlight javascript linenos %}
def safe_filename(text, replace_with='-'):
    """
    Replaces invalid characters in given text and returns the escaped text
    and used invalid characters as a tuple.

    Reference: https://docs.microsoft.com/en-us/windows/win32/fileio/naming-a-file
    """
    invalid = [
        *r'< > : " / \ | ? *',
        *r'CON PRN AUX NUL COM1 COM2 COM3 COM4 COM5 COM6 COM7 COM8 COM9 LPT1 LPT2 LPT3 LPT4 LPT5 '
         r'LPT6 LPT7 LPT8 LPT9'.split(),
        # Characters whose integer representations are in the range from 1 through 31 are not
        # allowed.
        *[chr(n) for n in range(0, 32)]
    ]

    invalids = []
    for char in invalid:
        if char in text:
            text = text.replace(char, replace_with)
            invalids.append(char)
    return text.strip(), invalids
    
{% endhighlight %}
