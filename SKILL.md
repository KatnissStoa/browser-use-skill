---
name: browser-use-autopilot
description: "Browser automation orchestrator that executes multi-step web tasks on behalf of the user via browser-use (LLM + Playwright). Handles login with credential persistence, task planning, step execution with snapshot-based state classification, user confirmation checkpoints, and interrupt routing. Use for: automating social media posting, form filling, subscription cancellation, cross-platform web workflows, scheduled browser tasks."
---

# Browser Use Autopilot

Orchestrate multi-step browser automation tasks via conversation. The user never sees the browser — all interaction happens through text (with optional screenshots when ambiguity arises). Built on browser-use (LLM + Playwright).

## Architecture Overview

```
User request (text)
  ↓
[Task Planner] → intent parse + step chain
  ↓
[Task Queue] → persist to JSON
  ↓
[Scheduler] → immediate or cron
  ↓
[Step 0: Auth Check]
  → logged in → continue
  → not logged in + has credential → auto-login (silent)
  → not logged in + reusable credential → prompt reuse
  → not logged in + no credential → ask user → login → offer to save
  ↓
[Auth Profile] ← encrypted persistent store (user-authorized)
  ↓
[Step Executor] → act → snapshot → classify
  ↓
[State Classifier] → 5 states (A/B/C/D/success)
  ↓
[Interrupt Router] → dialog with user → resume
```

## Trigger Conditions

Activate when:
- User asks to perform an action on a website (post, fill, cancel, subscribe, scrape, navigate)
- User asks to schedule a recurring browser task
- User asks to manage saved login credentials
- User provides a URL and an action to perform
- User references a platform by name with an operational intent (e.g. "post on Twitter", "cancel my Netflix")

## Non-Trigger Conditions

Do NOT activate when:
- User asks a general question about a website (use general knowledge)
- User wants to browse or read content without taking action
- User asks about browser-use setup or installation (provide docs instead)
- Request involves illegal activity or ToS-violating automation (mass spam, credential stuffing)

## Confirmation Gates

Always confirm with the user before:
- **Publishing** any content (social media posts, comments, replies) — show draft text first
- **Submitting** any form (applications, payments, registrations) — section-level + final checkpoint
- **Cancelling** any subscription or account — confirm entry point + final action
- **Deleting** any data (posts, accounts, files)
- **Making payments** or financial transactions

## Stop Conditions

Stop immediately and escalate when:
- Browser-use cannot reach the target URL after 2 retries
- Login fails with saved credentials AND user does not provide new ones
- Page structure deviates from expectation with no recoverable path (Path D)
- User says "stop", "cancel", "abort"
- CAPTCHA/slider that cannot be automated — inform user

## Risk Levels

| Level | Definition | Action |
| --- | --- | --- |
| Low | Read-only navigation, data gathering | Proceed silently |
| Medium | Form filling, content drafting | Checkpoint before submit |
| High | Publishing, payments, account changes | Full confirmation with preview |
| Critical | Account deletion, irreversible financial action | Double confirmation, require explicit "yes" |

## Workflow

### Step 0: Auth Check

See [references/auth-profile.md](references/auth-profile.md) for full credential storage schema.

```
Navigate to target platform → snapshot → detect login state

IF logged_in:
  proceed to Step 1

IF not_logged_in:
  1. Check Auth Profile for this platform's credentials
     → found → auto-login silently
       → success → continue
       → fail → tell user "password may have changed", ask for new credentials
  2. Check Auth Profile for same-email credentials on other platforms
     → found → prompt:
       "Need to log in to {platform}. You used {email} on {other_platform}.
        Use the same credentials?
        ① Yes  ② Use a different account"
     → user confirms → attempt login
     → user declines → ask for new credentials
  3. No credentials at all → ask user for email + password
  4. Execute login via browser-use
     → 2FA → "Platform sent a code to your phone/email. Tell me the code."
     → Slider/CAPTCHA → "Can't automate this. ① Handle manually ② Retry later ③ Abort"
     → Email confirm → "Check your inbox, click confirm, tell me when done"
  5. Login success →
     "Logged in. Save credentials for next time?
      ① Save  ② Don't save"
     → ① encrypt + store to Auth Profile
     → ② discard after session
```

Auto-login with saved credentials is **silent** — no notification (user already authorized).
Credential failure triggers **proactive notice** + re-acquisition.

### Step 1: Task Planning

1. Parse user intent → identify platform + action type
2. Check Playbook registry (see [references/playbooks.md](references/playbooks.md))
   - Playbook found → use predefined step chain
   - No playbook → LLM generates steps with tighter retry/deviation thresholds
3. Persist step chain to Task Queue as JSON

### Step 2: Step Execution Loop

For each step in the chain:

1. **Act** — browser-use performs action (click, type, navigate, select)
2. **Snapshot** — capture page state (accessibility tree + optional screenshot)
3. **Classify** — State Classifier assigns one of 5 states

### Step 3: State Classification → Routing

See [references/state-classification.md](references/state-classification.md) for details.

```
Path A — User intervention required:
  - 2FA / verification code
  - Email confirmation link
  - Slider / CAPTCHA
  - Permission denied / account restricted
  - Session expired mid-task
  → Interrupt Router: describe + options + wait

Path B — Data missing:
  - Required field has no value
  - Format mismatch / character limit
  → Interrupt Router: describe need + options (provide / skip / abort)

Path C — Recoverable error:
  - Slow load / element not visible
  - Transient network error
  → Retry up to 2×, re-snapshot each time
  → After 2 failures → escalate to Path A

Path D — Page inconsistency:
  - URL / structure deviates from expectation
  - Elements missing or relocated
  → Zero retries
  → Describe deviation + possible causes
  → Attach screenshot if text is ambiguous
  → User decides: retry with guidance / abort

Success:
  → Log result, proceed to next step
```

### Step 4: Interrupt Router

1. Compose message describing situation
2. Decide screenshot:
   - **Text only** (default): unambiguous in words
   - **Text + screenshot**: multiple similar elements, visual layout matters, or Path D
3. Present numbered options (① ② ③)
4. Wait for user reply
5. Map reply → action → resume

### Step 5: Checkpoints

See [references/checkpoints.md](references/checkpoints.md) for per-scenario rules.

- **Social media**: draft preview → publish confirmation
- **Form filling**: section summary → final summary before submit
- **Subscription cancel**: entry-point confirmation → final confirmation

### Step 6: Completion

1. Report result via text
2. Log operation (see [references/operation-log.md](references/operation-log.md))
3. Update Task Queue status

## Credential Management

Users manage saved credentials via conversation:
- "Delete my saved login for Twitter" → remove specific platform
- "Delete all my saved logins" → clear entire Auth Profile
- "Show saved logins" → list platforms only (never show passwords)

## References

- **Auth Profile**: See [references/auth-profile.md](references/auth-profile.md)
- **State Classification**: See [references/state-classification.md](references/state-classification.md)
- **Checkpoints**: See [references/checkpoints.md](references/checkpoints.md)
- **Playbooks**: See [references/playbooks.md](references/playbooks.md)
- **Operation Log**: See [references/operation-log.md](references/operation-log.md)
