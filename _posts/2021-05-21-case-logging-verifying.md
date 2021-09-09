---
layout: post
title: Case Logging & Case Verifying Script 
cover-img: /assets/img/computer-screen-monitor-text.jpg
thumbnail-img: /assets/img/python.png
share-img: /assets/img/computer-screen-monitor-text.jpg
tags: [Python, Scripting, Automation, Validation]
---

Logging information on cases within Digital Forensics is important, ensuring as little tampering as possible of a case file after completion is part of the process of ensuring the integrity of evidence. 

Verifying information from a log in the case is also just as important, so I did my best to achieve both with a script.

### The Background
My intention with this script was to create a way of verifying if a case has been edited or changed in any form after completion, and create an audit trail of every file present from the root of a case directory. 

This could then be used in conjunction with another script I plan on creating which will then compare two log files and highlight any differences between them, which could then be used as part of a peer review process to ensure integrity between the case file at different times.

### The Aim
I had considered how far this could potentially go, and what requirements would be needed to allow for robust scripts:

**Case Logging Script**
- Log all files within directories from the location of the script
- Pull Filename, Last Accessed Time and Directory Path
- Hyperlink the Directory Path, meaning someone can click on it to jump to the file
- Present this information in a **.xlsx** file

**Case Verifying Script**
- Compare two outputs of **Case Logger**
- Determine deleted, modified or created data from comparing the outputs

### Case Logging Script Breakdown

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

### Last Accessed Time

I then wrote a function to obtain the last access time of a file, this would convert it into a readable timestamp, and handle an incorrect or missing date.

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

### Navigating Folders

I then wrote another function which would obtain all files within a directory the script is ran from, and go down every subfolder, this was achieved using **os.walk**.

I also used a module called **tqdm** to provide a progress bar in the command prompt to show the progress of this action.

There is also the ability to handle **UTF-8** files, decoding them so the information can be read. This will obtain the filename, last accessed time and directory path and save them to variables.

_**os.walk**_ is part of the **os** module, it will follow each directory recursively until no further sub-directories are available from the initial directory that walk was called upon.

_**tqdm**_ is a extensible progress bar for Python and CLI.

_**UTF-8**_ is a variable-width character encoding.

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

### Hyperlinks

Below is another function, which will allow entries to be put into a hyperlink formula automatically. If the source filepath is over 255 characaters long, it will not hyperlink as this is over the limit in excel for a hyperlink.

{% highlight javascript linenos %}
def to_excel_hyperlink(source: str, espacefunc=escape):
    """Puts a source in excel hyperlink formula."""
    escaped = espacefunc(source)
    return source if len(escaped) > 255 else f'=HYPERLINK("{escape(source)}")'
{% endhighlight %}


### Column Widths

Following this, I then created a function which would take the length of the max entry in each column and adjust the size accordingly to the biggest entry.

_**Pandas**_ is a data analysis and manipulation module.

_**Dataframe**_ is a 2-dimensional labeled data structure with columns of potentially different types. You can think of it like a spreadsheet or SQL table.

{% highlight javascript linenos %}
def calculate_col_widths(dataframe):
    """Calculate the width for each column in dataframe."""
    col_widths = {}
    for column_name, value in COLUMN_NAMES.items():
        if isinstance(value, int):
            width = value
        else:
            # Find max length of a value in current column
            longest_val: pd.Series = dataframe[column_name].astype(str).str.len().max()
            # Max len of any value including col header
            width = max(longest_val + 4, len(str(column_name))) + 2
        col_widths[column_name] = width
    return col_widths
{% endhighlight %}

### Worksheets

Using the **math** module, i then created a function which will work out how many worksheets the data may need to be spread over.

_**Math** module provides access to the mathematical functions defined by the C standard.

{% highlight javascript linenos %}
def get_worksheet_count(dataframe, chunk_size):
    total_rows = dataframe.shape[0]
    worksheet_count = math.ceil(total_rows / chunk_size)
    return worksheet_count
{% endhighlight %}

I then created a function which would yield the dataframe view with `EXCEL_ROW_LIMIT` number of rows so that the amount will always fit one excel sheet.

{% highlight javascript linenos %}
def split_dataframe(dataframe, chunk_size) -> pd.DataFrame:
    worksheet_count = get_worksheet_count(dataframe, chunk_size)
    for sheet_i in range(worksheet_count):
        start_row = sheet_i * chunk_size
        end_row = (sheet_i + 1) * chunk_size - sheet_i
        chunk = dataframe.iloc[start_row:end_row]
        yield chunk
{% endhighlight %}

### Converting the dataframe to Excel



As shown below, the script when running locally is able to reach **12,180** files processed a second!

<a href="https://imgbb.com/"><img src="https://i.ibb.co/txyhQSG/ss-2021-07-23-at-09-08-41.png" alt="ss-2021-07-23-at-09-08-41" border="0" /></a>
