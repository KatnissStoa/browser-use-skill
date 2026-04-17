# Playbooks

Predefined step chains for high-frequency scenarios. When a matching playbook exists, use it instead of LLM-generated steps for higher reliability.

## Playbook Structure

Each playbook defines:
- **Platform**: target website
- **Action**: what the user wants to do
- **Steps**: ordered list of browser-use actions
- **Expected states**: what each step's success looks like
- **Known variants**: platform-specific quirks (A/B tests, regional differences)
- **Checkpoints**: which CPs apply and when

## Playbook Registry

### Social Media — Post

| Platform | Steps |
| --- | --- |
| Twitter/X | Navigate home → click compose → type text → attach media (if any) → CP-draft → click post → verify posted |
| LinkedIn | Navigate home → click "Start a post" → type text → attach media (if any) → CP-draft → click Post → verify posted |
| Instagram | Navigate to create → select media → add caption → CP-draft → click Share → verify posted |

### Social Media — Delete Post

| Platform | Steps |
| --- | --- |
| Twitter/X | Navigate to post URL → click ··· menu → click Delete → CP-final → confirm delete → verify deleted |
| LinkedIn | Navigate to post URL → click ··· menu → click Delete → CP-final → confirm → verify deleted |

### Subscription — Cancel

General flow (adapt per platform):
1. Navigate to account/settings
2. Find subscription/billing section
3. Locate cancel/downgrade option
4. CP-entry (confirm which path)
5. Follow cancellation wizard (may have multiple "are you sure" pages)
6. CP-final before last confirm
7. Verify cancellation confirmed

### Form — Fill and Submit

General flow:
1. Navigate to form URL
2. For each section:
   a. Identify fields
   b. Fill from user profile / provided data
   c. Flag missing required fields (→ Path B)
   d. CP-section after major sections
3. CP-final with full summary
4. Submit
5. Verify submission confirmation

## Long-tail Scenarios (No Playbook)

When no playbook matches:
1. LLM generates step chain based on the user's request
2. Apply tighter thresholds:
   - Path C retries: max 1 (not 2)
   - Path D tolerance: zero (any unexpected page → immediately ask user)
3. Log the generated step chain for potential future playbook creation
4. After successful completion, consider adding to playbook registry

## Playbook Maintenance

- Playbooks may break when platforms update their UI
- When a playbook step hits Path D (page inconsistency):
  1. Flag the playbook as potentially stale
  2. Fall back to LLM-guided navigation for remaining steps
  3. Log the deviation for playbook update
