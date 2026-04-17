# Operation Log

Every task execution produces a structured log for debugging and audit.

## Log Schema

```json
{
  "task_id": "uuid",
  "user_id": "uuid",
  "created_at": "ISO-8601",
  "completed_at": "ISO-8601",
  "status": "completed | failed | aborted",
  "platform": "twitter",
  "action": "post",
  "steps": [
    {
      "step_number": 1,
      "action": "navigate_to_compose",
      "screenshot_before": "path/to/before.png",
      "screenshot_after": "path/to/after.png",
      "state_classification": "success",
      "retry_count": 0,
      "duration_ms": 1200,
      "notes": null
    },
    {
      "step_number": 2,
      "action": "type_post_content",
      "screenshot_before": "path/to/before.png",
      "screenshot_after": "path/to/after.png",
      "state_classification": "success",
      "retry_count": 0,
      "duration_ms": 800,
      "notes": null
    },
    {
      "step_number": 3,
      "action": "click_post_button",
      "screenshot_before": "path/to/before.png",
      "screenshot_after": "path/to/after.png",
      "state_classification": "path_c",
      "retry_count": 1,
      "duration_ms": 4500,
      "notes": "Button not visible on first attempt, retried after 2s wait"
    }
  ],
  "auth": {
    "was_logged_in": false,
    "login_method": "saved_credential",
    "credential_reused_from": null,
    "login_success": true
  },
  "checkpoints": [
    {
      "type": "CP-draft",
      "presented_at": "ISO-8601",
      "user_response": "publish",
      "response_time_ms": 15000
    }
  ],
  "interrupts": [
    {
      "path": "B",
      "reason": "GPA field missing",
      "presented_at": "ISO-8601",
      "user_response": "provided value: 3.8",
      "response_time_ms": 30000
    }
  ],
  "error": null
}
```

## Log Retention

- Logs persist locally alongside the Task Queue
- Screenshots stored in a `logs/screenshots/` directory, referenced by path
- No automatic cleanup — user can request "clear old logs"

## What Gets Logged

| Event | Logged? |
| --- | --- |
| Each step's before/after screenshots | Yes |
| State classification per step | Yes |
| Retry count per step | Yes |
| Auth check result | Yes |
| Checkpoint presentations + user responses | Yes |
| Interrupt presentations + user responses | Yes |
| User-provided credentials | **No** (never log passwords) |
| Verification codes | **No** |
| Full page DOM | No (too large; use screenshots) |
