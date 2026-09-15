---
type: runbook
system: "<% tp.system.prompt('System (e.g. live-trader, data-pipeline)') %>"
created: <% tp.date.now("YYYY-MM-DD") %>
updated: <% tp.date.now("YYYY-MM-DD") %>
tags: [runbook, live]
---

# Runbook: <% tp.file.title %>

**System:** `= this.system`

## When to use this
Trigger condition.

## Procedure
1. 
2. 

## Kill switch / rollback
Exact command / steps to stop safely.

## Verification
How to confirm the system is healthy again.

## Escalation
Who/what if this doesn't work.
