---
name: cloud-status
description: >-
  Cloud service status monitoring and operational intelligence via Atlassian
  Statuspage.io API. Activates when the user asks about cloud status, service
  health, F5 status, active incidents, outages, degraded services, maintenance
  windows, component status, operational status, or requests a status briefing.
  Also activates for: "is [service] up", "any outages", "what's down",
  "status report for stakeholders", "executive status summary". Delegates
  all API calls to the status-operator agent — never runs cURL in the main
  session.
user-invocable: false
compatibility: Requires cURL, jq, and network access to the Statuspage.io API
---

# Cloud Status Skill

Status monitoring via Statuspage.io public API v2.

**Default:** F5 Distributed Cloud at `https://www.f5cloudstatus.com`
**Generic:** Set `STATUSPAGE_URL` env var for any Statuspage.io page.

## Intent Routing

| User Intent | Operation |
| ---------------------------------------------------- | --------------------- |
| "What's the overall status?" / "Is everything OK?" | `overall-status` |
| "Show me all components" / "What services exist?" | `list-components` |
| "Is [service] healthy?" / "Status of [component]" | `check-component` |
| "Any active incidents?" / "Current outages?" | `active-incidents` |
| "Show recent incidents" / "Incident history" | `recent-incidents` |
| "Scheduled maintenance?" / "Maintenance windows?" | `maintenance` |
| "Full status briefing" / "What's going on?" | `full-briefing` |
| "Search for [query]" | `search` |
| "Stakeholder report" / "Executive summary" | `stakeholder-report` |

### Command Argument Mapping (`/cloud-status [arg]`)

| Argument | Operation |
| ------------------ | --------------------- |
| (none) | `full-briefing` |
| `status` | `overall-status` |
| `incidents` | `active-incidents` |
| `maintenance` | `maintenance` |
| `briefing` | `full-briefing` |
| `search <query>` | `search` |
| `components` | `list-components` |

## Delegation

Match the operation, then dispatch the agent with the EXACT prompt below.
Replace `$BASE` with `${STATUSPAGE_URL:-https://www.f5cloudstatus.com}/api/v2`.

### overall-status

```
Agent(
  subagent_type="f5xc-cloudstatus:status-operator",
  description="overall cloud status check",
  prompt="Run this single command:\n\ncurl -sL --connect-timeout 10 --max-time 15 '$BASE/status.json' | jq '{page: .page.name, indicator: .status.indicator, description: .status.description, updated_at: .page.updated_at}'\n\nFormat as:\n## Cloud Status — [page]\n**[updated_at]** | **Status:** [emoji] [description]\n\nEmoji: none=✅ minor=⚠️ major=🟠 critical=🔴\nIf not operational, add one sentence about the current state."
)
```

### list-components

```
Agent(
  subagent_type="f5xc-cloudstatus:status-operator",
  description="list cloud status components",
  prompt="Run this single command:\n\ncurl -sL --connect-timeout 10 --max-time 15 '$BASE/components.json' | jq '(.components | map(select(.group == true)) | map({(.id): .name}) | add // {}) as $groups | [.components[] | select(.group == false or .group == null) | {name, status, group: (if .group_id then ($groups[.group_id] // \"Ungrouped\") else \"Ungrouped\" end)}] | group_by(.group) | map({group: .[0].group, total: length, operational: [.[] | select(.status == \"operational\")] | length, degraded: [.[] | select(.status != \"operational\") | .name]})'\n\nFormat as a table grouped by group name. Include summary counts.\n## Cloud Status Report — Components\n**Generated:** [timestamp]\n\n### Components by Group\n| Group | Total | Operational | Degraded |\n[one row per group]\n\n### Summary\n[total] components across [N] groups."
)
```

### check-component

