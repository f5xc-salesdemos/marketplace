# Azure SSO Flow — MCP Tool Sequences

Detailed Chrome DevTools MCP tool call sequences for each
authentication path. The `console-auth` skill references
this file for implementation details.

## Path A: Cached Session (Single-Click)

Pre-condition: User previously authenticated and selected
"Remember this device" / "Stay signed in: Yes".

```
1. navigate_page(url="${F5XC_API_URL}/web/login")
2. wait_for(text=["Sign in with Azure"])
3. take_snapshot()
4. click(uid=<azure-sso-button>)          # Text: "Sign in with Azure"
5. wait_for(text=["web/workspaces"], timeout=15000)
6. take_snapshot()                          # Verify console loaded
```

Expected duration: 3–8 seconds.

## Path B: Account Picker → Cached Session

Pre-condition: Multiple Azure AD accounts cached in browser.

```
1. navigate_page(url="${F5XC_API_URL}/web/login")
2. wait_for(text=["Sign in with Azure"])
3. take_snapshot()
4. click(uid=<azure-sso-button>)
5. wait_for(text=["Pick an account"], timeout=15000)
6. take_snapshot()
7. click(uid=<account-matching-F5XC_SSO_USER>)  # Match by email text
8. wait_for(text=["web/workspaces", "Enter password"], timeout=15000)
   # If workspaces → done. If password → continue to Path C Step 6.
9. take_snapshot()
```

## Path C: Full MFA Flow

Pre-condition: No cached session or session expired.

```
 1. navigate_page(url="${F5XC_API_URL}/web/login")
 2. wait_for(text=["Sign in with Azure"])
 3. take_snapshot()
 4. click(uid=<azure-sso-button>)

 # Username screen
 5. wait_for(text=["Sign in", "Enter your email"], timeout=15000)
 6. take_snapshot()
 7. fill(uid=<email-input>, value="${F5XC_SSO_USER}")
 8. click(uid=<next-button>)                # Text: "Next"

 # Password screen
 9. wait_for(text=["Enter password"], timeout=10000)
10. take_snapshot()
11. fill(uid=<password-input>, value="${F5XC_CONSOLE_PASSWORD}")
12. click(uid=<signin-button>)              # Text: "Sign in"

 # DUO MFA
13. wait_for(text=["Approve sign-in request", "Verify your identity",
                   "Stay signed in"], timeout=10000)
14. take_snapshot()
    # If DUO screen: notify operator, then wait
15. wait_for(text=["Stay signed in", "web/workspaces"], timeout=60000)

 # Stay signed in
16. take_snapshot()
17. click(uid=<yes-button>)                 # Text: "Yes"
18. wait_for(text=["web/workspaces"], timeout=15000)
19. take_snapshot()                          # Verify console loaded
```

Expected duration: 15–90 seconds (depends on MFA approval speed).

## URL Patterns for Detection

| URL Pattern | Meaning |
| ------------- | --------- |
| `*/web/login*` | F5 XC login page |
| `*/web/workspaces*` | Console loaded (authenticated) |
| `login.microsoftonline.com*` | Azure AD login screens |
| `*duosecurity.com*` | DUO MFA challenge |
| `*/kmsi*` | "Keep me signed in" prompt |

## Element Identification Strategy

Always use `take_snapshot()` before interacting. Find
elements using this priority:

1. **Text content** — most reliable for auth flows since
   Azure AD pages use standard button text ("Next",
   "Sign in", "Yes", "No")
2. **Input type** — `type="email"` for username,
   `type="password"` for password
3. **aria-label** — backup for buttons without visible text
4. **Placeholder text** — "Email, phone, or Skype" for
   the username field on Azure AD
