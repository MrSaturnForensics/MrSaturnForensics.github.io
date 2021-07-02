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
- Add a fail-safe to make sure this cannot overwrite existing case files with the same filename or exhibit filename

### The Code

### Template Files & Locations
I started off by tackling some of the directories for templates and location of cases. 

**TEMPLATE_FOLDER** represents anything which should be copied into the root of the case folder, such as an empty template keyword txt file for use later. 

**CASE_DIRECTORIES** represents any location of which the case folder can be saved to, and the input defined at the start is linked to the directory. 

**KEYWORD_LISTS_DIR** represents the file path to the folder containing all of the template keyword files.

The **r'** means that the string is to be treated as a raw string, which means all escape codes will be ignored.

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



### Safe Character Inputs
This function was necessary to allow any input that contains data that cannot be accepted as part of a filename to be highlighted to the user, suggesting an alternative input, and giving them the option to accept it, any invalid character will be replace with "-". I found a microsoft [document](https://docs.microsoft.com/en-us/windows/win32/fileio/naming-a-file) helpful in writing this.

{% highlight javascript linenos %}
def safe_filename(text, replace_with='-'):
    """
    Replaces invalid characters in given text and returns the escaped text
    and used invalid characters as a tuple.
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


### Case Location Selection
This function will display all of the values defined previously in **CASE_DIRECTORIES**, and will allow an input that matches any that exist in either uppercase or lowercase, if an incorrect input is given, it will keep looping and asking. 

{% highlight javascript linenos %}
def ask_case_type():
    """Ask the user for what letter case for directory purposes."""
    # List of defined case directories in the constant `CASE_DIRECTORIES`
    case_types = CASE_DIRECTORIES.keys()

    print("Type one of the casetypes below and press ENTER.")
    while True:
        # Display all case letters to user
        print(' / '.join(case_types))

        # Ask for case type and return it if its a case type listed in `case_types` list.
        case_type = input()
        if case_type.upper() in case_types:
            return case_type.upper()
        else:
            print("Invalid input, please select again")
{% endhighlight %}

### Case Reference
This function will ask the user to input the case reference for the filename of the root case folder. This makes use of the previously defined **safe_filename** function, and will convert any invalid filename into an alternative or allow the user to re-enter an input.

{% highlight javascript linenos %}
def ask_case_reference():
    """Ask the user for full case reference and change values to work in a filename"""
    while True:
        case_filename = input("Please type case reference such as 'A-123-99-DF' and press ENTER: ")
        escaped, invalids = safe_filename(case_filename)

        if invalids:
            print(f"The input contains the following invalid names: {invalids}")
            print(f'Your input was converted to -> "{escaped}"')
            ans = input("Do you accept the above as the case reference? [Y/N] ")
            if ans not in 'yY\n':
                continue
        return escaped
{% endhighlight %}

### Exhibit Type Selection
This section will ask the user for an input based on the values present in **case_structure**, both containing filepaths to the different directory structures for a phone or computer. The user will input either 'P' or 'C' to select which directory structure is required for an exhibit.

{% highlight javascript linenos %}
def ask_exhibit_type():
    """
    Ask the user for what letter case for directory structure to be copied.
    :returns 'C' for Computer,'P' for Phone.
    """
    while True:
        case_structure = input("Press 'C' for Computer / 'P' for Phone, then press ENTER: ").upper()
        exhibit_directory = EXHIBIT_DIRECTORIES.get(case_structure, False)

        # If the `exhibit_directory` is defined, return the path.
        # Otherwise ask for valid input again.
        if exhibit_directory:
            return exhibit_directory
        else:
            print("Invalid input, please select again")
            # Goes back to start of the loop
{% endhighlight %}

### Exhibit Name & Adding More Exhibits
This function allows the user to provide the filename for the exhibit, which will rename the root exhibit folder and a few template files present within it, This makes use of the previously defined **safe_filename** function, and will convert any invalid filename into an alternative or allow the user to re-enter an input.

{% highlight javascript linenos %}
def ask_one_exhibit():
    while True:
        exhibit = input("Please type exhibit reference and press ENTER: ")

        # safe_filename returns the escaped filename and the invalid
        # characters used in the original:
        escaped_exhibit, invalids = safe_filename(exhibit)

        # If the user gives an invalid character, ask whether to keep the escaped
        # one or to ask for it again.
        if invalids:
            print(f"The input contains the following invalid names: {invalids}")
            print(f'Your input was converted to -> "{escaped_exhibit}"')
            ans = input("Do you accept the above as an exhibit? [Y/N] ")
            if ans not in 'yY\n':
                continue

        return escaped_exhibit
{% endhighlight %}

### Main Function
This is where all the functions are called and assigned. I initally called all of them under some variables to use in **main()**.

I used **.is_dir()** to ensure that if the case folder is already exisiting the process would be halted so no overwriting could occur, this will prompt the user and close the script. 

Following this I used the **distutils.dir_util - copy tree** which copied the files from the source template file path to the root of the case folder. 

**_distutils.dir_util_** is a utility function for manipulating directories and directory trees.

Then the keyword template txt present in the root folder is renamed using an **F-String** with the user's inputs, and the selected contents of the keyword template files is copied into this txt. 

**_F-strings_** are string literals that have an f at the beginning and curly braces containing expressions that will be replaced with their values.

{% highlight javascript linenos %}
def main():
    """Based on what the user selects for "get_case_type" assign value
       to variable case_directory, then append the filename given as the directory to make"""
    # Calling other function variables created to be used here
    case_type = ask_case_type()  # 'C/'
    case_dir = CASE_DIRECTORIES[case_type]  # 'F:/C'
    case_reference = ask_case_reference()  # 'C-123-20-DF'
    case_ref_dir = Path(case_dir) / case_reference  # 'F:/C/C-123-20-DF/'

    if case_ref_dir.is_dir():
        input('This case folder already exists, please re-run the script and '
              'check case reference is accurate.\nPress ENTER to close.'
              'If you are trying to create another exhibit, please run script in case folder')
        print('>')
        sys.exit()

    print(f"Creating Case Directory \"{case_ref_dir}\"")
    # Copy subdirectory from template to main case folder.
    # `copy_tree` creates all directories for dst.
    copy_tree(src=str(TEMPLATE_FOLDER), dst=str(case_ref_dir))

    keyword_file = case_ref_dir.joinpath('Case_Ref_Keywords.txt').rename(
        case_ref_dir / f"{case_reference}_Keywords.txt")
    keyword_file_content = ask_for_keyword_files()
    keyword_file.write_text(keyword_file_content)

    # Copy the keyword file into a CSV file
    shutil.copy(case_ref_dir / f"{case_reference}_Keywords.txt", case_ref_dir / f"{case_reference}_Keywords.csv")

    # Add ".CASE REF_SFR" Folder
    SFR_Folder = case_ref_dir / f".{case_reference}_SFR"
    SFR_Folder.mkdir(parents=True, exist_ok=False)
    print(f"Case Directory created succesfully: \"{case_ref_dir}\".")

    # Loop in which user adds exhibits
    while True:
        exhibit_name = ask_one_exhibit()
        # 'CM-01'

        exhibit_dir_path = case_ref_dir / exhibit_name
        # 'F:/C/C-123-20-DF/CM-01'

        exhibit_template_dir = ask_exhibit_type()
        # 'E:\HTML & CSV output\HTML & CSV output\Computer'

        # Phone
        exhibit_image_path = case_ref_dir / exhibit_name / "Memory Card" / "Image Files"
        # 'F:\C\C-123-20-DF\CM-01\Memory Card\Image Files\CM-01_M1'

        exhibit_sim_path = case_ref_dir / exhibit_name / "SIM"
        # 'F:\C\C-123-20-DF\CM-01\SIM\Exhibit_Ref_S1'

        # Computer
        exhibit_pc_image_path = case_ref_dir / exhibit_name / "Image Files"
        # 'F:\C\C-123-20-DF\CM-01\Memory Card\Image Files\CM-01_M1'

        if exhibit_dir_path.is_dir():
            print('This exhibit exists already. Please re-enter a different exhibit reference.')
            continue    # Back to start of loop

        print("Creating Exhibit directory...")
        copy_tree(src=str(exhibit_template_dir), dst=str(exhibit_dir_path))

        # Rename exhibit files & dirs using its specific case ref and exhibit ref names
        exhibit_dir_path.joinpath('.Case_Ref_Exhibit_Ref_Reports').rename(
            exhibit_dir_path / f".{case_reference}_{exhibit_name}_Reports")
        if exhibit_template_dir == r'K:\#ISO SOFTWARE APPROVED#\#CASE TEMPLATES (DO NOT EDIT)\PHONE': # CHANGE ME AS NEEDED
            exhibit_image_path.joinpath('Exhibit_Ref_M1').rename(
                exhibit_image_path / f"{exhibit_name}_M1")
            exhibit_sim_path.joinpath('Exhibit_Ref_S1').rename(
                exhibit_sim_path / f"{exhibit_name}_S1")
        else:
            exhibit_pc_image_path.joinpath('Exhibit_Ref_H1',).rename(
                exhibit_pc_image_path / f"{exhibit_name}_H1")
            exhibit_pc_image_path.joinpath('Exhibit_Ref_H2',).rename(
                exhibit_pc_image_path / f"{exhibit_name}_H2")
            exhibit_pc_image_path.joinpath('Exhibit_Ref_U1',).rename(
                exhibit_pc_image_path / f"{exhibit_name}_U1")

        ans_add_more = input("Do you want to add another exhibit? [Y/N] ")
        if ans_add_more not in 'yY\n':
            break
main()
{% endhighlight %}

### Final Code
~~~
TEMPLATE_FOLDER = r"K:\#ISO SOFTWARE APPROVED#\#CASE TEMPLATES (DO NOT EDIT)\CASE FOLDER" #CHANGE ME AS NEEDED
CASE_DIRECTORIES = {  # CHANGE ME AS NEEDED
    'C': r'K:\Case Files\C',
    'D': r'K:\Case Files\D',
    'E': r'K:\Case Files\E',
    'F': r'K:\Case Files\F',
    'G': r'K:\Case Files\G',
    'H': r'K:\Case Files\H',
    'N': r'K:\Case Files\N',
    'S': r'K:\Case Files\S',
    'V': r'K:\Case Files\V',
    'W': r'K:\Case Files\W',
}
EXHIBIT_DIRECTORIES = {
    'C': r'K:\#ISO SOFTWARE APPROVED#\#CASE TEMPLATES (DO NOT EDIT)\COMPUTER',  # CHANGE ME AS NEEDED
    'P': r'K:\#ISO SOFTWARE APPROVED#\#CASE TEMPLATES (DO NOT EDIT)\PHONE' # CHANGE ME AS NEEDED
}
KEYWORD_LISTS_DIR = Path(
    r"K:\#ISO SOFTWARE APPROVED#\#KEYWORD LISTS" # CHANGE ME AS NEEDED
)


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
        
        print("1) Cats")
        print("2) Dogs")
        print("3) Birds")
        
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


def ask_case_type():
    """Ask the user for what letter case for directory purposes."""
    # List of defined case directories in the constant `CASE_DIRECTORIES`
    case_types = CASE_DIRECTORIES.keys()

    print("Type one of the casetypes below and press ENTER.")
    while True:
        # Display all case letters to user
        print(' / '.join(case_types))

        # Ask for case type and return it if its a case type listed in `case_types` list.
        case_type = input()
        if case_type.upper() in case_types:
            return case_type.upper()
        else:
            print("Invalid input, please select again")


def ask_case_reference():
    """Ask the user for full case reference and change values to work in a filename"""
    while True:
        case_filename = input("Please type case reference such as 'A-123-99-DF' and press ENTER: ")
        escaped, invalids = safe_filename(case_filename)

        if invalids:
            print(f"The input contains the following invalid names: {invalids}")
            print(f'Your input was converted to -> "{escaped}"')
            ans = input("Do you accept the above as the case reference? [Y/N] ")
            if ans not in 'yY\n':
                continue
        return escaped


def ask_exhibit_type():
    """
    Ask the user for what letter case for directory structure to be copied.
    :returns 'C' for Computer,'P' for Phone.
    """
    while True:
        case_structure = input("Press 'C' for Computer / 'P' for Phone, then press ENTER: ").upper()

        # If user inputs something that isn't defined in `exhibit_directories`
        # then `exhibit_directory` becomes False like defined below.
        exhibit_directory = EXHIBIT_DIRECTORIES.get(case_structure, False)

        # If the `exhibit_directory` is defined, return the path.
        # Otherwise ask for valid input again.
        if exhibit_directory:
            return exhibit_directory
        else:
            print("Invalid input, please select again")
            # Goes back to start of the loop


def ask_one_exhibit():
    while True:
        exhibit = input("Please type exhibit reference and press ENTER: ")

        # safe_filename returns the escaped filename and the invalid
        # characters used in the original:
        escaped_exhibit, invalids = safe_filename(exhibit)

        # If the user gives an invalid character, ask whether to keep the escaped
        # one or to ask for it again.
        if invalids:
            print(f"The input contains the following invalid names: {invalids}")
            print(f'Your input was converted to -> "{escaped_exhibit}"')
            ans = input("Do you accept the above as an exhibit? [Y/N] ")
            if ans not in 'yY\n':
                continue

        return escaped_exhibit


def main():
    """Based on what the user selects for "get_case_type" assign value
       to variable case_directory, then append the filename given as the directory to make"""
    # Calling other function variables created to be used here
    case_type = ask_case_type()  # 'C/'
    case_dir = CASE_DIRECTORIES[case_type]  # 'F:/C'
    case_reference = ask_case_reference()  # 'C-123-20-DF'
    case_ref_dir = Path(case_dir) / case_reference  # 'F:/C/C-123-20-DF/'

    if case_ref_dir.is_dir():
        input('This case folder already exists, please re-run the script and '
              'check case reference is accurate.\nPress ENTER to close.'
              'If you are trying to create another exhibit, please run script in case folder')
        print('>')
        sys.exit()

    print(f"Creating Case Directory \"{case_ref_dir}\"")
    # Copy subdirectory from template to main case folder.
    # `copy_tree` creates all directories for dst.
    copy_tree(src=str(TEMPLATE_FOLDER), dst=str(case_ref_dir))

    keyword_file = case_ref_dir.joinpath('Case_Ref_Keywords.txt').rename(
        case_ref_dir / f"{case_reference}_Keywords.txt")
    keyword_file_content = ask_for_keyword_files()
    keyword_file.write_text(keyword_file_content)

    # Copy the keyword file into a CSV file
    shutil.copy(case_ref_dir / f"{case_reference}_Keywords.txt", case_ref_dir / f"{case_reference}_Keywords.csv")

    # Add ".CASE REF_SFR" Folder
    SFR_Folder = case_ref_dir / f".{case_reference}_SFR"
    SFR_Folder.mkdir(parents=True, exist_ok=False)
    print(f"Case Directory created succesfully: \"{case_ref_dir}\".")

    # Loop in which user adds exhibits
    while True:
        exhibit_name = ask_one_exhibit()
        # 'CM-01'

        exhibit_dir_path = case_ref_dir / exhibit_name
        # 'F:/C/C-123-20-DF/CM-01'

        exhibit_template_dir = ask_exhibit_type()
        # 'E:\HTML & CSV output\HTML & CSV output\Computer'

        # Phone
        exhibit_image_path = case_ref_dir / exhibit_name / "Memory Card" / "Image Files"
        # 'F:\C\C-123-20-DF\CM-01\Memory Card\Image Files\CM-01_M1'

        exhibit_sim_path = case_ref_dir / exhibit_name / "SIM"
        # 'F:\C\C-123-20-DF\CM-01\SIM\Exhibit_Ref_S1'

        # Computer
        exhibit_pc_image_path = case_ref_dir / exhibit_name / "Image Files"
        # 'F:\C\C-123-20-DF\CM-01\Memory Card\Image Files\CM-01_M1'

        if exhibit_dir_path.is_dir():
            print('This exhibit exists already. Please re-enter a different exhibit reference.')
            continue    # Back to start of loop

        print("Creating Exhibit directory...")
        copy_tree(src=str(exhibit_template_dir), dst=str(exhibit_dir_path))

        # Rename exhibit files & dirs using its specific case ref and exhibit ref names
        exhibit_dir_path.joinpath('.Case_Ref_Exhibit_Ref_Reports').rename(
            exhibit_dir_path / f".{case_reference}_{exhibit_name}_Reports")
        if exhibit_template_dir == r'K:\#ISO SOFTWARE APPROVED#\#CASE TEMPLATES (DO NOT EDIT)\PHONE': 
            exhibit_image_path.joinpath('Exhibit_Ref_M1').rename(
                exhibit_image_path / f"{exhibit_name}_M1")
            exhibit_sim_path.joinpath('Exhibit_Ref_S1').rename(
                exhibit_sim_path / f"{exhibit_name}_S1")
        else:
            exhibit_pc_image_path.joinpath('Exhibit_Ref_H1',).rename(
                exhibit_pc_image_path / f"{exhibit_name}_H1")
            exhibit_pc_image_path.joinpath('Exhibit_Ref_H2',).rename(
                exhibit_pc_image_path / f"{exhibit_name}_H2")
            exhibit_pc_image_path.joinpath('Exhibit_Ref_U1',).rename(
                exhibit_pc_image_path / f"{exhibit_name}_U1")

        ans_add_more = input("Do you want to add another exhibit? [Y/N] ")
        if ans_add_more not in 'yY\n':
            break
main()
~~~
