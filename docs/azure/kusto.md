Access the <a href="https://aka.ms/lademo">demo environment</a>

# Kusto Query Language
KQL is used to run queries in Azure Sentinel. Queries start with the data source followed
by a set of data transformation operators bound together by using the pipe delimiter:
```
SecurityEvent 
| where EventID == "4626" 
| summarize count() by Account 
| limit 10
```

# 'let' Statement
'let' statements are used to bind names to expressions and can be used
to define variables, functions and views
```
let timeOffset = 7d;
let discardEventId = 4688;
SecurityEvent
| where TimeGenerated &gt; ago(timeOffset*2) and TimeGenerated &lt; ago(timeOffset)
| where EventId != discardEventId
| where AccountType =~ "user"
| limit 100
```

```
let suspiciousAccounts = datatable(account: string) [
  @"\administrator",
  @"NT AUTHORITY\SYSTEM"
];
SecurityEvent | where Account in (suspiciousAccounts)
| limit 100
```

```
let LowActivityAccounts = 
  SecurityEvent
  | summarize cnt = count() by Account
  | where cnt &lt; 1000;
LowActivityAccounts | where Account contains "SQL"
| limit 100
```

# 'search' Operator
Use the 'search' operator to search for text in multiple tables and colums.
'search' is not as efficient as 'where' but is useful to trackdown data when 
you're unsure which table or column to filter:
```
search "temp\\startup.bat"

search in (Event) "temp\\startup.bat"
```

# 'extend' Operator
The 'extend' operator will create calculated columns and append the new columns 
to the resultset
```
SecurityEvent
| where ProcessName != "" and Process != ""
| extend StartDir =  substring(ProcessName,0, string_size(ProcessName)-string_size(Process))
| order by StartDir desc, TimeGenerated asc
| limit 100
```

# 'project' Operator
The 'project' operator can be used to control output columns from the query
<dl>
  <dt>project</dt>
  <dd>select columns to include, rename, drop or insert</dd>
  <dt>project-away</dt>
  <dd>select columns to exclude</dd>
  <dt>project-keep</dt>
  <dd>select columns to keep</dd>
  <dt>project-rename</dt>
  <dd>rename columns</dd>
  <dt>project-reorder</dt>
  <dd>reorder columns</dd>
</dl>
```
SecurityEvent
| where ProcessName != "" and Process != ""
| extend StartDir =  substring(ProcessName,0, string_size(ProcessName)-string_size(Process))
| order by StartDir desc, TimeGenerated asc
| project TimeGenerated, StartDir, Computer, SubjectUserName, SubjectMachineName
| limit 100
```

# Aggregation Operators
'summarize' can be used to group result rows. Aggregate operators can be used 
to add columns to the resultset. List of aggregate operators:

- count(), countif()
- dcount(), dcountif()
- avg(), avgif()
- max(), maxif()
- min(), minif()
- percentile()
- stdev(), stdevif()
- sum(), sumif()
- variance(), varianceif()

```
SecurityEvent
| summarize IPAddressCount = dcount(IpAddress)

SecurityEvent
| where TimeGenerated > ago(1h)
| where EventID == 4624
| where AccountType == "Machine"
| summarize howMany = count(), firstTime = min(TimeGenerated), lastTime = max(TimeGenerated) by Computer
| project Computer, firstTime, lastTime, howMany

let timeframe = 1d;
let threshold = 3;
SigninLogs
| where TimeGenerated >= ago(timeframe)
| summarize applicationCount = dcount(AppDisplayName) by UserPrincipalName, IPAddress
| where applicationCount >= threshold
```
The 'arg_max' and 'arg_min' functions return rows for the max and min values of their
first parameter respectively. Subsequent parameters specify the list of columns to return. An 
asterix can be used to return all columns:
```
SecurityEvent 
| where Computer == "SQL12.na.contosohotels.com"
| summarize arg_max(TimeGenerated, Account, AccountType, EventID) by Computer
```
'make_' functions return a dynamic (JSON) array:
```
SecurityEvent
| where EventID == "4624"
| summarize make_list(Account) by Computer

SecurityEvent
| where EventID == "4624"
| summarize make_set(Account) by Computer
```
'make_set' will return a list of unique values for the given expression

# 'render' Operator
The 'render' operator is used to generate visualisations:
```
SecurityEvent 
| summarize count() by Account
| render barchart

SecurityEvent 
| summarize count() by bin(TimeGenerated, 1d) 
| render timechart
```

