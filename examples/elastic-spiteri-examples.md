Some example ESQL queries to get started...

### Aggregating, manipulating and transforming process events
```
from logs-endpoint.events.process-default [metadata _index,_id] 
| where event.category == "process"
| eval process_args_string = MV_CONCAT(TO_STRING(process.args), ", ")
| where length(process.name) > 10 and user.name is not null
| stats counted = count(*) by _index,host.name,user.name,process.command_line,process_args_string,host.os.family
| eval cmd_length = length(process.command_line)
| grok process.command_line "\"%{DATA:drive}:\\\\%{DATA:path}\"%{SPACE}%{GREEDYDATA:args}"
| dissect process.command_line "%{path_macos} %{args_macos}"
| eval drive = case (  
    host.os.family == "macos", "N/A",
    host.os.family == "windows", drive
  )
| eval path = case (  
    host.os.family == "macos", path_macos,
    host.os.family == "windows", path
)
| eval args = case (  
    host.os.family == "macos", args_macos,
    host.os.family == "windows", args
)
| drop path_macos, args_macos
| where process.command_line like "*Google*Update*"
| where host.os.family rlike "win.*|ma.*"
| where cmd_length > 0
| eval frozen = case (
    _index like "partial*", "true", "false"
)
| sort counted desc
| limit 10000
| keep _index,frozen,counted,host.name,host.os.family,user.name,cmd_length,process.command_line,process_args_string,drive,path,args
```

### Aggregating and enriching windows event logs
```
from logs-*
| where event.code is not null
| stats event_code_count = count(event.code) by event.code,host.name
| enrich win_events on event.code with EVENT_DESCRIPTION
| where EVENT_DESCRIPTION is not null and host.name is not null
| rename EVENT_DESCRIPTION as event.description
| sort event_code_count desc
| keep event_code_count,event.code,host.name,event.description
```

### summing outbound traffic from curl.exe
```
from logs-endpoint
| where process.name == "curl.exe"
| stats bytes = sum(destination.bytes) by destination.address
| eval kb =  bytes/1024
| sort kb desc
| limit 10
| keep kb,destination.address
```

### Manipulating DNS logs to find a high number of unique dns queries per registered domain
```
from logs-*
| grok dns.question.name "%{DATA}\\.%{GREEDYDATA:dns.question.registered_domain:string}"
| stats unique_queries = count_distinct(dns.question.name) by dns.question.registered_domain, process.name
| where unique_queries > 10
| sort unique_queries desc
| rename unique_queries as `Unique Queries`, dns.question.registered_domain as `Registered Domain`, process.name as `Process`
```

### This query counts the number of outbound connections made to external IP addresses broken down by user and host. 

It uses a case statement to add a new field called "follow_up". If the sum of connections is greater or equal to 100, the value of the follow_up field is set to true. It also enriches the user names with their respective ldap groups.

```
FROM logs-*
| WHERE NOT CIDR_MATCH(destination.ip, "10.0.0.0/8", "172.16.0.0/12", "192.168.0.0/16")
| STATS destcount = COUNT(destination.ip) by user.name, host.name
| ENRICH ldap_lookup_new ON user.name
| WHERE group.name IS NOT NULL
| EVAL follow_up = CASE(
    destcount >= 100, "true",
     "false")
| SORT destcount desc
| KEEP destcount, host.name, user.name, group.name, follow_up
```

### This query searches for real users attempting to transfer data greater than 1gb from multiple different hosts on a given day.
```
FROM logs-endpoint.events.network-default
| WHERE NOT CIDR_MATCH(destination.ip, "10.0.0.0/8", "172.16.0.0/12", "192.168.0.0/16") and destination.as.organization.name IS NOT NULL
| EVAL date = DATE_TRUNC(1 day,@timestamp)
| EVAL day = DATE_FORMAT("YYYY-MM-dd", date)
| STATS destcount = COUNT(*), hostcount = count_distinct(host.name), total_bytes = SUM(source.bytes) by user.name,process.name, day, destination.as.organization.name
| ENRICH users ON user.name
| WHERE user.name IS NOT NULL and group.name IS NOT NULL
| EVAL dbl = TO_DOUBLE(total_bytes)
| EVAL total_gb = ROUND(dbl / 1024 / 1024 / 1024,2)
| WHERE total_gb > 0
| EVAL follow_up = CASE(
    total_gb > 1 and hostcount >= 2, "true",
     "false")
| WHERE follow_up == "true" and group.name != "system_users"
| SORT total_gb desc
| LIMIT 10
| KEEP day,destcount,hostcount,destination.as.organization.name,process.name,total_gb,user.name,group.name,follow_up
```