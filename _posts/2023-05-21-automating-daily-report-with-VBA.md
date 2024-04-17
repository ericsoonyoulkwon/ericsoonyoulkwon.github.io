# Automating daily reports with VBA

Writing a macro to automate folder editing and Excel reports because I want to avoid doing it manually every day!

#### Category: MS Excel & VBA

---


### So what is currently done manually?

One of my daily tasks is to create a response report for the data interfaced from our clients and the tasks involved break down into the following:

1. Creating a folder in a network drive and name it after today’s date
2. Creating an Excel reporting template file with today’s date in its name
3. Running the SQL query of data processed to the system on the previous business date
4. Confirming all data received as XML file was processed
5. Pasting the queried data to the template to be sent
6. Printing the report as PDF in the network folder created
7. Saving the Excel file in the same folder and close it
8. Logging in to our ERP and reconciling the number of data in the report
9. Opening the shared file-transfer-protocol (sFTP) application and log in to upload the Excel and PDF files

The query was already proven to be reliable as I run it every day and I already customized the SQL script and Excel ODBC connection to the server so it runs automatically as soon as the file is opened and [grabs all data transmitted on the evening of the previous business day](https://ericsoonyoulkwon.github.io/2023/06/02/selecting-data-transacted-on-previous-business-day.html){:target="_blank"}. (i.e. The query captures the data for Friday when I run the script on Monday. It also captures the interface on Thursday or Friday when it was a long weekend that I didn’t run the report on Friday or Monday, respectively.)

### Let’s get down to business

The list of tasks is already well defined, so I started to write down procedures on the VBA editor built in MS Excel. Please feel free to [contact me](ericsoonyoulkwon@gmail.com){:target="_blank"} for any suggestions.

#### Check if there is already a folder with the same name

I started to create a function to check if the folder with the name that I wanted to use already exists. The function takes whatever string that I put as a parameter and checks if the same directory exists or not. If there already is one, it will return True.

```
Function FolderExists(ByVal Path As String) As Boolean

    FolderExists = False
    Dim fso As New FileSystemObject

    If fso.FolderExists(Path) Then FolderExists = True

End Function
```

Then I defined the directory of the folder where the new daily folders are saved. I wanted the format of the date used in the folder to be like “May 20 2023 – M”

```
PathName = "K:\the directory of the folder that I am checking\"
FolderName = MonthName(DatePart("m", Date), False) & _
    " " & Day(Date) & _
    " " & DatePart("yyyy", Date) & _
    " - M"
```

Now I call the function that I just created to check if the directory of the folder with today’s date exists or not. If it doesn’t exist and the function returns False, it will proceed with the code that I write in the section after ‘Then’. Otherwise, it will prompt a message box to tell me the folder already exists with no further actions.

```
If Not FolderExists(PathName & FolderName) Then
    'code to be executed when there is no folder with the same name
Else
    MsgBox ("Folder for today already exists.")
End if
```

#### Create a folder if there isn’t already one

Then I began to write things that I wanted to execute when there was no folder with today’s date. Of course, creating the folder is the first thing that I would do. To access and modify files and folders, I needed to use the file system object of MS Scripting Library. It was optional that I included the library name ‘scripting’ just to add more clarity when I create the object even if I didn’t see there is potential ambiguity when there isn’t another library with the same object name `filesystemobject`.

After concatenating PathName and FolderName, I am creating a folder. (i.e. “K:\the directory of the folder that I am checking\May 20 2023 – M”) The FTP application that I am going to log in as the last step of the process has access to the folder so I don’t need to see it in the file explorer. It was an optional step for adding visibility.

```
Set fso = CreateObject("scripting.filesystemobject")
fso.CreateFolder PathName & FolderName

Call Shell("explorer.exe" & " " & PathName & FolderName, vbNormalFocus)
```

#### The folder is created. Now what? Gotta finish the report!

The tasks are very light and won’t take up much memory but I prefer not to see every single change in Excel so I usually turn off the screen update and turn it back on at the end of the script. I found this helps lower the chance of seeing the grey frozen screen with the notorious “(Not Responding)” message and losing the data of all unsaved workbooks.

The template is pasted to the folder just created and named with the date at the end of the file name. (i.e. the name of the file to be sent out will be like “file name-20230520.xlsm, the macro-enabled file that automatically updates its headers according to the date in the file name when it’s opened and prints the report as PDF when it is closed.)


```
Application.ScreenUpdating = False

Template = "K:\where I saved the template to be uploaded\the name of the template file.xlsm"
NewFile = PathName & FolderName & "\the name of the template file-" & Format(Now, "YYYYMMDD") & ".xlsm"

FileCopy Template, NewFile
```

Then the Excel file with a table where the SQL query is built is opened, the table is refreshed, and the script is set to wait until the query is refreshed. No one wants the empty table to be copied and pasted if it happens too soon.

I always prefer to count the last rows, and columns if necessary, of the sheet and copy the right amount of data so there is no waste of memory and time. For the same reason, I want this to be done only when there is data to be reported as there sometimes are days with no data interfaced and the report would be empty with just column headers.

```
Sqlquery = "K:\where I saved an excel file with the SQL query\an excel file with the ODBC connection to the server that the query script can be run on.xlsm"
Set wsCopy = Workbooks.Open(Sqlquery)
Set wsDest = Workbooks.Open(NewFile)

wsCopy.RefreshAll
wsCopy.Application.CalculateUntilAsyncQueriesDone

lastRowCopy = wsCopy.Worksheets("Query").Cells(Rows.Count, 1).End(xlUp).Row
lastRowDest = wsDest.Worksheets("Report").Cells(Rows.Count, 1).End(xlUp).Row

If lastRowDest > 2 Then _
    wsDest.Worksheets("Report").Range("A2:N" & lastRowDest).EntireRow.Delete
If lastRowCopy > 2 Then _
    wsCopy.Worksheets("Query").Range("A2:N" & lastRowCopy).Copy _
    wsDest.Worksheets("Report").Range("A2:N" & lastRowCopy).PasteSpecial _
        Paste:=xlPasteValues
```

When the client requires to use the same file name for the PDF version, I take the name of the file without “.xlms”, save it as pdf, then save and close the Excel file.

```
With wsDest
    ThisFile = Left(ActiveWorkbook.Name, Len(ActiveWorkbook.Name) - 5)
    SvAs = PathName & FolderName & "\" & ThisFile & ".pdf" _
        .ActiveSheet.ExportAsFixedFormat Type:=xlTypePDF, FileName:=SvAs, _
        Quality:=xlQualityStandard, IncludeDocProperties:=False, _
        IgnorePrintAreas:=False, OpenAfterPublish:=True
    .Close Savechanges:=True
End With
```

#### Quality control of the report

There is no guarantee that all records received via the interface are necessarily processed by the application. Traditionally, this could be checked by comparing the number of records in the XML file received and the number of records processed and captured in the SQL query.

I decided to take advantage of certain elements tagged to each record in an XML file. So, I was able to count the records received on the evening of the previous business day. When an XML file is opened and closed by Excel, there are a series of pop-ups that will halt the automated process. So, I disabled alerts and reactivated them at the end. Simply incrementing (aka rolling-sum) the variable by 1 when there is a row with the tagging element would result in the number of records received. I am going to include the reconciliation between the record count in XML received and the record processed in the SQL query in the message box that will pop up at the end of the automated process.

```
Application.DisplayAlerts = False
Application.EnableEvents = False

inbound_date = Format(Evaluate("WORKDAY(TODAY(),-1,Days_off!A:A)"), "YYYYMMDD")
XML_location = "\\location where the XML files are saved\"
Set inbound_xml = Workbooks.OpenXML(FileName:=XML_location & "name of the XML file" & inbound_date & ".xml", LoadOption:=xlXmlLoadImportToList)
lRow_inbound = Cells(Rows.Count, 9).End(xlUp).Row
    
inbound_records_count = 0
For i = 1 To lRow_inbound
    If Cells(i, 9) = 1 Then inbound_records_count = inbound_records_count + 1
Next i

inbound_xml.Close
Application.DisplayAlerts = True
Application.EnableEvents = True
```

#### A little bit trickier to count the records processed and included in the report

The SQL report is the summary of the action items created by the system and it is possible that one record interfaced creates more than one item in the system. So, simply comparing the number of records in the interface that I just counted above against the number of rows in the SQL report is not going to help reconciliation.

My goal here is to get 1 for each record as duplicates don’t count. I was almost going to increment a variable by 1 whenever I came across a new record by skipping the records already counted. Then, I thought creating and appending the array of new records and checking duplicates against the array takes more memory than the following simpler solution.

Keeping the same goal of having 1 for each unique record, I started to categorize the types of records based on the occurrence. First of all, there will be records that occur only once with no duplicates. However, there could be unlimited cases for duplicated records when it can create how many ever action-items in the system. It is a one-or-many situation. So, I tried summing the product of dividing 1 by the occurrence of the record will result in 1 for all unique records (n * 1/n = 1) and it worked.

```
Set report = wsDest.Worksheets("Report")

records_processed = 0
    For p = 2 To lastRowCopy
        duplicates = Application.WorksheetFunction. _
            CountIf(report.Range("A2:A" & lastRowCopy), report.Cells(p, 1))
        If duplicates > 1 Then
            records_processed = records_processed + (1 / duplicates)
        ElseIf duplicates = 1 Then
            records_processed = records_processed + 1
        End If
        duplicates = 0
    Next p
```

#### Heavy lifting is done. Just a few more steps…

The majority of tasks are now done; a folder is created, queried data is pasted to the file to be sent out, and the same report is saved as a pdf. I go to the ERP that opens up in the web browser and log in to see the dashboard because if the transmitted data was picked up and processed last night, I should be able to see the same change in the ERP as well. (Although I didn’t include this step in the SOP, I perform this as an optional check myself.) I found that waiting 5 seconds is sufficient to open the web browser. Then send the user ID, tab key, password, and enter key to see the dashboard.

```
Application.ThisWorkbook.FollowHyperlink _
    Address:="https://address of the ERP/"
Application.Wait (Now + TimeValue("00:00:05"))
Application.SendKeys ActiveSheet.Range("cell that I save User ID").Value
Application.SendKeys ("{TAB}")
Application.SendKeys ActiveSheet.Range("ell that I save User PW").Value
Application.SendKeys ("~")
```

Then the FTP application is opened, allowing 5 seconds again to wait until it opens. Pressing the enter key is enough when the login credential is saved in the application.

```
sFTP = Shell("C:\location\FTPApplication.exe", 1)
Application.Wait (Now + TimeValue("00:00:05"))
Application.SendKeys ("~")
```

Finally, showing a message box that reminds me what to do next such as final quality check and upload.

Not sure why, but I noticed that the Number lock is on when SendKeys is executed. So, I always turn it off after I use the SenKeys prompt so that users don’t get surprised when they use the number pad.

Also, the data types of variables used were declared.

```
Application.ScreenUpdating = True
wsCopy.Worksheets("Report completed").Activate
Application.SendKeys ("{NUMLOCK}")
        
MsgBox ("Daily Response Report for " & FolderName & " was successfully created." & vbCrLf & "Please perform the following" & vbCrLf _
& "   - confirm the pdf = excel" & vbCrLf _
& "   - reconcile ToDo's" & vbCrLf _
& "   - upload via sFTP" & vbCrLf & vbCrLf _
& "Records received through interface: " & inbound_records_count & vbCrLf _
& "Records processed by application: " & records_processed)
```

```
Dim PathName, FolderName, Template, NewFile, Sqlquery, wsCopy, wsDest, ThisFile, SvAs, sFTP, inbound_date, XML_location As String
Dim lastRowCopy, lastRowDest, lRow_inbound, inbound_records_count, records_processed, duplicates As Long
Dim fso As New FileSystemObject
Dim inbound_xml As Excel.Workbook
```

### Was this worth it?

After I spent a maximum of 5 hours to automate the report preparation, the estimated time that I spend on the series of procedures is a minimum of 5 minutes every day and it would be around 20 hours a year assuming there are 250 working days in a year if I still do this manually. Plus, we have mirrored teams with identical responsibilities for two of our clients and this means the benefit of this will be doubled when one of my coworkers can save their time too. I do not doubt that this adds value when benefits outweigh costs, both measured in time.

Automating no-brainer tasks, I can spend more time with more valuable work now.




