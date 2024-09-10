# How To Build Your Own KQL (Basic)

## Intent of Instruction

The intent of this instruction is to broadly educate analysts in the use of KQL where it relates to investigating Microsoft alerts. This instruction will assume the reader has a basic understanding of syntax and logic in regard to coding/scripting/querying languages.

- For more information on what a "[querying langauge](https://www.splunk.com/en_us/blog/learn/query-languages.html)" or "[syntax](https://www.codingdojo.com/blog/syntax-in-programming)" or "[logic gates](https://www.codecademy.com/resources/blog/what-is-boolean-logic/)" are please follow the hyperlinked text.
- Additionally, for more information on KQL you can visit this link here: [Must Learn KQL GitHub Repo](https://github.com/rod-trent/MustLearnKQL/tree/main)
    - GitHub has links that will also point to specific articles in the Microsoft Learn site.

* * *

## Table of Contents

- [What is KQL?](#what-is-kql?)
    - [Where To Query Logs](#where-to-query-logs)
- [Tables and How to Find Them](#tables-and-how-to-find-them)
    - [Common Tables](#common-tables)
- [Filtering Your Results](#filtering-your-results)
    - [Using The UI](#using-the-ui)
- [Cross-referencing Tables](#cross-referencing-tables)
    - [Formatting Our Results](#formatting-our-results)
- [Bonus Information](#bonus-information)
- [KQL Best Practices](#kql-best-practices)
- [Appendix](#appendix)
    - [Defender For Endpoint Queries](#defender-for-endpoint-queries)
    - [365 Defender For Office Queries](#365-defender-for-office-queries)
    - [Sentinel Queries](#sentinel-queries)

* * *

## What Is KQL?

According to [Microsoft Documentation](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/query/) on KQL:

- "Kusto Query Language (KQL) is a powerful tool to explore your data and discover patterns, identify anomalies and outliers, create statistical modeling, and more. KQL is a simple yet powerful language to query structured, semi-structured, and unstructured data. The language is expressive, easy to read and understand the query intent, and optimized for authoring experiences. The query uses schema entities that are organized in a hierarchy similar to SQLs: databases, tables, and columns."

In simpler terms, KQL is a [querying language](https://en.wikipedia.org/wiki/Query_language#:~:text=A%20query%20language%2C%20also%20known,Structured%20Query%20Language%20%28SQL%29. "https://en.wikipedia.org/wiki/Query_language#:~:text=A%20query%20language%2C%20also%20known,Structured%20Query%20Language%20(SQL).") whose syntax allows for the filtering of information from within Microsoft Sentinel down to the desired output. Much of which is presented to the user relatively quickly and in a way that is easy to understand.

> [Microsoft Sentinel](https://learn.microsoft.com/en-us/azure/sentinel/overview?tabs=azure-portal) is a scalable, *cloud-native* security information and event management (SIEM) that delivers an intelligent and comprehensive solution for SIEM and security orchestration, automation, and response (SOAR).

The thing that makes KQL so different from other querying languages, like Splunk's SPL or SQL, is that KQL was built to parse data from a *cloud* database. KQL was created specifically for Sentinel. Not only that, it was built to be more intuitive to the analyst when writing queries. Instead of SQL syntax like

```sql
SELECT CustomerName, City FROM Customers;
```

An analyst can instead use a syntax much more similar to how they think about the type of data they are gathering:

```kql
// From the table Customers summarize the customer name and city
Customers
| summarize by CustomerName,City
```

* * *

### Where To Query Logs

While every tool or suite that a client has may aggregate information into Microsoft Sentinel, Sentinel is not the only tool that can be used to search logs. Microsoft 365's "Advanced Hunting" feature can also be used to investigate logs for information related to a given alert.

To access this functionality, utilize at least one of the following:

1.  Microsoft Sentinel under ***Logs***  
    ![Screenshot 2024-07-20 115637.png](/images/d6b023e0dc1541aa80077396dab586fe.png)
    
2.  Microsoft 365 Defender/ Microsoft XDR console under ***Hunting*** -> ***Advanced Hunting***  
    ![Screenshot 2024-07-20 115649.png](/images/7b3676eade4c40008403453491186c98.png)
    

* * *

## Tables and How to Find Them

The most common tables you may want to query for information from depend on the type of information you are searching for. Knowing which tables to search in takes a little bit of experience and you pick up on it as you learn more about the tools. A lot of the time, the most relevant table you can check for information is noted in the IOC query for any given alert. It is important to note that sometimes that you may not have access to the query, which can be troublesome for any beginner.

> Note: Logs are stored in tables, many of which can be found here: [Tables - MS Documentation on Learn](https://learn.microsoft.com/en-us/azure/azure-monitor/reference/tables/tables-category)

The easiest way to find something when you do not know how to to find it is using the [search operator](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/query/search-operator?pivots=azuredataexplorer).

- The `search` operator will search for a string of text across **ALL** available tables you will have access to. To use the search operator, use the following syntax:

```kql
search "<insert relevant string>"
```

- To search for an item/string in a specific table or set of tables you can use the following query.

```kql
search in (Table1,Tab2,Tabl*) "<insert relevant string>"
```

- Reviewing the above query you can see that different tables can be grouped together (Table1 and Tab2 table) as well as wildcard a table (Tabl\*) to include multiple tables that start with a certain set of characters.

> Note: Generally speaking, a wildcard is a special character that can represent unknown characters in a search term or pattern to help find multiple items with similar data.

But lets say you do not know *what table* to search in. Well, there is an answer to that. You can use the search operator followed by some filtering operators!

- The filtering operator you are going to use to find tables where the item you are searching for exists, is the `distinct operator`. [The distinct operator](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/query/distinct-operator) produces a table of distinct columns based on the item provided. In the following query you are creating a distinct column for the [entity](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/query/schema-entities/entity-names) ($) table:

```kql
search "filename.exe"
| distinct $table
```

- The query will return results that involve the string "filename.exe" AND define the table corresponding to where that data was found. From here you can pick out a table or tables that closely resembles the data you deem useful and filter further with other operators.

### Common Tables

Some of the more common tables you may find yourself searching for information in are the following:

- When searching for files relating to Microsoft Defender for Endpoint alerts:
    
    - [DeviceNetworkEvents](https://learn.microsoft.com/en-us/azure/azure-monitor/reference/tables/devicenetworkevents) - Holds network information from a monitored device.
    - [DeviceEvents](https://learn.microsoft.com/en-us/azure/azure-monitor/reference/tables/deviceevents) - Holds general events from a monitored device which can include AV and exploit protection.
- When searching for emails relating to Microsoft 365 Defender detections:
    
    - [EmailEvents](https://learn.microsoft.com/en-us/azure/azure-monitor/reference/tables/emailevents) - Holds the information as it relates to any given email.
    - [EmailAttachmentInfo](https://learn.microsoft.com/en-us/azure/azure-monitor/reference/tables/emailattachmentinfo) - Holds information as it relates to the attachments in an email.
    - [EmailUrlInfo](https://learn.microsoft.com/en-us/azure/azure-monitor/reference/tables/emailurlinfo) - Holds information as it relates to a URL found in an email.
    - [UrlClickEvents](https://learn.microsoft.com/en-us/azure/azure-monitor/reference/tables/urlclickevents) - Contains events involving URLs clicked, selected, or requested on Microsoft Defender for Office 365.
    - [DeviceNetworkEvents](https://learn.microsoft.com/en-us/azure/azure-monitor/reference/tables/devicenetworkevents) - (*This only exists if the client has Defender for Endpoint*)
    - [AADSignInEventsBeta](https://learn.microsoft.com/en-us/defender-xdr/advanced-hunting-aadsignineventsbeta-table) - Contains information about Microsoft Entra interactive and non-interactive sign-ins.
        - This can be useful when pivoting to the client's Sentinel console is not viable.
- When searching for information within Sentinel, you can review the following tables:
    
    - [CommonSecurityLog](https://learn.microsoft.com/en-us/azure/azure-monitor/reference/tables/commonsecuritylog) - Holds data pulled from different security appliances such as Check Point, Palo Alto and more.
    - [SigninLogs](https://learn.microsoft.com/en-us/azure/azure-monitor/reference/tables/signinlogs) - Also holds sign-in logs.
    - Occasionally you'll have custom log sources and these will begin with `_TableName`.

You may find other tables relevant to your search as your experience grows from detection to detection. Its important to explore additional options when possible.

[Back to Top](#how-to-build-your-own-kql-basic)

* * *

## Filtering Your Results

So now you have your relevant tables with a ton of results. Now what? You filter that down of course! The most common filtering operator you will use is the [where operator](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/query/where-operator).

- The `where` operator takes a "predicate", which is a column presented in the table such as `FileName` in the table `DeviceEvents` followed by a Boolean operator such as `=`, `!=`, `==` and `=~` (though more can be found here: [Logical Operators](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/query/logical-operators), [Numerical Operators](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/query/numerical-operators), [String Operators](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/query/datatypes-string-operators)) and then finally a string (within double or single quotes) or an integer :

```kql
DeviceEvents
| where FileName =~ "filename.exe"
```

- This example demonstrates how the `where operator` helps to find a file containing the string `filename.exe`. You can supply more filtering `where operators` with subsequent pipe `|` characters followed by the `where operator`:

```kql
DeviceNetworkEvents
| where DeviceName == "MacbookPro"
| where RemoteURL contains "google.com"
```

- You may also get even more granular with your queries by providing Boolean logic! You can try adding `and`, `or` and `not` as well as others (hyperlinked above). This includes using `!` as a `not operator`. You can even use groups to further complicate your filtering by encasing them in parenthesis:

```kql
DeviceEvents
| where DeviceName contains "mac" 
    and (DeviceName contains "book" or DeviceName contains "pro")
| where DeviceName !contains "Desktop"
```

The `where operator` can also define time constraints as well as using wildcards in place of a "predicate" to search for any given string!

```kql
EmailEvents
| where TimeGenerated < ago(5min)
| where * contains "C O N G R A T U L A T I O N S"
```

### Using The UI

Of course, you don't have to sift through your results by hand. There are handy tools at your disposal within the User Interface as well. This is true for both Query locations within Sentinel and the 365/XDR Defender console.

- Lets say, for example, you opt to use the `search operator` to attempt to find an artifact or IOC found in the CORR alert you are working. You go to the console and pop in a simple KQL Query using the `search operator` as follows:

```kql
search 'googl.com'
```

- In Sentinel, hidden behind three dots on left hand side where the `Table`, `Queries` and `Functions` tabs are displayed, is the `Filter` tab!  
    ![Screenshot 2024-07-20 191005.png](/images/992388410c7b478599b008f469938f2c.png)

This tab is very powerful and highly useful for both skilled and unskilled analysts alike. The `Filter` tab helps to filter your data based on the information you want to see. For example, lets say from the query we've run above, that we want to see `UrlClickEvents` involving `google.com`. We can check the box for `UrlClickEvents` and then click the blue `Run & Apply` button for the query to update.  
![Screenshot 2024-07-20 191636.png](/images/df9ad8f07a4b4fdcb5680cac7057f10e.png)

Once you make your selection and click the blue button, the query updates and now you are able to filter your results further!

![Screenshot 2024-07-20 191925.png](/images/0230176ef7f54cca902693cd8b3adc6d.png)

Similar functionality exists in 365 Defender as well, just under a different UI button:  
![Screenshot 2024-07-21 103031.png](/images/452824d1a80943f4a3da0dbabbdd63c6.png)

The filtering doesn't end there! You can also use the "search bar", which acts like an in UI `search operator` to filter through the current results you have queried using a string of text:

![Screenshot 2024-07-21 103206.png](/images/c7dd5e51d997464e9cfab961a3d38a06.png)

There is a limitation to using the `Filter` tab, though. You can only filter based on parameters available for **the top 10 results**, meaning that if an `AccountUPN` you are searching for is not present in the first top 10 results, it does not necessarily mean that the `AccountUPN` you are concerned with is not in those results at all.

Additionally the filters are applied using `==` Boolean operator, meaning ***exact match*** when the row is **not empty**, so use of this is limited. Once you know what you are doing, you can alter the query to achieve your desired results.

- The only exception to this being the Isempty() function when an item a row does not have a value assigned to it, but covering that exceeds the purpose of this portion.
- For more info you can review that particular function here: [isempty()](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/query/isempty-function)

[Back to Top](#how-to-build-your-own-kql-basic)

* * *

## Cross-referencing Tables

So, lets say you've found your desired data, but you also want to cross-reference that data to another table. You can use the [join operator](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/query/join-operator?pivots=azuredataexplorer). The `join operator` merges the rows of two tables to form a new table by matching values of the specified columns from each table. This operator alone can be pretty tricky to use when defining some of its more advanced options, however by default it is actually pretty simple.

- To give an example, this can come in handy when working with emails because the data for emails (`EmailEvents`) and the data for email content (such as attachments (`EmailAttachmentInfo`) and URLs (`EmailUrlInfo`)) are kept in separate tables. The code block below demonstrates how the join operator works to this effect:

```kql
EmailAttachmentInfo
| where FileName contains "malware.pdf"
| join EmailEvents on NetworkMessageID
```

- The above query works because both tables will have the `NetworkMesageID` predicate/column. If you were looking for an email with an attachment under the file name `malware.pdf` AND you wanted to look at the emails containing that pdf you can match them to each other allowing you, as the analyst, to review that email directly from the results of your query (by clicking on the now hyperlinked `NetworkMessageID` data).
- Additionally, you can also join more than two tables together in a query such as what is demonstrated below. The following code block can be useful when looking for emails with a specific URL (`EmailUrlInfo`) that have been clicked (`UrlClickEvents`) and track that email through `EmailEvents`.

```Kql
EmailUrlInfo
| where Url contains "companydomain.com"
| join UrlClickEvents on NetworkMessageID
| join EmailEvents on NetworkMessageID
```

- Its important to note though that when using the `join operator` it is considered best practice to join from the smallest dataset to a larger data set, then filtering those results with a `where operator`.

The `join operator` is not the only way to bring tables together. The other option to join tables is the `union operator`. The [union operator](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/query/union-operator?pivots=azuredataexplorer) takes two or more tables and returns the rows of all of them. This is actually useful in a different way because when you union tables together their data is not "joined", its displayed independently in its own row. This is similar to how the `search operator` displays its results.

To use the `union operator` the syntax looks a lot like this:

```kql
union DeviceEvents,DeviceProcessEvents
| where FileName =~ "malware.exe"
```

- In the above example, you are filtering a like predicate, `FileName`, that is present in both tables and the results will filter both tables for results containing the string `malware.exe` for that row.
- You can also wildcard multiple tables together which can be helpful when searching across the `Device*` tables, mostly because their predicates match in many useful ways.

```kql
union Device*
| where TimeGenerated < ago(5min)
| where SHA256 == "131f95c51cc819465fa1797f6ccacf9d494aaaff46fa3eac73ae63ffbdfd8267"
```

[Back to Top](#how-to-build-your-own-kql-basic)

* * *

### Formatting Our Results

Now that you have the data you want, you will need to format your results so that you can provide the client with the details they require. To do this, you use a few different operators as well as some functions native to KQL. These are the [summarize operator](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/query/summarize-operator) and [project operator](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/query/project-operator) as well as the functions: [count()](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/query/count-aggregation-function). [count_distinct()](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/query/count-distinct-aggregation-function), [make_set()](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/query/make-set-aggregation-function) and [make_list()](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/query/make-list-aggregation-function) though you can review any and all others here: [KQL Functions](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/query/aggregation-functions)

First I'll cover the `summarize operator` mentioned above:

- The `summarize operator` produces a table that aggregates the content of the input table. What does this mean? In layman's terms, it "summarizes" the results of any given predicate or column.
    - An example of this is could be when you are trying to find a list of user's affected by a suspected phishing campaign:

```kql
EmailEvents
| where SenderFromAddress contains "@phishingdomain.com"
| summarize by RecipientEmailAddress
```

- This will produce a list of Recipients that received an email from an email address containing that string, `@phishingdomain.com`. Depending on what events are available this can produce some duplicates and/or other undesirable data in your results. The `summarize operator` is not designed to be the end all for fine tuning your output. Many of the function covered in the remainder of this portion depend on the `summarize operator` being present.
    
- For example, it may be beneficial to supply a client with the number of times an individual event may have occurred. This is where you can use the `count()` function in conjunction with the `summarize operator`. The `count()` function simply counts the number of records per summarization group, or total if summarization is done without grouping.
    

```kql
EmailEvents
| where Subject contains "C O N G R A T U L A T I O N S"
    and Subject !contains "Re:"
| summarize count() by Subject
```

- The above example demonstrates how you might use the `count()` function. In this example you want to find emails where the `Subject` has the string `C O N G R A T U L A T I O N S`, but filters out responses (`Re:`) and creates a `count()` of all emails with Subject lines that fit that criteria.
    
- Other times it may be beneficial to supply only the unique instances where a particular event may have occurred. To filter out duplicates you can use the `count_distinc()` function!
    
- The `count_distinct()` function "counts" unique values specified by the scalar expression per summary group, or the total number of unique values if the summary group is omitted. This one is important because essentially, it will produce a count of *distinct* or *unique* events that match that criteria.
    

```kql
EmailEvents
| where SenderFromAddress contains "@phishingdomain.com"
| summarize by count_distinct(RecipentEmailAddress)
```

- The above example shows the syntax you would use to get a distinct count of unique recipient addresses that received an email from a sender with `@phishingdomain.com` in the address.
    
- The `make_set()` function creates a dynamic array of the set of distinct values that expr takes in the group. This means that it will produce a set of *unique* or *distinct* values of a given predicate. The results are each unique value in an array.
    

```kql
EmailEvents
| where SenderFromAddress contains "@phishingdomain.com"
| summarize make_set(RecipientEmailAddress)
```

- The `make_list()` function is similar to the `make_set()` function mentioned previously. The `make_list()` function creates a dynamic array of all the values of expr in the group, so if there are duplicates in your results for any given column/predicate then those duplicates will be in your list.

Lastly, you have the `project operator`. This operator can select the columns to include, rename or drop, and insert new computed columns. There are sibling operators such as [project-rename](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/query/project-rename-operator) and [project-reorder](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/query/project-reorder-operator) though all `project operators` modify the columns present in a table.

```kql
EmailEvents
| where SenderFromAddress contains "@companydomain.com"
| project-rename Sender=SenderFromAddress, Sender_IP=SenderIPv4
| project-reorder Sender,NetworkMessageId,Subject,RecipientEmailAddress
| project Sender,NetworkMessageId,Subject,RecipientEmailAddress
```

- The `project operator` can define column order and what columns you want projected.
    
- The `project-rename operator` can change the name of a column and what columns you want projected.
    
- The `project-reorder operator` can change the order in which columns are presented from left to right. It can also ascend (`asc`) or descend (`desc`) the columns alphabetically or numerically (`granny-asc` and `granny-desc`)
    

[Back to Top](#how-to-build-your-own-kql-basic)

* * *

## Bonus Information

In some cases it may be beneficial to create `variables` in KQL. You can create `variables` by using the [let statement](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/query/let-statement). A `let statement` is used to set a variable name equal to an expression or a function. These **statements** allow for cleaner queries as it allows you as an analyst to re-use information. These must end with a `;` character in order for KQL to interpret the data and store it for future use in the query.

> **NOTE**: `Let statements must be followed by a semicolon. There can be no blank lines between let statements or between let statements and other query statements.`

`Let statements` can be useful when you want to supply a value or a list of values to a predicate. For example, lets say you have multiple Network Message ID's from multiple alerts you've grabbed from the SOC Tier 1 Queue and you'd like to review each one in a single query. This will require the use of setting a [Dynamic Data Type](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/query/scalar-data-types/dynamic), specifically a `dynamic literal`. The syntax for creating this kind of data type for use in as a variable in the context of network message ID's would look like this:

`dynamic([value [, ...]])` which would translate to -> `dynamic([1, 2, "hello"])`

Now, using this in mind you can use the following query to find exactly what you need!

```kql
let nwmids = dynamic([
"NetworkMessageID1", 
"NetworkMessageID2", 
"NetworkMessageID3", 
"NetworkMessageID4"]);
EmailEvents
| where NetworkMessageID in (nwmids)

```

It should be said that while variables can be used in this way, if you review some IOC queries currently in production, they are actually used in a variety of ways. Next time you are working a Defender alert check out the query and how they might use a variable.

[Back to Top](#how-to-build-your-own-kql-basic)

* * *

## KQL Best Practices

Microsoft provides a page which includes many "KQL Best Practices" when querying for information. You can review that information here: [Best practices for Kusto Query Language queries](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/query/best-practices)

- It is highly recommended to review these tips so that you can get the results you need without unnecessary slow down or overconsumption processing power.

To reference some specific rules or "best practices" to keep in mind I'll post the most relevant ones here:

- Only reference tables whose data is needed by the query. For example, when using the union operator with wildcard table references, it is better from a performance point-of-view to only reference a handful of tables, instead of using a wildcard (`*`) to reference all tables and then filter data out using a predicate on the source table name.
    
- Apply the `where query operator` immediately following table references.
    
- Then apply predicates that act upon datetime table columns. Kusto includes a very efficient index on such columns, often eliminating whole data shards completely without needing to access those shards.
    
- Apply predicates that act upon string and dynamic columns, especially such predicates that apply at the term-level.
    
- Then apply predicates that are selective and are based on numeric columns.
    
- Last, for queries that scan a table column's data, for example, for predicates such as `contains "@!@!"` that have no terms and do not benefit from indexing, order the predicates such that the ones that scan columns with less data will be first.
    

[Back to Top](#how-to-build-your-own-kql-basic)

* * *

## Appendix

For ease of use, this section will provide useful queries for various scenarios:

### Defender For Endpoint Queries

To check if a file was removed or quarantined from an Endpoint:

- Documentation for this information was found here: [Threat Severity Default Action](https://learn.microsoft.com/en-us/windows/client-management/mdm/policy-csp-defender#threatseveritydefaultaction)

```kql
DeviceEvents
// Put the hostname or part of the suspected hostname here.
| where DeviceName contains "hostname"
| where ActionType contains "Anti"
// place the filename in the section, or parts of the filename
| where FileName == "Filename"
| extend parse_json(AdditionalFields)
    | extend Action = tostring(AdditionalFields.Action)
// The action number determines whether it was removed (6) or quarantined (2) 
    | where Action == 2 or Action == 6
```

To determine whether the activity originated from a USB or removable media:

```kql
DeviceEvents
| where ActionType == "PnpDeviceConnected"
| where DeviceName has "host_name"
//| where ActionType has "File"
| extend parsed=parse_json(AdditionalFields)
| extend 
DeviceDescription=tostring(parsed.DeviceDescription),
ClassName=tostring(parsed.ClassName)
| where
ClassName in ("DiskDrive", "CDROM")
or ClassName contains "nas"
or ClassName contains "SCSI"
```

### 365 Defender for Office Queries

To check for UrlClickEvents for any given Network Message ID:

- The following query checks for a Url click where the click was allowed OR was it was blocked to check whether the user clicked through the block.

```kql
EmailEvents
// Put the NetworkMessageId for any given email
| where NetworkMessageId == "networkmessageid"
| join UrlClickEvents on NetworkMessageId
// To check if the user was tagged as clicking from the email
// put their UPN here.
| where AccountUpn contains "upn"
| where ActionType =~ "ClickAllowed" or IsClickedThrough != 0
```

To check multiple emails by their Network Message ID's:

```kql
let nwmids = dynamic([
"NetworkMessageID1", 
"NetworkMessageID2", 
"NetworkMessageID3", 
"NetworkMessageID4"]);
EmailEvents
| where NetworkMessageID in (nwmids)

```

### Sentinel Queries

To find a user's or multiple users' risk level during sign-in:

```kql
let AADUSERIDs = dynamic([
  "AAD ID1",
  "AAD ID2",
  "AAD ID3"]);
SigninLogs
| where UserId in (AADUSERIDs) and RiskLevelDuringSignIn !~ "none"
```

To find if a host made a connection to an external host:

```kql
CommonSecurityLog
// here you provide the domain name to the URL the user tried to access
| where DestinationHostName contains "external_host"
// here you provide the source device by name
| where SourceHostName contains "internal_host"
// you can also provide the source IP of the src host making the outbound connection
//| Where SourceIP contains "source_ip"
```

* * *

[Back to Top](#how-to-build-your-own-kql-basic)
