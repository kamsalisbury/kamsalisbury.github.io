I have listed these Azure Log Analytics [Kusto Language](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/concepts/) Queries here to get you started fast with Log Analytics in Azure!

*Note:* The Free Tier of Log Analytics Workspace has been retired but is still grandfathered for subscriptions that had the free tier in the past. You have to specifically select the free tier and acknowledge the warning concerning the 500MB daily data ingestion cap and the seven day log data retention limit. Otherwise, the Azure subscription will have to select a tier of service and retention limit. I recommend reviewing the tier of service options closely to avoid unnecessary charges to the subscription. 

Each Kusto query shown below is formatted for easy copy-n-pasting. I recommend retaining the leading comments so that months from now it is documented what the query does.

```
// List Computers that have not sent an agent check-in "heartbeat"  for X hours. Heartbeat 
| summarize LastHeartbeat=max(TimeGenerated) by Computer 
| where LastHeartbeat < ago(1h)
```
```
// Monitor on premise backup failures and issues
// In the Azure Log Analytics Workspace, Advanced Settings, Data, Windows Event Logs, add the Microsoft-Windows-Backup event log source to list all Microsoft-Windows-Backup warning or errors
Event
| where EventLog == "Microsoft-Windows-Backup"
| where EventLevelName == "Warning" or EventLevel == "Error"
```
```
// In the Azure Log Analytics Workspace, Advanced Settings, Data, WIndows Performance Counters, Add the Logical Disk % Free Space Performance Counter source
// List all logical disk disk % free space less than or equal to specified value 
Perf
| where Computer == "Server1" or Computer == "Server2" or Computer == "DC2.domain.tld" or Computer == "DC1.domain.tld"
| where CounterName == "% Free Space" | where InstanceName == "C:" or InstanceName == "E:"
| where CounterValue <= 10
```
```
// Display the latest Web App HTTP Logs, requires the Web App, Diagnostics Settings, are set to "Send to  Log Analytics"
AppServiceHTTPLogs
| sort by TimeGenerated desc
```
```
// Display Time service and synchronization related warning and errors from the System event log
Event
| where EventLog == "System"
| where Source == "Microsoft-Windows-Time-Service"
| where EventLevelName == "Warning" or EventLevel == "Error"
| where EventID == "24"
```
```
// Query FiOS remote security notices sent to syslog
Syslog
| where HostName == "gateway"
| where Facility == "local5"
```
```
// Query syslog events for blocked packets
Syslog
| where HostName == "gateway"
| extend Message = split(SyslogMessage, ' ') 
| extend SourceIP = tostring(Message[4])
| extend TypeOfService = tostring(Message[7])
| extend Protocol = tostring(Message[11])
| extend DestPort = tostring(Message[13])
| parse SourceIP with * "SRC=" SRC
| parse TypeOfService with * "TOS=" TOS
| parse Protocol with * "PROTO=" PROTO
| parse DestPort with * "DPT=" DPT
| summarize count() by SRC 
```
```
// Query syslog events for FiOS G3100 IPTABLES blocked packets
Syslog
| where HostName == "G3100"
| where Facility == "kern"
| extend Message = split(SyslogMessage, ' ')
| extend EventType = tostring(Message[3])
| extend SourceIP = tostring(Message[8])
| extend TypeOfService = tostring(Message[11])
| extend Protocol = tostring(Message[15])
| extend DestPort = tostring(Message[17])
| parse SourceIP with * "SRC=" SRC
| parse TypeOfService with * "TOS=" TOS
| parse Protocol with * "PROTO=" PROTO
| parse DestPort with * "DPT=" DPT
| where EventType == "remoteGUI"
```
```
// Query syslog events for FiOS G3100 IPTABLES blocked packets, summary
Syslog
| where HostName == "G3100"
| where Facility == "kern"
| extend Message = split(SyslogMessage, ' ')
| extend EventType = tostring(Message[3])
| extend SourceIP = tostring(Message[8])
| extend TypeOfService = tostring(Message[11])
| extend Protocol = tostring(Message[15])
| extend DestPort = tostring(Message[17])
| parse SourceIP with * "SRC=" SRC
| parse TypeOfService with * "TOS=" TOS
| parse Protocol with * "PROTO=" PROTO
| parse DestPort with * "DPT=" DPT
| where EventType == "remoteGUI"
| summarize count() by SRC 
```
```
// Query Azure Activity Log connected to Log Analytics
// Display Resource Group created
AzureActivity
| where OperationName == "Update resource group"
| where ActivityStatus == "Succeeded"
| where ActivitySubstatus == "Created (HTTP Status Code: 201)"
```
```
// Query syslog events for alert level events, requires agent on linux system
Syslog
| where SeverityLevel == "alert" 
```
```
// Query syslog events for failed sudo and other authentication alert level activities
Syslog
| where Facility == "authpriv" 
| where SeverityLevel == "alert" 
```
