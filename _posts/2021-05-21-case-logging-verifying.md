---
layout: post
title: Case Logging & Case Verifying Script **WIP**
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

_**Math**_ module provides access to the mathematical functions defined by the C standard.

{% highlight javascript linenos %}
def get_worksheet_count(dataframe, chunk_size):
    total_rows = dataframe.shape[0]
    worksheet_count = math.ceil(total_rows / chunk_size)
    return worksheet_count
{% endhighlight %}

I then created a function which would yield the dataframe view with **EXCEL_ROW_LIMIT** number of rows so that the amount will always fit one excel sheet.

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

The following is then the majority of all the work coming together! First off the filename is built, and a workaround for any filenames beginning with '=' has been used, as if not then excel will assume this represents the beginning of a formula. This then writes all of the data stored in the dataframe to an excel sheet using the **pyexelrate** module. This also provides alot of the formatting and allocation of how the data is presented. 

_**pyexelrate**_ is a module for writing Excel-compatible XLSX spreadsheet files, with an emphasis on speed.

{% highlight javascript linenos %}
ef dataframe_to_excel(dataframe,
                       filename=EXCEL_FILENAME,
                       sheet_name_template='{n}-Case Folder Log',
                       topleft_corner=(1, 1)):
    
    # Mark all filenames that start with a `=` as a string formula like `="file"`
    dataframe['Filename'] = dataframe.loc[:, 'Filename'].apply(lambda s: f'="{s}"' if s.startswith('=') else s)
    
    """
    Write given DataFrame to Excel file using pyexelerate.

    Coordinates for dataframe in excel:
    topleft_corner=(y, x)
    X and Y are non-zero indexed, so first top left cell is (1, 1)
    """
    workbook = Workbook()

    col_headers = dataframe.columns.tolist()
    col_widths = calculate_col_widths(dataframe)
    even_row_style = Color(226, 239, 218, 255)
    col_header_style = Style(
        font=Font(bold=True),
        fill=Fill(background=Color(112, 173, 71)),
        alignment=Alignment(horizontal='center', vertical='center')
    )

    # column headers coordinates
    header_row_left = (topleft_corner[0], topleft_corner[1])
    header_row_right = (topleft_corner[0], topleft_corner[1] + len(col_headers))

    # Row and column to start writing the dataframe to (in the worksheet)
    start_row = topleft_corner[0] + 1  # +1 to overwrite header row
    start_col = topleft_corner[1]

    # Chunk size is excel row limit minus header row index
    chunk_size = EXCEL_ROW_LIMIT - start_row

    dataframe_chunks = split_dataframe(dataframe, chunk_size)
    worksheet_count = get_worksheet_count(dataframe, chunk_size)

    # Write the data to sheets
    logging.info(f"A total of {worksheet_count} worksheets will be written")
    logging.info("Starting to write data...")
    for worksheet_i, chunk in enumerate(dataframe_chunks, 1):
        chunk: pd.DataFrame

        sheet_name = sheet_name_template.format(n=worksheet_i)
        worksheet = workbook.new_sheet(sheet_name=sheet_name)
        # consistently
        col_count = chunk.shape[1]
        row_count = chunk.shape[0]

        # Make hyperlinks out of first `EXCEL_HYPERLINK_LIMIT` full paths
        chunk.update(chunk.loc[:HYPERLINK_LIMIT, 'Full Path'].apply(to_excel_hyperlink, 1))

        # Write dataframe column headers to current worksheet
        worksheet.range(header_row_left, header_row_right).value = [[*col_headers]]
        # Set column width and column header style
        for column_i, column_name in enumerate(dataframe.columns, start_col):
            worksheet.set_col_style(column_i, Style(size=col_widths[column_name]))
            worksheet.set_cell_style(topleft_corner[1], column_i, col_header_style)

        # Write actual dataframe_chunk data to worksheet
        logging.info(f"Writing {row_count} rows of data to sheet -> {sheet_name}")
        values_to_write = chunk.values.tolist()
        data_top_left = (start_row, start_col)
        data_bottom_right = (start_row + row_count, start_col + col_count)
        worksheet.range(data_top_left, data_bottom_right).value = values_to_write

        # Enable auto filter + other styling to the worksheet
        worksheet.auto_filter = True
        # Takes some extra time, makes it look more pretty
        for row_i in range(start_row, start_row + row_count, 2):
            row_bounds = (row_i, start_col), (row_i, start_col - 1 + len(col_headers))
            worksheet.range(*row_bounds).style.fill.background = even_row_style

    logging.info("Saving file... Please wait!")
    path = os.path.abspath(filename)
    workbook.save(path)
    logging.info(f"Done! File save to -> {path}")
{% endhighlight %}


