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





The following table lists the supported logical and comparison operators.
Table 1. Logical and comparison operators Operator 	Description 	Example
* 	Multiplies two values and returns the result. 	

SELECT *
FROM flows 
WHERE sourceBytes * 1024 < 1 
Copy

= 	The equal to operator compares two values and returns true if they are equal. 	

SELECT * 
FROM EVENTS
WHERE sourceIP = destinationIP 
Copy

!= 	Compares two values and returns true if they are unequal. 	

SELECT *
FROM events
WHERE sourceIP
!= destinationip
Copy

< AND <= 	Compares two values and returns true if the value on the left side is less than or equal to, the value on the right side. 	

SELECT *
FROM flows 
WHERE sourceBytes < 64 
AND
destinationBytes <= 64  
Copy

> AND >= 	Compares two values and returns true if the value on the left side is greater than or equal to the value on the right side. 	

SELECT * 
FROM flows 
WHERE sourceBytes > 64 
AND
destinationBytes >= 64
Copy

/ 	Divides two values and returns the result. 	

SELECT * 
FROM flows 
WHERE sourceBytes / 8 > 64 
Copy

+ 	Adds two values and returns the result. 	

SELECT * 
FROM flows 
WHERE sourceBytes + 
destinationBytes < 64 
Copy

- 	Subtracts one value from another and returns the result. 	

SELECT * 
FROM flows 
WHERE sourceBytes - 
destinationBytes > 0
Copy

^ 	Takes a value and raises it to the specified power and returns the result. 	

SELECT * 
FROM flows 
WHERE sourceBytes ^ 2 < 256
Copy

% 	Takes the modulo of a value and returns the result. 	

SELECT * 
FROM flows 
WHERE sourceBytes % 8 == 7
Copy

AND 	Takes the left side and right side of a statement and returns true if both are true. 	

SELECT *
FROM events 
WHERE (sourceIP = destinationIP)
AND (sourcePort = destinationPort)
Copy

BETWEEN (X,Y) 	Takes in a left side and two values and returns true if the left side is between the two values. 	

SELECT * 
FROM events 
WHERE magnitude
BETWEEN 1 AND 5 
Copy

COLLATE 	Parameter to order by that allows a BCP47 language tag to collate. 	

SELECT * 
FROM EVENTS ORDER BY
sourceIP DESC COLLATE 'de-CH' 
Copy

IN 	Specifies multiple values in a WHERE clause. The IN operator is a shorthand for multiple OR conditions. 	

SELECT *
FROM EVENTS
WHERE SourceIP IN ('192.0.2.1', '::1', '198.51.100.0')
Copy

INTO 	Creates a named cursor that contains results that can be queried at a different time. 	

SELECT * FROM EVENTS INTO
 'MyCursor' WHERE....
Copy

NOT 	Takes in a statement and returns true if the statement evaluates as false. 	

SELECT * FROM EVENTS 
WHERE NOT 
(sourceIP = destinationIP) 
Copy

ILIKE 	Matches if the string passed is LIKE the passed value and is not case sensitive. Use % as a wildcard. 	

SELECT * 
FROM events WHERE userName 
ILIKE '%bob%' 
Copy

IMATCHES 	Matches if the string matches the provided regular expression and is not case sensitive. 	

SELECT * 
FROM events 
WHERE userName
IMATCHES '^.bob.$' 
Copy

LIMIT 	Limits the number of results to the provided number. 	

SELECT * 
FROM events LIMIT 100 
START '2015-10-28 10:00' 
STOP '2015-10-28 11:00'
Copy

Note Place the LIMIT clause in front of a START and STOP clause.
LIKE 	Matches if the string passed is LIKE the passed value but is case sensitive. Use % as a wildcard. 	

SELECT * 
FROM events WHERE userName 
LIKE '%bob%' 
Copy

MATCHES 	Matches if the string matches the provided regular expression. 	

SELECT * 
FROM events 
WHERE userName MATCHES
'^.bob.$' 
Copy

NOT NULL 	Takes in a value and returns true if the value is not null. 	

SELECT *
FROM events 
WHERE userName 
IS NOT NULL
Copy

OR 	Takes the left side of a statement and the right side of a statement and returns true if either side is true. 	

SELECT *
FROM events 
WHERE (sourceIP = destinationIP)
OR (sourcePort = destinationPort)
Copy

TEXT SEARCH 	

Full-text search for the passed value.

TEXT SEARCH is valid with AND operators. You can't use TEXT SEARCH with OR or other operators; otherwise, you get a syntax error.

Place TEXT SEARCH in the first position of the WHERE clause.

You can also do full-text searches by using the Quick filter in the QRadar® user interface. For information about Quick filter functions, see the IBM® QRadar User Guide.
	

SELECT * 
FROM events 
WHERE TEXT SEARCH 'firewall'
AND sourceip='192.168.1.1' 
Copy

SELECT sourceip,url 
FROM events 
WHERE TEXT SEARCH 
'download.cdn.mozilla.net' 
AND sourceip='192.168.1.1' 
START '2015-01-30 16:10:12' 
STOP '2015-02-22 17:10:22'
Copy

Examples of logical and comparative operators

    To find events that are not parsed, type the following query:

    SELECT * FROM events 
    WHERE payload = 'false'
    Copy

    To find events that return an offense and have a specific source IP address, type the following query:

    SELECT * FROM events 
    WHERE sourceIP = '192.0.2.0' 
    AND
    hasOffense = 'true'
    Copy

    To find events that include the text "firewall", type the following query:

    SELECT QIDNAME(qid) 
    AS EventName, 
    * FROM events 
    WHERE TEXT SEARCH 'firewall'
    
    
    
    