```
Agent(
  subagent_type="f5xc-cloudstatus:status-operator",
  description="check [COMPONENT] status",
  prompt="Run this single command:\n\ncurl -sL --connect-timeout 10 --max-time 15 '$BASE/components.json' | jq --arg n 'COMPONENT_NAME' '[.components[] | select(.name | ascii_downcase | contains($n | ascii_downcase)) | {name, status, description}]'\n\nIf results are empty, report: No component matching 'COMPONENT_NAME' found.\nOtherwise format as:\n## Cloud Status — [component name]\n**Status:** [emoji] [status]\n**Description:** [description or 'N/A']"
)
```

### active-incidents

```
Agent(
  subagent_type="f5xc-cloudstatus:status-operator",
  description="active incidents check",
  prompt="Run this single command:\n\ncurl -sL --connect-timeout 10 --max-time 15 '$BASE/incidents/unresolved.json' | jq '[.incidents[] | {name, status, impact, created_at, shortlink, duration_h: (((now - (.created_at | sub(\"\\\\.[0-9]+\"; \"\") | fromdateiso8601)) / 3600) | floor), latest_update: (.incident_updates[0].body // \"No updates\")[0:300], affected: [(.components // [])[] | .name]}]'\n\nFormat as:\n## Cloud Status Report — Active Incidents\n**Generated:** [timestamp]\n\n### Active Incidents ([count])\n| Severity | Service | Status | Duration | Latest Update |\n[one row per incident]\n\nIf no incidents, report: ✅ No active incidents."
)
```

### recent-incidents

```
Agent(
  subagent_type="f5xc-cloudstatus:status-operator",
  description="recent incidents",
  prompt="Run this single command:\n\ncurl -sL --connect-timeout 10 --max-time 15 '$BASE/incidents.json' | jq '[.incidents[] | {name, status, impact, created_at, resolved_at, shortlink, latest_update: (.incident_updates[0].body // \"No updates\")[0:200], affected: [(.components // [])[] | .name]}]'\n\nApply any user-specified filters: FILTERS\n\nFormat as:\n## Cloud Status Report — Recent Incidents\n**Generated:** [timestamp]\n\n### Incidents ([count])\n| Severity | Service | Status | Created | Resolved |\n[one row per incident]\n\n### Summary\n[count] incidents in the last [window]."
)
```

### maintenance

```
Agent(
  subagent_type="f5xc-cloudstatus:status-operator",
  description="maintenance windows",
  prompt="Run these two commands:\n\ncurl -sL --connect-timeout 10 --max-time 15 '$BASE/scheduled-maintenances/upcoming.json' | jq '[.scheduled_maintenances[] | {name, status, impact, scheduled_for, scheduled_until, affected: [(.components // [])[] | .name]}]'\n\ncurl -sL --connect-timeout 10 --max-time 15 '$BASE/scheduled-maintenances/active.json' | jq '[.scheduled_maintenances[] | {name, status, impact, scheduled_for, scheduled_until, affected: [(.components // [])[] | .name]}]'\n\nFormat as:\n## Cloud Status Report — Maintenance\n**Generated:** [timestamp]\n\n### Upcoming ([count])\n| Service | Scheduled | Until | Impact |\n\n### Active ([count])\n| Service | Status | Until | Impact |\n\nIf both empty: ✅ No scheduled or active maintenance."
)
```

### full-briefing

