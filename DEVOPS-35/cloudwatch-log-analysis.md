CloudWatch Log Analysis
Introduction

AWS CloudWatch Logs allows DevOps engineers to collect, store, search, and analyze application and infrastructure logs.

CloudWatch Logs Insights provides a query language that can be used to search logs and identify errors, warnings, exceptions, timeouts, and connection problems.

This task focuses on:

Searching logs
Filtering logs
Sorting results
Limiting results
Searching for common error patterns
Understanding CloudWatch Logs Insights queries
1. What is CloudWatch Logs?

CloudWatch Logs is an AWS service used to collect and store log data.

Typical log sources include:

EC2 instances
ECS containers
Lambda functions
Applications
Load balancers
AWS services

Basic architecture:

Application
     │
     ▼
Generate Logs
     │
     ▼
CloudWatch Log Group
     │
     ▼
Log Streams
     │
     ▼
CloudWatch Logs Insights
     │
     ▼
Search / Filter / Analyze
2. Important CloudWatch Log Concepts
Log Group

A Log Group is a logical container for logs.

Example:

/ecs/devops-training-app

A log group can contain multiple log streams.

Log Stream

A Log Stream is a sequence of log events from a specific source.

For example:

ECS Task 1
    │
    ▼
Log Stream

Another task may have:

ECS Task 2
    │
    ▼
Log Stream

Both streams can belong to the same Log Group.

Log Event

A log event is an individual log entry.

Example:

2026-09-28 10:15:30 ERROR Database connection failed
3. What is CloudWatch Logs Insights?

CloudWatch Logs Insights is a tool for interactively searching and analyzing log data.

It allows you to:

Select fields
Filter logs
Search for text
Sort results
Limit results
Count events
Group results
Analyze timestamps

Example:

fields @timestamp, @message
| filter @message like /ERROR/
| sort @timestamp desc
| limit 20
4. Understanding the Basic Query

Consider:

fields @timestamp, @message
| filter @message like /ERROR/
| sort @timestamp desc
| limit 20

Let's understand each line.

fields
fields @timestamp, @message

fields specifies which fields should be displayed.

Two commonly used fields are:

@timestamp
@message
@timestamp

The time when the log event was generated.

@message

The actual log message.

5. filter

Example:

filter @message like /ERROR/

filter selects only log events that match a condition.

In this example, only messages containing:

ERROR

are returned.

6. sort

Example:

sort @timestamp desc

This sorts the logs by timestamp.

desc means descending order.

Therefore:

Newest
  ↓
Older

For oldest first:

sort @timestamp asc
7. limit

Example:

limit 20

This limits the query output to 20 results.

This is useful when a log group contains thousands or millions of events.

8. Search for ERROR

Use:

fields @timestamp, @message
| filter @message like /ERROR/
| sort @timestamp desc
| limit 20

Purpose:

Find application errors

Example result:

2026-09-28 10:20:01 ERROR Database connection failed
2026-09-28 10:19:45 ERROR API request failed
2026-09-28 10:18:32 ERROR Payment service unavailable
9. Search for WARNING

Query:

fields @timestamp, @message
| filter @message like /WARNING/
| sort @timestamp desc
| limit 20

Purpose:

Find warning messages

Example:

WARNING High memory usage
WARNING Retry attempt 2
WARNING API response slow
10. Search for Exception

Query:

fields @timestamp, @message
| filter @message like /Exception/
| sort @timestamp desc
| limit 20

Purpose:

Find application exceptions

Example:

NullPointerException
DatabaseException
ConnectionException
TimeoutException
11. Search for Timeout

Query:

fields @timestamp, @message
| filter @message like /Timeout/
| sort @timestamp desc
| limit 20

Purpose:

Find timeout-related problems

Example:

Database Timeout
Request Timeout
Connection Timeout
API Timeout
12. Search for Connection

Query:

fields @timestamp, @message
| filter @message like /Connection/
| sort @timestamp desc
| limit 20

Purpose:

Find connection-related events

Example:

Connection refused
Connection timeout
Database connection failed
Connection reset
13. Case-Insensitive Searching

Sometimes logs contain:

ERROR
error
Error

To search without depending on capitalization, use a regular expression.

Example:

fields @timestamp, @message
| filter @message like /(?i)error/
| sort @timestamp desc
| limit 20

The (?i) makes the pattern case-insensitive.

14. Search Multiple Error Types

You can search for multiple patterns.

Example:

fields @timestamp, @message
| filter @message like /ERROR/
   or @message like /Exception/
   or @message like /Timeout/
| sort @timestamp desc
| limit 50

This helps investigate several types of failures together.

15. Search ERROR and Connection

Example:

fields @timestamp, @message
| filter @message like /ERROR/
   and @message like /Connection/
| sort @timestamp desc
| limit 20

This searches for messages containing both:

ERROR

and:

Connection

Example:

ERROR Database Connection failed
16. Search for HTTP 500 Errors

Application logs may contain HTTP status codes.

Query:

fields @timestamp, @message
| filter @message like /500/
| sort @timestamp desc
| limit 20

This can help locate server-side failures.

Example:

500 Internal Server Error
17. Search for HTTP 404 Errors

Query:

fields @timestamp, @message
| filter @message like /404/
| sort @timestamp desc
| limit 20

Purpose:

Find not-found errors
18. Search Recent Errors

You can restrict the time range using the CloudWatch Logs Insights time selector.

For example:

Last 15 minutes
Last 1 hour
Last 3 hours
Last 24 hours

Then execute:

