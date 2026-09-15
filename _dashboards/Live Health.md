---
type: dashboard
tags: [dashboard]
---

# Live Health

## Open incidents
```dataview
TABLE severity, status, occurred
FROM "07_Live/Incidents"
WHERE status != "resolved"
SORT severity ASC
```

## Recent incidents (all)
```dataview
TABLE severity, status, occurred
FROM "07_Live/Incidents"
SORT occurred DESC
LIMIT 10
```

## Runbooks
```dataview
LIST FROM "07_Live/Runbooks"
```

## Production models live now
```dataview
LIST FROM "04_Models" WHERE stage = "production"
```
