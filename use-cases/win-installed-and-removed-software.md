# Detect installed and removed Software on Windows

## Use-case
Find all installed and/or removed software on Windows. 

## ESQL Queries

### Windows Software Installed
The following query searches for Windows Application logs with Event ID 1033, Software Installed, and GROKs the message and extracts product, version, vendor and language and status information from the log event: 

```
FROM logs-system.application-default* |
WHERE event.code=="1033" and event.provider=="MsiInstaller" |
GROK message "Windows Installer installed the product. Product Name: %{GREEDYDATA:product_name}. Product Version: %{GREEDYDATA:product_version}. Product Language: %{NUMBER:product_language}. Manufacturer: %{GREEDYDATA:product_vendor}. Installation success or error status: %{NUMBER:status}." |
EVAL alert_name = CONCAT("Software Installed on System: ", product_name, "(", product_vendor, ") was installed on host ", host.hostname, " by user ", winlog.user.name) |
STATS count = count(*) by host.hostname, winlog.user.name, product_name, product_version, product_vendor, status, alert_name
```

**Event Log:**
```
Windows Installer installed the product. Product Name: WithSecure™ Elements Agent. Product Version: 24.2.187.0. Product Language: 1033. Manufacturer: WithSecure Corporation. Installation success or error status: 0.
```

### Windows Software Removed
The following query searches for Windows Application logs with Event ID 1034, Software Removed, and GROKs the message and extracts product, version, vendor and language and status information from the log event: 

```
FROM logs-system.application-default* |
WHERE event.code=="1034" and event.provider=="MsiInstaller" |
GROK message "Windows Installer removed the product. Product Name: %{GREEDYDATA:product_name}. Product Version: %{GREEDYDATA:product_version}. Product Language: %{NUMBER:product_language}. Manufacturer: %{GREEDYDATA:product_vendor}. Removal success or error status: %{NUMBER:status}." |
EVAL alert_name = CONCAT("Software Removed from System: ", product_name, "(", product_vendor, ") was removed from host ", host.hostname, " by user ", winlog.user.name) |
STATS count = count(*) by host.hostname, winlog.user.name, product_name, product_version, product_vendor, status, alert_name
```

**Event Log:**
```
Windows Installer removed the product. Product Name: elastic-agent-8.10.4-windows-x86_64. Product Version: 8.10.4.0. Product Language: 1033. Manufacturer: SofectaLabs. Removal success or error status: 0.
```

## References

- Event ID 1033: https://kb.eventtracker.com/evtpass/evtpages/EventId_1033_MsiInstaller_63308.asp
- Event ID 1034: https://kb.eventtracker.com/evtpass/evtpages/EventId_1034_MsiInstaller_63315.asp