fields @timestamp, @message
| filter @message like /ERROR/
| sort @timestamp desc
| limit 20

The query searches only within the selected time range.

19. Count Errors

Logs Insights can also be used to count matching events.

Example:

filter @message like /ERROR/
| stats count() as errorCount

Example result:

errorCount
----------
37

This tells us that 37 matching error events were found in the selected time range.

20. Count Errors Over Time

Example:

filter @message like /ERROR/
| stats count() as errorCount by bin(5m)

This groups errors into five-minute intervals.

Example:

10:00 → 2 errors
10:05 → 3 errors
10:10 → 15 errors
10:15 → 17 errors

This can help identify when an incident started.

21. Find the Latest Errors

Query:

fields @timestamp, @message
| filter @message like /ERROR/
| sort @timestamp desc
| limit 10

This shows the 10 most recent errors.

22. Find the Oldest Errors

Query:

fields @timestamp, @message
| filter @message like /ERROR/
| sort @timestamp asc
| limit 10

This shows the earliest matching errors in the selected time range.

23. Useful Logs Insights Commands
Command	Purpose
fields	Select fields to display
filter	Filter log events
sort	Sort results
limit	Limit number of results
stats	Calculate statistics
count()	Count log events
bin()	Group events by time
parse	Extract information from log messages
24. Common Investigation Queries
ERROR
fields @timestamp, @message
| filter @message like /ERROR/
| sort @timestamp desc
| limit 20
WARNING
fields @timestamp, @message
| filter @message like /WARNING/
| sort @timestamp desc
| limit 20
Exception
fields @timestamp, @message
| filter @message like /Exception/
| sort @timestamp desc
| limit 20
Timeout
fields @timestamp, @message
| filter @message like /Timeout/
| sort @timestamp desc
| limit 20
Connection
fields @timestamp, @message
| filter @message like /Connection/
| sort @timestamp desc
| limit 20
25. Practical Exercise

Use one of your CloudWatch log groups.

For example:

/ecs/devops-training-app

Open:

AWS Console
   ↓
CloudWatch
   ↓
Logs
   ↓
Logs Insights

Select your log group.

Then run the following queries one at a time.

Step 1 — View recent logs
fields @timestamp, @message
| sort @timestamp desc
| limit 20
Step 2 — Search ERROR
fields @timestamp, @message
| filter @message like /ERROR/
| sort @timestamp desc
| limit 20
Step 3 — Search WARNING
fields @timestamp, @message
| filter @message like /WARNING/
| sort @timestamp desc
| limit 20
Step 4 — Search Exception
fields @timestamp, @message
| filter @message like /Exception/
| sort @timestamp desc
| limit 20
Step 5 — Search Timeout
fields @timestamp, @message
| filter @message like /Timeout/
| sort @timestamp desc
| limit 20
Step 6 — Search Connection
fields @timestamp, @message
| filter @message like /Connection/
| sort @timestamp desc
| limit 20
Step 7 — Count Errors
filter @message like /ERROR/
| stats count() as errorCount
26. Troubleshooting Workflow

When investigating an application problem:

Incident
   │
   ▼
Open CloudWatch Logs
   │
   ▼
Select Log Group
   │
   ▼
Select Time Range
   │
   ▼
Search ERROR
   │
   ▼
Search Exception
   │
   ▼
Search Timeout
   │
   ▼
Search Connection
   │
   ▼
Analyze Timestamp
   │
   ▼
Identify Pattern
   │
   ▼
Investigate Root Cause
27. Example Incident Investigation

Suppose users report:

Application is slow.

First search:

ERROR

You find:

ERROR Database connection failed

Then search:

Connection

You find:

Connection timeout after 30 seconds

Then search:

Timeout

You find multiple events around the same time.

You can then compare the timestamps with:

Application deployments
Database availability
CPU utilization
Memory utilization
Network metrics
Other application logs

This demonstrates how logs can be combined with metrics and other observability data during troubleshooting.

28. Best Practices
1. Use a specific time range

Avoid searching an unnecessarily large time period.

2. Start with ERROR

Start with the most important failure indicators.

3. Check timestamps

Look for patterns around the time of the incident.

4. Search related keywords

For example:

ERROR
Exception
Timeout
Connection
5. Do not rely only on logs

Combine logs with:

Metrics
Traces
Dashboards
Alarms
6. Keep queries simple initially

Start with:

fields @timestamp, @message
| filter @message like /ERROR/

Then make the query more specific if required.

29. Observability Investigation Model
Metrics
   │
   ▼
Detect abnormal behavior
   │
   ▼
Logs
   │
   ▼
Find what happened
   │
   ▼
Traces
   │
   ▼
Find where it happened
   │
   ▼
Root Cause Investigation
Key Takeaways
CloudWatch Logs stores application and infrastructure log events.
CloudWatch Logs Insights allows interactive log analysis.
fields selects fields to display.
filter filters log events.
sort controls result order.
limit controls the number of returned results.
stats performs calculations such as counts.
bin() groups results into time periods.
Search for ERROR, WARNING, Exception, Timeout, and Connection during basic troubleshooting.
Always select an appropriate time range before investigating.
Logs should be combined with metrics, traces, dashboards, and alarms for complete observability.
Task 6 Deliverable

Create the file:

docs/cloudwatch-log-analysis.md

The completed file should contain:

CloudWatch Logs
      │
      ▼
Logs Insights
      │
      ├── ERROR
      ├── WARNING
      ├── Exception
      ├── Timeout
      └── Connection
              │
              ▼
       Investigation
              │
              ▼
         Root Cause