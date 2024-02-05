# Show M365 audit log user information


### Count the number of events

```
from logs-o365*
| where user.id == "USER UPN" and event.action in ("UserLoginFailed", "UserLoggedIn")
| STATS COUNT(*) BY user.id
```

### Count amount of sucessful and failed logins for user
```
from logs-o365*
| where user.id == "USER UPN" and event.action in ("UserLoginFailed", "UserLoggedIn")
| STATS COUNT(*) BY event.action
```

### Show all succeful and failed logins for a user
This query finds all successful and failed logins and groups the findings by source.ip, source.geo.country_name, source.as.organization.name and event.action

```
from logs-o365*
| where user.id == "USER UPN" and event.action in ("UserLoginFailed", "UserLoggedIn")
| STATS COUNT(*) BY source.ip, source.geo.country_name, source.as.organization.name, event.action

```