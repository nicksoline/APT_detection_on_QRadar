# APT_detection_on_QRadar

Coding tips

Before you implement custom AQL functions, consider these items:

    Scripts are not throttled and cannot be canceled. Be careful of infinite loops and resource leaks in your code.
    Deletion of custom functions is possible, but not supported.
    QRadar parses AQL strings much more than it needs to. Expensive init_function_name implementations that are combined with the use of your function in a literal context can be expensive. Use with caution.
    The execute_function_name implementation must be thread-safe. Use the Utils.concurrent library to ensure thread safety.

SQL Structure
[SELECT *, column_name, column_name]
[FROM table_name]
[WHERE search clauses]
[GROUP BY column_reference*]
[HAVING clause]
[ORDER BY column_reference*]
[LIMIT numeric_value] 
[TIMEFRAME]

When you use a GROUP BY or ORDER BY clause to sort information, you can reference column_names from your existing SELECT statement only.

By default, if the TIMEFRAME value is not specified, the query runs against the last five minutes of Ariel data. 

Remember to use single quotation marks to specify literal values or variables and use double quotation marks for column names that contain spaces or non-ASCII characters:

Single quotation marks
    Use single quotation marks when you reference the beginning and end of a string, as shown in these examples:
    username LIKE '%User%‘
    sourceCIDR= '192.0.2.0'
    TEXT SEARCH = 'VPN Authenticated user‘
    QIDNAME(qid) AS ‘Event Name’ 

Double quotation marks
    Use double quotation marks when column names contain spaces or non-ASCII characters, as shown in these examples:
    Custom property names with spaces, such as “Account Security ID”.
    Values that have non-ASCII characters. 
    
    
    
Simple AQL queries

Table 1. Simple AQL queries Basic AQL Commands 	Comments

SELECT * FROM events LAST 10 MINUTES
Copy

	Returns all the fields from the events table that were sent in the last 10 minutes.

SELECT sourceip,destinationip FROM events 
LAST 24 HOURS
Copy

	Returns the sourceip and destinationip from the events table that were sent in the last 24 hours.

SELECT * FROM events START '2017 01 01 9:00:00' 
STOP '2017 01 01 10:20:00'
Copy

	Returns all the fields from the events table during that time interval.

SELECT * FROM events limit 5 LAST 24 HOURS
Copy

	Returns all the fields in the events table during the last 24 hours, with output limited to five results.

SELECT * FROM events ORDER BY magnitude DESC 
LAST 24 HOURS
Copy

	Returns all the fields in the events table sent in the last 24 hours, sorting the output from highest to lowest magnitude.

SELECT * FROM events WHERE magnitude >= 3 
LAST 24 HOURS
Copy

	Returns all the fields in the events table that have a magnitude that is less than three from the last 24 hours.

SELECT * FROM events WHERE sourceip = '192.0.2.0' 
AND destinationip = '198.51.100.0' START '2017 01 01 
9:00:00' STOP '2017 01 01 10:20:00'
Copy

	Returns all the fields in the events table that have the specified source IP and destination IP within the specified time period.

SELECT * FROM events WHERE INCIDR('192.0.2.0/24',
sourceip)
Copy

	Returns all the fields in the events table where the source IP address is within the specified CIDR IP range.

SELECT * FROM events WHERE username LIKE '%roul%'
Copy

	Returns all the fields in the events table where the user name contains the example string. The percentage symbols (%) indicate that the user name can match a string of zero or more characters.

SELECT * FROM events WHERE username ILIKE '%ROUL%'
Copy

	Returns all the fields in the events table where the user name contains the example string, and the results are case-insensitive. The percentage symbols (%) indicate that the user name can match a string of zero or more characters.

SELECT sourceip,category,credibility FROM events 
WHERE (severity > 3 AND category = 5018)OR 
(severity < 3 AND credibility > 8)
Copy

	Returns the sourceip, category, and credibility fields from the events table with specific severity levels, a specific category, and a specific credibility level. The AND clause allows for multiple strings of types of results that you want to have.

SELECT * FROM events WHERE TEXT SEARCH 'firewall'
Copy

	Returns all the fields from the events table that have the specified text in the output.

SELECT * FROM events WHERE username ISNOT NULL
Copy

	Returns all the fields in the events table where the username value is not null.



To use AQL in the search fields, consider the following functions:

    In the search fields on the Log Activity or Network Activity tabs, type Ctrl + Space to see the full list of AQL functions, fields, and keywords.
    Ctrl + Enter helps you create multiline AQL queries in the user interface, which makes the queries more readable.
    By using the copy (Ctrl + C) and paste (Ctrl + V) keyboard commands, you can copy directly to and from the Advanced search field. 



Table 1. Ariel Query Language categories Category 	Definition
Database 	The name of an Ariel database, or table, that you can query. The database is either events or flows.
Keyword 	Typically core SQL clauses. For example, SELECT, OR, NULL, NOT, AS, ACS (ascending), and more.
Field 	Indicates basic information that you can query from the database. Examples include Access intent, VPC ID, and domainid.
Function 	Various functions from string functions to call in more information. Functions work on all fields and databases. Examples of functions include DATEFORMAT, HOSTNAME, and LOWER. 


