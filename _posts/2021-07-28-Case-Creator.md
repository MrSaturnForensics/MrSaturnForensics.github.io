---
layout: post
title: Case Creation Script
cover-img: /assets/img/computer-screen-monitor-text.jpg
thumbnail-img: /assets/img/Folder.PNG
share-img: /assets/img/computer-screen-monitor-text.jpg
tags: [Python, Scripting, Automation]
---

Creating a case file is something done on a very regular basis with Digital Forensics, I wondered how easily this process could be automated and streamlined, so I had a go at writing a script for the entire process!


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


{% highlight javascript linenos %}
def ask_for_keyword_files():
    """Returns a string that has all the contents of the files that the user selected."""
    # List of all txt files present within directory
    keyword_list_files = sorted(KEYWORD_LISTS_DIR.glob(r'*.txt'))

    # Notify if no files
    if len(keyword_list_files) == 0:
        print(f"Couldn't find keyword list files: \"{KEYWORD_LISTS_DIR}\"")
        input('Press enter to exit.')
        sys.exit()

    # Print a list of files to the user to select from
    # Also preview their content a little bit?
    print("Please select one or more keyword lists for examination by\n"
          "giving one or more numbers from the list below.\n"
          'Examples: "1,2,3" or "5"')
    print()

    for file_index in range(len(keyword_list_files)):
        preview = keyword_list_files[file_index].read_text()[:45]

    # Get the index of the file the user wants to use
    while True:
        print("0) No Keyword List Required")
        
        print("1) Adult Sexual Assault")
        print("2) Darknet")
        print("3) Drugs")
        print("4) Extreme Pornography")
        print("5) Fraud")
        print("6) Grooming")
        print("8) Misper - Suicide")
        print("9) Violence - Weapons")
        print("10) Voyeurism")
        keyword_answer = input('> ')
        selections_strings = keyword_answer.split(',')

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
                # Given integer didn't point to a file
                print(f"This keyword list does not exist, \"{integer}\" Please only select from the list shown!")
                print()
                break  # Back to start

            # Integer was valid
            selections.append(integer)
        else:
            # Whole input list was processed successfully!
            break

    # Create a variable that will contain all the keyword files selected
    concatenated_output = ''
    for selection_int in selections:
        # user gave non-zero indexed values, so -1
        selected_file_content = keyword_list_files[selection_int - 1].read_text()
        concatenated_output += selected_file_content + '\n'

    return concatenated_output
{% endhighlight %}
