---
type: incident
severity: sev2        # sev1 (money/at-risk) | sev2 | sev3
status: open          # open | mitigated | resolved
occurred: <% tp.date.now("YYYY-MM-DD HH:mm") %>
git_commit: ""
tags: [incident, live]
---

# <% tp.file.title %>

**Severity:** `= this.severity` · **Status:** `= this.status` · **When:** `= this.occurred`

## What happened
Timeline of events (timestamps).

## Impact
Positions, P&L, data integrity — quantified.

## Root cause
The actual cause, not the symptom.

## Fix / mitigation
What stopped the bleeding. Link commit: https://github.com/linus-lind/Nekron/commit/`= this.git_commit`

## Prevention
- [ ] Guardrail / test / alert to add
- [ ] Runbook to update: [[ ]]
