# Find the DNS name from DNS query logs for an IP 

## Use-case
Find the DNS name for an IP-address seen in logs. For example, if there are any Threat Intelligence matches for IP-addresses, we want to check where the IP was used and what the DNS query results where for that IP. 

## ESQL query

### Show all events with DNS question resolving to IP
```
FROM logs-endpoint.events.network-*
| where event.action == "lookup_result" and event.outcome == "success"
| dissect message "%{content}::ffff:%{dns.resolved_ip};"
| where dns.resolved_ip is "INSERT IP HERE"
| keep @timestamp,host.name,dns.question.name,dns.question.type,dns.resolved_ip, message

```

### Show all dns results grouped by dns question result
```
FROM logs-endpoint.events.network-*
| where event.action == "lookup_result" and event.outcome == "success"
| dissect message "%{content}::ffff:%{dns.resolved_ip};"
| where dns.resolved_ip is "INSERT IP HERE"
| keep @timestamp,host.name,dns.question.name,dns.question.type,dns.resolved_ip, message
| STATS COUNT(*) BY dns.question.name
```

### Show all dns results grouped by dns question result and hostname
```
FROM logs-endpoint.events.network-*
| where event.action == "lookup_result" and event.outcome == "success"
| dissect message "%{content}::ffff:%{dns.resolved_ip};"
| where dns.resolved_ip is "INSERT IP HERE"
| keep @timestamp,host.name,dns.question.name,dns.question.type,dns.resolved_ip, message
| STATS COUNT(*) BY dns.question.name, host.name

```