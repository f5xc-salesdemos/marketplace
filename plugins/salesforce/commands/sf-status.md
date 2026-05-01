---
description: Check Salesforce org connection status and list authenticated orgs
---

Delegate to the cli-operator agent to check Salesforce connectivity.

## Delegation

```
Agent(
  subagent_type="salesforce:cli-operator",
  description="Check Salesforce org status",
  prompt="Check Salesforce org connectivity:\n\n1. Verify sf CLI is installed: sf --version\n2. List authenticated orgs: sf org list --json\n3. For the default org (if set), run: sf org display --json\n4. Report:\n   - sf CLI version\n   - Number of authenticated orgs\n   - Default org alias, username, instance URL, connected status\n   - API version\n5. If no orgs are authenticated, suggest using /sf-login"
)
```
