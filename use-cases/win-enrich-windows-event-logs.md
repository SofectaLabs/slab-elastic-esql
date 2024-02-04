# Enrich Windows Event logs with the Event ID Description

## Use Case
This ESQL query can be used to enrich windows event logs with the Event Log Description that is currently mising from Windows logs and from Elastic internal enrichments. In some environments the Windows default language is non-English, which produces log events in the local languange.

Enrichment source: 
https://github.com/PerryvandenHondel/windows-event-id-list-csv/blob/master/windows-event-id.csv

## Pre-requisites:
The enrichment source must be uploaded to Kibana so that the enrichments can be mapped properly. 

## ESQL Query
```
from logs-endpoint
| where event.code is not null
| stats event_code_count = count(event.code) by event.code,host.name
| enrich win_events on event.code with EVENT_DESCRIPTION
| where EVENT_DESCRIPTION is not null and host.name is not null
| rename EVENT_DESCRIPTION as event.description
| sort event_code_count desc
| keep event_code_count,event.code,host.name,event.description

```