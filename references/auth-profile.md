# Auth Profile

Persistent encrypted credential store. Serves as the backend for Step 0 Auth Check.

## Storage Schema

```json
{
  "user_id": "uuid",
  "credentials": {
    "<platform>": {
      "email": "user@example.com",
      "password": "encrypted:<base64>",
      "saved_at": "ISO-8601",
      "last_used": "ISO-8601",
      "last_success": true
    }
  }
}
```

- `platform`: lowercase identifier (e.g. `twitter`, `linkedin`, `netflix`)
- `password`: AES-256-GCM encrypted, key derived from user-specific secret
- `last_success`: updated after every login attempt

## Auth Check Flow

### Priority order (top to bottom):

1. **Browser already logged in** → proceed (no Auth Profile interaction)
2. **Auth Profile has platform credential** → auto-login silently
3. **Auth Profile has same-email credential for another platform** → prompt cross-platform reuse
4. **No credential** → ask user for email + password

### Auto-login (saved credential exists)

```
Read credential from Auth Profile
  → Fill email + password via browser-use
  → Submit login form
  → Snapshot → check login success
  → Success:
      update last_used + last_success=true
      proceed silently (no user notification)
  → Failure:
      set last_success=false
      tell user: "Saved password for {platform} didn't work (may have changed).
                  Please provide your current password."
      → user provides → retry login
      → success → update credential in Auth Profile
      → failure again → "Still can't log in. ① Try again ② Abort"
```

### Cross-platform reuse (same email, different platform)

```
Scan Auth Profile for any credential with matching email
  → Found (e.g. email used on Twitter, now need LinkedIn):
    "Need to log in to LinkedIn. You used {email} on Twitter before.
     Use the same credentials?
     ① Yes, use this
     ② Use a different account"
  → User picks ①:
    Attempt login with the reused password
    → Success → offer to save as new platform entry
    → Failure → "Password for LinkedIn differs from Twitter.
                 Please provide LinkedIn's password."
  → User picks ②:
    Ask for new email + password
```

### First-time login (no credential)

```
"Need to log in to {platform}. Please provide your email and password."
  → User provides credentials
  → browser-use executes login
  → Handle interrupts:
    - 2FA: "Platform sent a 6-digit code to your phone/email. Tell me the code."
    - Slider/CAPTCHA: "Hit a CAPTCHA I can't automate.
      ① Handle it manually and tell me when done
      ② Retry later
      ③ Abort"
    - Email confirm: "Confirmation email sent. Click the link, then tell me 'done'."
  → Login success:
    "Logged in successfully. Save your login info for future use?
     ① Save (auto-login next time)
     ② Don't save (this session only)"
    → ① encrypt password → write to Auth Profile
    → ② hold in memory for session, discard on exit
```

## Security Rules

- Passwords encrypted at rest (AES-256-GCM)
- Encryption key derived per-user, never stored in plaintext
- Auto-login with saved credentials: **silent execution, no user notification** (already authorized)
- Credential failure: **proactive notification** + request new credentials
- User can delete at any time:
  - "Delete my saved login for {platform}" → remove single entry
  - "Delete all saved logins" → wipe entire credentials object
  - "Show my saved logins" → list platform names + emails only, never passwords
- No OAuth button paths — always use email + password directly
- Credentials never sent to any external service; used only in browser-use login execution