```
Agent(
  subagent_type="f5xc-cloudstatus:status-operator",
  description="full cloud status briefing",
  prompt="Run these three commands in a SINGLE Bash call:\n\nBASE='$BASE'\necho '===STATUS===' && curl -sL --connect-timeout 10 --max-time 15 \"${BASE}/summary.json\" | jq '{page: .page.name, status: .status, incidents: [.incidents[] | {name, status, impact, created_at, duration_h: (((now - (.created_at | sub(\"\\\\.[0-9]+\"; \"\") | fromdateiso8601)) / 3600) | floor), latest_update: (.incident_updates[0].body // \"None\")[0:300], affected: [(.components // [])[] | .name]}], maintenance: [.scheduled_maintenances[] | {name, status, impact, scheduled_for, scheduled_until, affected: [(.components // [])[] | .name]}]}' && echo '===COMPONENTS===' && curl -sL --connect-timeout 10 --max-time 15 \"${BASE}/components.json\" | jq '(.components | map(select(.group == true)) | map({(.id): .name}) | add // {}) as $groups | [.components[] | select(.group == false or .group == null) | {name, status, group: (if .group_id then ($groups[.group_id] // \"Ungrouped\") else \"Ungrouped\" end)}] | group_by(.group) | map({group: .[0].group, total: length, operational: [.[] | select(.status == \"operational\")] | length, degraded: [.[] | select(.status != \"operational\") | .name]})' && echo '===TRENDS===' && curl -sL --connect-timeout 10 --max-time 15 \"${BASE}/incidents.json\" | jq '[.incidents[] | {name, impact, created_at, resolved_at: (.resolved_at // null), affected: [(.components // [])[] | .name]}]'\n\nFormat as a Full Intelligence Report:\n## Cloud Status Report — [page]\n**Generated:** [timestamp]\n**Overall Status:** [emoji] [indicator] — [description]\n\n### Active Incidents ([N])\n| Severity | Service | Status | Duration | Latest Update |\n\n### Upcoming Maintenance ([N])\n| Service | Scheduled | Until | Impact |\n\n### Component Health\n| Group | Total | Operational | Degraded |\n\n### Analysis\nFrom the trends data: flag any component with 3+ incidents in 7 days as WARNING.\nNote if any incidents overlap maintenance windows (created_at within 2h of scheduled_for).\n\n### Recommendations\n[actionable items based on findings]\n\nF5 XC regions: Services=Global, North/South America PoPs=Americas, Europe/Middle East PoPs=EMEA, Asia/Oceania PoPs=APAC, Legacy groups=low impact."
)
```

### search

```
Agent(
  subagent_type="f5xc-cloudstatus:status-operator",
  description="search cloud status for QUERY",
  prompt="Run these three commands in a SINGLE Bash call:\n\nBASE='$BASE'\nQUERY='SEARCH_QUERY'\necho '===COMPONENTS===' && curl -sL --connect-timeout 10 --max-time 15 \"${BASE}/components.json\" | jq --arg q \"$QUERY\" '($q | ascii_downcase) as $ql | [.components[] | select(.name | ascii_downcase | contains($ql)) | {name, status}]' && echo '===INCIDENTS===' && curl -sL --connect-timeout 10 --max-time 15 \"${BASE}/incidents.json\" | jq --arg q \"$QUERY\" '($q | ascii_downcase) as $ql | [.incidents[] | select((.name | ascii_downcase | contains($ql)) or ((.incident_updates // []) | any(.body | ascii_downcase | contains($ql)))) | {name, status, impact, created_at}]' && echo '===MAINTENANCES===' && curl -sL --connect-timeout 10 --max-time 15 \"${BASE}/scheduled-maintenances.json\" | jq --arg q \"$QUERY\" '($q | ascii_downcase) as $ql | [.scheduled_maintenances[] | select((.name | ascii_downcase | contains($ql)) or ((.incident_updates // []) | any(.body | ascii_downcase | contains($ql)))) | {name, status, scheduled_for}]'\n\nFormat as:\n## Cloud Status — Search: 'SEARCH_QUERY'\n**Generated:** [timestamp]\n\n### Matching Components ([N])\n### Matching Incidents ([N])\n### Matching Maintenances ([N])\n\nIf all empty: No results matching 'SEARCH_QUERY'."
)
```

### stakeholder-report

Use the same prompt as `full-briefing` but append:

```
Also append a stakeholder communication block:

F5 Distributed Cloud Status Update — [date/time UTC]
Current Status: [indicator] — [description]
Affected Services: [names from active incidents]
Customer Impact: [assessment — Services=control plane, PoPs=regional traffic, DNS=critical for new connections]
Status Page: https://www.f5cloudstatus.com
Estimated Resolution: [from latest incident update, or "Monitoring — no ETA"]

Tone: factual, calm, non-alarmist. Avoid "emergency", "disaster", "catastrophic".
```

## After Delegation

Present the agent's report to the user as-is. If the user asks follow-up
questions, determine the new operation and delegate again.