### The Result

The script when running locally is able to reach **12,180** files processed a second!

<a href="https://imgbb.com/"><img src="https://i.ibb.co/txyhQSG/ss-2021-07-23-at-09-08-41.png" alt="ss-2021-07-23-at-09-08-41" border="0" /></a>

This then creates a **.xlsx** file, made up of the user pc, date and time.

<a href="https://imgbb.com/"><img src="https://i.ibb.co/f8P7yq1/excel.png" alt="excel" border="0" /></a>

This is then the final format of the log file, allowing someone to easily search through all files and see files from the root of where it was ran.

<a href="https://ibb.co/x72F0ww"><img src="https://i.ibb.co/P6rDnJJ/excel-view.png" alt="excel-view" border="0" /></a>


### Case Verifying Script Breakdown

### The Code

Initially I created a function which would highlight changes between the two dataframes, showing the original and updated value, this uses the **groupby.apply** function within **Pandas**.

{% highlight javascript linenos %}
def report_diff(x):
    return x[0] if x[0] == x[1] or pd.isna(x).all() else f'{x[0]} ---> {x[1]}'
{% endhighlight %}

I then created another function which would help to remove the whitespace from within the data frames, making it easier to work with.

{% highlight javascript linenos %}
def strip(x):
    """Function to use with applymap to strip whitespaces from a dataframe."""
    return x.strip() if isinstance(x, str) else x
{% endhighlight %}

{% highlight javascript linenos %}
def diff_pd(old_df, new_df, idx_col):
    """
    Identify differences between two pandas DataFrames using a key column.

    Key column is assumed to have a unique row identifier, i.e. no duplicates.

    Args:
        old_df (pd.DataFrame): first dataframe
        new_df (pd.DataFrame): second dataframe
        idx_col (str|list(str)): column name(s) of the index,
          needs to be present in both DataFrames
    """
    # Setting the column name as index for fast operations
    old_df = old_df.set_index(idx_col)
    new_df = new_df.set_index(idx_col)
    # Get the added and removed rows
    old_keys = old_df.index
    new_keys = new_df.index
    if isinstance(old_keys, pd.MultiIndex):
        removed_keys = old_keys.difference(new_keys)
        added_keys = new_keys.difference(old_keys)
    else:
        removed_keys = np.setdiff1d(old_keys, new_keys)
        added_keys = np.setdiff1d(new_keys, old_keys)
    # Populate the output data with non empty dataframes
    out_data = {}
    removed = old_df.loc[removed_keys]
    if not removed.empty:
        out_data["Added Data"] = removed
    added = new_df.loc[added_keys]
    if not added.empty:
        out_data["Deleted Data"] = added
    # Focusing on common data of both dataframes
    common_keys = np.intersect1d(old_keys, new_keys, assume_unique=True)
    common_columns = np.intersect1d(
        old_df.columns, new_df.columns, assume_unique=True
    )
    new_common = new_df.loc[common_keys, common_columns].applymap(strip)
    old_common = old_df.loc[common_keys, common_columns].applymap(strip)
    # Get the changed rows keys by dropping identical rows
    # (indexes are ignored, so we'll reset them)
    common_data = pd.concat(
        [old_common.reset_index(), new_common.reset_index()], sort=True
    )
    changed_keys = common_data.drop_duplicates(keep=False)[idx_col]
    if isinstance(changed_keys, pd.Series):
        changed_keys = changed_keys.unique()
    else:
        changed_keys = changed_keys.drop_duplicates().set_index(idx_col).index
    # Combining the changed rows via multi level columns
    df_all_changes = pd.concat(
        [old_common.loc[changed_keys], new_common.loc[changed_keys]],
        axis='columns',
        keys=['old', 'new']
    ).swaplevel(axis='columns')
    # Using report_diff to merge the changes in a single cell with "-->"
    df_changed = df_all_changes.groupby(level=0, axis=1).apply(
        lambda frame: frame.apply(report_diff, axis=1))
    # Add changed dataframe to output data only if non empty
    if not df_changed.empty:
        out_data['Modified Data'] = df_changed

    return out_data
    {% endhighlight %}
