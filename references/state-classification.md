# State Classification

After each step execution: act → snapshot → classify. The State Classifier maps the current page state to one of 5 outcomes.

## Classification Inputs

- Accessibility tree / DOM snapshot
- Current URL vs expected URL
- Visible text content
- Element presence/absence checks
- Optional: screenshot for visual analysis

## Five States

### Path A — User Intervention Required

**Trigger conditions:**
- Login/auth prompt detected (2FA, verification code input field)
- Email confirmation page ("check your email")
- CAPTCHA / slider / "prove you're human" element
- "Permission denied" / "account restricted" / "upgrade required" message
- Session expired indicator (redirect to login page mid-task)

**Handler behavior:**
- Describe the situation in plain text
- Present numbered options (① ② ③)
- If situation is visually complex (multiple similar buttons, unclear layout) → attach screenshot
- Wait for user text reply
- Map reply to action → resume

**Screenshot decision rule:**
- Text sufficient: "Platform is asking for a 6-digit verification code sent to your phone."
- Screenshot needed: "There are two 'Cancel' buttons on this page — one downgrades to free, the other closes the account entirely." + [screenshot]

### Path B — Data Missing

**Trigger conditions:**
- Required form field has no available value (not in user profile, not inferable)
- Value format doesn't match field requirement (e.g. date format, phone format)
- Text exceeds character limit
- Dropdown option not matching any expected value

**Handler behavior:**
- Describe what's needed:
  "Filling the Stanford application — 'Academic History' section needs your GPA. Not in your profile.
   ① Tell me your GPA
   ② Skip this field
   ③ Abort"
- Never guess or fabricate data
- If multiple fields missing in same section → batch them in one message

### Path C — Recoverable Error

**Trigger conditions:**
- Page still loading (spinner visible, elements not yet rendered)
- Element exists in DOM but not visible/clickable (overlay, animation)
- Transient network error (timeout, 5xx)
- Stale element reference (DOM changed between snapshot and action)

**Handler behavior:**
- Wait 2-3 seconds
- Re-snapshot
- Retry the same action
- Maximum 2 retries per step
- After 2 failures → escalate to Path A: "Having trouble with this step. ① Retry ② Skip ③ Abort"

**No user notification** during retries — only escalate if retries exhausted.

### Path D — Page Inconsistency

**Trigger conditions:**
- Current URL doesn't match expected URL (unexpected redirect)
- Expected page structure/elements not found (site redesign, A/B test variant)
- Page content contradicts expectations (e.g. settings page shows different options than playbook)

**Handler behavior:**
- **Zero retries** — do not attempt the action on an unexpected page
- Describe the deviation:
  "Expected to find the subscription settings page, but landed on a different page.
   Possible reasons: site redesigned, region-specific variant, or wrong navigation path."
- Attach screenshot if text description is ambiguous
- List possible causes
- User decides: "① Try a different path ② Describe what you see so I can guide you ③ Abort"

### Success

**Trigger conditions:**
- Action completed and page state matches expected post-action state
- URL transitioned as expected
- Confirmation element appeared (success toast, redirect to expected page)

**Handler behavior:**
- Log: screenshot_before, screenshot_after, action_taken, state_classification
- Proceed to next step in chain
- If this was the last step → move to Checkpoint (if applicable) or Completion