# 'union' Operator
The 'union' operator takes two or more tables and returns the rows of
all of them. 
```
SecurityEvent
| union SigninLogs

union Security*
| summarize count() by Type
```

# 'join' Operator
The 'join' operator merges the rows of two tables to form a new table
by matching specified column values from each table
```
SecurityEvent 
| where EventID == "4624" 
| summarize LogOnCount=count() by EventID, Account 
| project LogOnCount, Account 
| join kind = inner (
  SecurityEvent 
  | where EventID == "4634" 
  | summarize LogOffCount=count() by EventID, Account 
  | project LogOffCount, Account 
) on Account
```
Various join types are available:

- inner: returns rows that match in both left and right
- leftsemi: returns all left-side rows that have a match on the right-side
- leftanti (leftantisemi): returns all left-side rows that don't have a match on the right-side
- leftouter (rightouter or fullouter): returns a row for every row on the left and right, even if it has no match
- rightsemi
- rightanti


# Unstructured Data
The 'extract' operator can be used to get a match for a regular expression from a text string. The
arguments for extract are:

- regex
- captureGroup
- text string
- type literal (optional)

```
SecurityEvent
| where EventID == 4672 and AccountType == 'User'
| extend Account_Name = extract(@"^(.*\\)?([^@]*)(@.*)?$", 2, tolower(Account))
| summarize LoginCount = count() by Account_Name
| where Account_Name != ""
| where LoginCount < 10
```
The 'parse' operator can be used to convert a string column into one or more calculated columns. The
computed columns will have nulls for unsuccessfullly parsed strings
```
let Traces = datatable(EventText:string)
[
"Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=27, sliceNumber=23, lockTime=02/17/2016 08:40:01, releaseTime=02/17/2016 08:40:01, previousLockTime=02/17/2016 08:39:01)",
"Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=27, sliceNumber=15, lockTime=02/17/2016 08:40:00, releaseTime=02/17/2016 08:40:00, previousLockTime=02/17/2016 08:39:00)",
"Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=27, sliceNumber=20, lockTime=02/17/2016 08:40:01, releaseTime=02/17/2016 08:40:01, previousLockTime=02/17/2016 08:39:01)",
"Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=27, sliceNumber=22, lockTime=02/17/2016 08:41:01, releaseTime=02/17/2016 08:41:00, previousLockTime=02/17/2016 08:40:01)",
"Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=27, sliceNumber=16, lockTime=02/17/2016 08:41:00, releaseTime=02/17/2016 08:41:00, previousLockTime=02/17/2016 08:40:00)"
];
Traces  
| parse EventText with * "resourceName=" resourceName ", totalSlices=" totalSlices:long * "sliceNumber=" sliceNumber:long * "lockTime=" lockTime ", releaseTime=" releaseTime:date "," * "previousLockTime=" previousLockTime:date ")" *  
| project resourceName, totalSlices, sliceNumber, lockTime, releaseTime, previousLockTime
```

# Structured Data
Dynamic fields in a table contain key-value pairs. Values can be accessed using dot notation
```
SigninLogs 
| extend OS = DeviceDetail.operatingSystem
```
Fields that store JSON data can be converted to dynamic fields using the todynamic or parse-json functions
```
SigninLogs
| extend Location =  todynamic(LocationDetails)
| extend City =  Location.city
| extend City2 = Location["city"]
| project Location, City, City2
```
'mv-expand' can be used to duplicate each row for every key-value pair found in the JSON field.
```
SigninLogs
| mv-expand Location = todynamic(LocationDetails)
| project Location, LocationDetails
| limit 20
```
'mv-apply' can be used to filter the output by key value
```
SigninLogs
| mv-apply Location = todynamic(LocationDetails) on ( where Location.city == "Cannock")
| project Location, LocationDetails
| limit 20
```

# External Data
The 'externaldata' operator is used to connect to and query from an external data source:
```
externaldata ( ColumnName : ColumnType [, ...] )
  [ StorageConnectionString [, ...] ]
  [with ( PropertyName = PropertyValue [, ...] )]
```
For example:
```
Users
| where UserID in ((externaldata (UserID:string) [
    @"https://storageaccount.blob.core.windows.net/storagecontainer/users.txt" 
      h@"?...SAS..." // Secret token needed to access the blob
    ]))
```

# Virtual Tables
Queries created in the query editor can be saved as functions to define a virtual table. The 
virtual table can then be access by calling the name of the function
