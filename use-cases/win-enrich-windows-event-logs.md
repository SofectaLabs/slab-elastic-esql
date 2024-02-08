# Enrich Windows Event logs with the Event ID Description

## Use Case
This ESQL query can be used to enrich windows event logs with the Event Log Description that is currently mising from Windows logs and from Elastic internal enrichments. In some environments the Windows default language is non-English, which produces log events in the local languange.

## Pre-requisites:
The enrichment source must be uploaded to Kibana so that the enrichments can be mapped properly: https://github.com/PerryvandenHondel/windows-event-id-list-csv/blob/master/windows-event-id.csv
Upload the file to a Elastic index and create a enrichment policy. 

## ESQL Query
```
FROM logs-endpoint
| WHERE event.code is not null
| STATS event_code_count = count(event.code) by event.code,host.name
| ENRICH win_events on event.code with EVENT_DESCRIPTION
| WHERE EVENT_DESCRIPTION is not null and host.name is not null
| RENAME EVENT_DESCRIPTION as event.description
| SORT event_code_count desc
| KEEP event_code_count,event.code,host.name,event.description
```
