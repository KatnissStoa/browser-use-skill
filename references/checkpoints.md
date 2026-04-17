# Checkpoints

Confirmation gates at critical moments during task execution. Checkpoints pause execution and require explicit user approval before proceeding.

## Checkpoint Types

### CP-draft — Content Preview

**When:** Before publishing any user-visible content (posts, comments, replies, messages).

**Format:**
```
Here's the draft for your {platform} post:

---
{content text}
{media attachments if any}
---

① Publish
② Edit (tell me what to change)
③ Cancel
```

### CP-section — Section Summary

**When:** After completing a major section of a multi-field form.

**Format:**
```
Completed the "{section_name}" section:
- Field 1: {value}
- Field 2: {value}
- Field 3: {value}

① Looks good, continue
② Fix something (tell me which field)
③ Abort
```

### CP-final — Pre-Submit Confirmation

**When:** Before any irreversible submit/publish/cancel action.

**Format:**
```
Ready to {action}. Full summary:

{complete summary of all filled fields / post content / cancellation details}

① Confirm
② Go back and edit
③ Cancel entirely
```

## Scenario-Specific Rules

### Social Media Posting

1. **CP-draft**: Show full post text + attached media description
2. **CP-final**: "About to publish on {platform}. Confirm?"
- If post includes mentions/tags → explicitly list them
- If post includes links → show full URL
- If scheduling → show scheduled time

### Form Filling (Applications, Registrations)

1. **CP-section**: After each major section (personal info, academic history, work experience, etc.)
   - Only if section has 3+ fields; skip for trivial sections (e.g. just name)
2. **CP-final**: Before submit, show all sections summarized
   - Highlight any skipped/empty optional fields
   - Flag any fields where value was inferred (not directly provided by user)

### Subscription Cancellation

1. **CP-entry**: Confirm which cancellation path:
   "Found two options:
    ① Downgrade to free plan (keep account)
    ② Close account entirely
    Which one?"
   - If only one path exists, still confirm: "This will cancel your {plan}. Proceed?"
2. **CP-final**: Before clicking the final cancel/confirm button
   - State exactly what will happen: "Your {plan} will end on {date}. You'll lose access to {features}."
   - Require explicit ① Confirm

### Account Settings Changes

1. **CP-final only**: Show what will change:
   "Changing email from {old} to {new}. Platform may send a verification email. Proceed?"

### Data Deletion

1. **CP-final**: Double confirmation:
   "This will permanently delete {what}. This cannot be undone.
    Type 'yes' to confirm, or ② Cancel."
   - Require the text "yes", not just option ①

## Screenshot Usage in Checkpoints

- CP-draft: text only (content is already text)
- CP-section: text only (field/value pairs)
- CP-final for cancellation: attach screenshot if the page shows additional warnings or fine print
- CP-entry when multiple paths: attach screenshot if buttons/links are visually confusing
