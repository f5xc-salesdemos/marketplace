---
description: Authenticate to a Salesforce org using container-adapted auth methods (JWT, access-token, SFDX URL, or web login)
argument-hint: "[org-alias]"
---

Delegate to the cli-operator agent to handle Salesforce authentication.

## Delegation

```
Agent(
  subagent_type="salesforce:cli-operator",
  description="Authenticate to Salesforce org",
  prompt="Help the user authenticate to a Salesforce org. The user-provided alias is: $ARGUMENTS (use 'my-dev-org' if empty).\n\n1. Run: sf org list --json\n2. If orgs exist, report them and ask if the user wants to use an existing org or add a new one.\n3. For a new org, check credentials in order and use the first fully satisfied option:\n   - SF_ACCESS_TOKEN + SF_ORG_INSTANCE_URL both set → sf org login access-token --instance-url $SF_ORG_INSTANCE_URL --no-prompt\n   - SF_JWT_KEY_FILE + SF_CLIENT_ID + SF_USERNAME all set → sf org login jwt\n   - SFDX_AUTH_URL set → echo $SFDX_AUTH_URL | sf org login sfdx-url --sfdx-url-stdin\n   - None fully satisfied → suggest sf org login web (requires browser/VNC)\n   Do NOT choose an option unless all required env vars for that option are present.\n4. Use the alias from above in --alias flag.\n5. After auth, run sf org display --target-org <alias> --json to confirm the newly authenticated org specifically.\n6. Never echo tokens or auth URLs in output."
)
```

## After Authentication

Report the result:

- **Connected:** Show org alias, username, instance URL, and API version
- **Failed:** Show the error and suggest checking credentials or trying a different auth method
- **No credentials:** Explain how to set up environment variables for headless auth
