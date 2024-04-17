# Last cell with value – Array and Xlookup

How to find the value of the last non-empty cell

#### Category: MS Excel & VBA

---

### Background

Many reports that I prepare are based on the SQL query built-in ODBC connection of Excel. When reports cover transactional data of a certain date range, SQL scripts pick up ‘From’ and ‘To’ dates from the cells in the worksheet where the linkages for parameters are established through the ODBC connection.

For my own sake, I have been keeping the list of the periods covered by the previous reports so I would know where I left off last time. The cells to be used in the connection for parameters (arrow 1 below) need to be static, I cannot bypass the first arrow by directly picking up the max values of the From/To list. So, I like to start to think of the formula that I can use to pick up the last value from the From/To list.

![last-value-from-the-list](/images/last-value-from-the-list.png)

### Why Can’t I just use the MAX function?

The data type of alias Change_Requested_Date is actually Datetime which includes a timestamp in addition to the date. So, I had to use To_Char to convert it to the character. I also considered To_Date but ended up using To_Char because Excel recognizes dates as numbers (i.e. It stores 45077 for 2023-05-31) and the number is not picked up by the SQL query unless I convert the number to date in the script. For this reason, the dates in From/To list in Excel are actually in text format by adding a prefix ‘ (i.e. ‘2023-05-31). So, the function MAX designed for picking up the maximum numbers from the list does not work for the string/text/character datatype in this case.

### The solution I came up with and debrief

![last-value-from-the-list-formula](/images/last-value-from-the-list-formula.png)

Let’s take a look at the syntax of xlookup before debriefing the solution.

=XLOOKUP(lookup_value, lookup_array, return_array, [if_not_found], [match_mode], [search_mode])

=XLOOKUP(FALSE ,(N:N=””) ,N:N , , 0 ,-1 )

We are going to have to tag cells differently depending on the **filled/empty status**. Since **it is a true/false question, a Boolean datatype would be ideal**. The first focus in this formula should be on the lookup_array argument. N:N is an array of text. It will be [2022-09-29, 2022-10-28, 2022-11-30, and so on]. N:N=”” will return the array of Falses and Trues. Checking if each element in array N:N is equal to “” (blank), it would return True when the cell is empty and False when the cell is filled with any value. The result of N:N=”” would be [True, True, True, True, True, True, True, True, True, False, False, False, …] and it will be False’s as long as the cells are empty.

Xlookup has a search mode that was not available with Vlookup. It provides an option among to-down, down-top, binary search ascending, and binary search descending. Search_mode -1 corresponding to down-top was used in my solution to **scan the array from bottom to top** until it finds the lookup_value that I provided which is “False”.

Once the function finds the first False which is the first non-empty cell from the end of the array, it will return the value with the same order from return_array N:N. Just for clarification on all other arguments in the formula, I left if_not_found blank because it was certain that the cell with value is found and 0 was used in match_mode to find the exact match as it would be either True or False only with no variation.

### Some variations of the formula

I also can think of some variations and like to explain the benefits of them depending on the circumstance.

Original formula: =XLOOKUP(FALSE,(N:N=””),N:N,,0,-1) is preferred if the scope of work is limited in the Excel workbook as TRUE/FALSE in the formula is intuitive.

If the work needs to be communicated with other scripts, I would use =XLOOKUP(0,1*(N:N=””),N:N,,0,-1). By multiplying 1 (or adding 0), Excel converts the True/False to 1/0. Replacing True/False to 1/0 will increase compatibility and enable the code reproducible in other scripts.

Searching for True (or 1) instead of False (or 0) could be an option =XLOOKUP(1,1*(N:N<>””),N:N,,0,-1) because the status of a cell filled is more in line with True and False is more intuitive for emptiness or Null.
