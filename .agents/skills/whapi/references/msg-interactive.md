---
title: Interactive Messages — Buttons, Lists, and Polls (Stability Warning)
impact: MEDIUM
impactDescription: Interactive buttons may fail silently depending on the recipient's WhatsApp version; polls are the stable alternative
tags: sendMessageInteractive, sendMessagePoll, buttons, list, interactive, action, header, footer, poll, stability
---

## Interactive Messages — Buttons, Lists, and Polls

WHAPI supports two approaches for interactive user input:
1. **Polls** (`sendMessagePoll`) — stable, recommended
2. **Interactive messages** (`sendMessageInteractive`) — unstable, depends on WhatsApp version

---

### Stability Warning for Interactive Messages

> **Warning:** The functionality of interactive messages (buttons, lists, carousels)
> is **not guaranteed to be stable**. WhatsApp controls whether buttons are rendered
> and may change behavior without notice. A message sent with `sendMessageInteractive`
> may be delivered as plain text on some versions, or buttons may not be tappable.

Use `sendMessagePoll` as the reliable alternative for collecting user choices.

---

### Polls (Stable — Use This First)

Polls are rendered natively in WhatsApp and work reliably across all versions.

**Correct — single-choice poll:**
```json
// Tool: sendMessagePoll
{
  "to": "14155552671@s.whatsapp.net",
  "title": "Which delivery time works for you?",
  "options": ["9:00 AM - 12:00 PM", "12:00 PM - 3:00 PM", "3:00 PM - 6:00 PM"],
  "count": 1
}
// count: 1 = single choice, count: 0 = multiple choices allowed
```

**Correct — multi-select poll:**
```json
// Tool: sendMessagePoll
{
  "to": "120363194050948049@g.us",
  "title": "Which features should we add next?",
  "options": ["Dark mode", "Offline mode", "Export to PDF", "API access"],
  "count": 0
}
```

Poll limitations:
- Maximum 12 options
- Title is required; options array is required (minimum 2 items)
- Poll responses are received via webhooks as regular message events

---

### Tracking Poll Responses

When a user votes, WHAPI sends a webhook event with `type: "poll_update"`.
The payload contains the updated poll with vote counts and voter Chat IDs.

**Webhook payload on vote:**
```json
{
  "messages": [
    {
      "id": "p.w30M7fgwWD4XwHu.g4CA-gBgTwl0rVw",
      "from_me": true,
      "type": "poll_update",
      "chat_id": "14155552671@s.whatsapp.net",
      "poll": {
        "title": "Which delivery time works for you?",
        "results": [
          {
            "name": "9:00 AM - 12:00 PM",
            "count": 1,
            "voters": ["14155552671@s.whatsapp.net"],
            "id": "YCUAzJ4pYtRgSP+k/udEwRdvHxxuRQmNCrdTKz/K2TU="
          },
          {
            "name": "12:00 PM - 3:00 PM",
            "count": 0,
            "voters": [],
            "id": "f61LjJqD6KEefNsuDQrvFXAB1iufPN3U1ykIOxD8DRc="
          }
        ]
      }
    }
  ]
}
```

Key fields:
- `messages[n].type` — will be `"poll_update"` for vote events
- `messages[n].poll.results[n].voters` — Chat IDs of everyone who picked this option
- `messages[n].poll.results[n].count` — current vote count for this option
- `messages[n].id` — the poll message ID (use this to retrieve current state via GET)

**Retrieve current vote counts via GET:**
```json
// Tool: getMessage
{ "MessageID": "p.w30M7fgwWD4XwHu.g4CA-gBgTwl0rVw" }
// Returns the full poll message with latest results[].count and results[].voters
// Use the ID of the original poll message (from the sendMessagePoll response or webhook)
```

> **Note:** `GET /messages/{MessageID}` works for regular chat polls.
> For Channel (newsletter) polls, a separate tracking subscription is required —
> see `channels-management.md`.

---

### Interactive Messages (Use With Caution)

If you need buttons or lists and accept the instability risk, use `sendMessageInteractive`.

**Required fields:** `to`, `action`

**Incorrect (missing action or wrong structure):**
```json
// Tool: sendMessageInteractive
{
  "to": "14155552671@s.whatsapp.net",
  "buttons": ["Yes", "No"]
}
// "buttons" is not a top-level field — structure must follow WhatsApp spec
```

**Correct — reply buttons:**
```json
// Tool: sendMessageInteractive
{
  "to": "14155552671@s.whatsapp.net",
  "type": "button",
  "body": { "text": "Confirm your appointment?" },
  "action": {
    "buttons": [
      { "type": "reply", "reply": { "id": "yes", "title": "Confirm" } },
      { "type": "reply", "reply": { "id": "no", "title": "Cancel" } }
    ]
  }
}
```

**Correct — list message:**
```json
// Tool: sendMessageInteractive
{
  "to": "14155552671@s.whatsapp.net",
  "type": "list",
  "body": { "text": "Choose a support category:" },
  "action": {
    "button": "Open menu",
    "sections": [
      {
        "title": "Support",
        "rows": [
          { "id": "billing", "title": "Billing", "description": "Payment issues" },
          { "id": "technical", "title": "Technical", "description": "App problems" }
        ]
      }
    ]
  }
}
```

---

### Decision Guide

```
Need user to pick one of N options?
│
├── Options are simple text choices → sendMessagePoll (reliable)
│
└── Need branded/styled buttons with custom IDs → sendMessageInteractive
    └── Accept that it may not render on all devices
```

---

**Anti-hallucination — parameters that do NOT exist:**
- `buttons` as a top-level parameter — buttons go inside `action.buttons`
- `text` as a top-level parameter — body text goes in `body.text`
- `options` in `sendMessageInteractive` — use `action.buttons` or `action.sections`
- `choices` — does not exist in either tool
- `quick_replies` — use `action.buttons` with type `reply`

Reference: [Button Status](https://support.whapi.cloud/help-desk/faq/current-status-of-buttons-on-whatsapp) | [Polls as Buttons](https://support.whapi.cloud/help-desk/hints/how-to-use-polls-as-buttons)
