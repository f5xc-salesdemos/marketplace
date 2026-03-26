---
name: console-auth
description: >-
  Azure SSO authentication for the F5 Distributed Cloud console.
  Handles both cached sessions (single-click re-login) and full
  MFA flow (username → password → DUO push). Use when the user
  says "log in", "authenticate", "sign in to the console", or
  when any console workflow requires an active session.
user-invocable: false
---

# Console Authentication — Azure SSO

Authenticate to the F5 Distributed Cloud console using Azure
SSO. Supports two modes automatically detected at runtime:

- **Cached session** — user previously selected "remember this
  device"; clicking "Sign in with Azure" auto-completes login
- **Full MFA** — username entry, password entry, DUO push
  notification, "Stay signed in?" confirmation

## Environment Variables

| Variable | Required | Default | Purpose |
| ---------- | ---------- | --------- | --------- |
| `F5XC_API_URL` | No | `https://f5-amer-ent.console.ves.volterra.io` | Tenant URL |
| `F5XC_SSO_USER` | Full MFA only | — | Azure AD email |
| `F5XC_CONSOLE_PASSWORD` | Full MFA only | — | Azure AD password |

## Prerequisites

- Chrome DevTools MCP server connected to the Claude Code session
- Browser instance open and accessible via MCP
- VPN connection if required by tenant network policy

## Authentication Flow

### Step 1: Read configuration

Read the tenant URL from `F5XC_API_URL`. If not set, default to
`https://f5-amer-ent.console.ves.volterra.io`.

Derive the login URL: `${F5XC_API_URL}/web/login`

### Step 2: Detect current state

Use `take_snapshot` on the current browser page.

- **Already authenticated** — URL contains `/web/workspaces`
  or snapshot shows workspace elements → skip auth, report
  success immediately
- **Login page visible** — URL contains `/web/login` or
  snapshot shows "Sign in with Azure" → proceed to Step 3
- **Other page** — navigate to the login URL, then proceed

### Step 3: Click "Sign in with Azure"

```
take_snapshot()
```

Find the Azure SSO button by text content matching
"Sign in with Azure" or "Sign in with SSO". Click it:

```
click(uid=<azure-sso-button-uid>)
```

### Step 4: Mode detection

Wait up to 15 seconds for one of these outcomes:

```
wait_for(text=["Pick an account", "Sign in to your account",
               "Enter password", "web/workspaces"],
         timeout=15000)
take_snapshot()
```

Determine the path:

- **URL contains `/web/workspaces`** → cached session
  succeeded. Jump to Step 8 (verification).
- **"Pick an account" visible** → click the account matching
  `F5XC_SSO_USER`, then monitor for auto-completion or MFA.
- **Email/username input visible** → full MFA flow. Continue.

### Step 5: Username entry (Full MFA)

```
take_snapshot()
fill(uid=<email-input-uid>, value="${F5XC_SSO_USER}")
click(uid=<next-button-uid>)
wait_for(text=["Enter password", "Password"], timeout=10000)
```

### Step 6: Password entry (Full MFA)

```
take_snapshot()
fill(uid=<password-input-uid>, value="${F5XC_CONSOLE_PASSWORD}")
click(uid=<signin-button-uid>)
wait_for(text=["Approve sign-in request", "Verify your identity",
               "Stay signed in", "web/workspaces"], timeout=10000)
```

### Step 7: DUO MFA and "Stay signed in"

**If DUO push screen is shown** ("Approve sign-in request" or
"Verify your identity"):

Report to the operator:

> **MFA push sent. Please approve the DUO notification on your
> phone.**

Then wait for the user to approve:

```
wait_for(text=["Stay signed in", "web/workspaces"], timeout=60000)
```

If timeout expires, report: "MFA approval timed out. Please
try again."

**If "Stay signed in?" is shown:**

```
take_snapshot()
click(uid=<yes-button-uid>)   # Click "Yes" to persist session
wait_for(text=["web/workspaces"], timeout=15000)
```

### Step 8: Verification

```
take_snapshot()
```

Confirm success:

- URL contains `/web/workspaces` or a known console path
- Snapshot contains workspace/tenant elements (navigation
  sidebar, user avatar, workspace name)

Report structured result:

```
## Console Authentication: SUCCESS
- Tenant: ${F5XC_API_URL}
- User: ${F5XC_SSO_USER}
- Mode: Cached Session | Full MFA
- Workspace: <detected workspace name>
```

## Error Handling

| Failure | Detection | Action |
| --------- | ----------- | -------- |
| VPN not connected | Navigation timeout or DNS error | Report: "Cannot reach tenant. Check VPN." |
| Invalid credentials | "Wrong password" or "Account doesn't exist" | Report error. Do NOT retry (lockout risk). |
| MFA timeout | `wait_for` exceeds 60s on DUO screen | Report: "MFA approval timed out." |
| Account locked | "Your account has been locked" | Report immediately. Do NOT retry. |
| Conditional Access | "You cannot access this" | Report: "Conditional Access policy blocked." |
| Session expired | Redirect to `/web/login` mid-workflow | Re-invoke this auth flow automatically. |
| Azure AD outage | "Something went wrong" page | Report: "Azure AD service error." |

## Security

- Credentials are read from environment variables only
- Passwords are never logged, stored in files, or echoed
- The `F5XC_CONSOLE_PASSWORD` value is passed directly to the
  browser `fill` tool and is not retained after use
- Session cookies are managed by the browser, not the plugin
