# Intent Classification Logic

## Overview

Every LinkedIn comment processed by the system passes through an AI classification
layer before any action is taken. The classifier determines which of three routing
paths the comment follows — freebie delivery, hot lead alert, or standard reply.

---

## Classification Categories

### Category A — Freebie Request
**Definition:** The commenter is explicitly or implicitly requesting the resource
offered in the post.

**Signal phrases (examples):**
- "Can you send me this?"
- "Drop the link please"
- "I want this / I'd love this"
- "How do I get this?"
- "Me please" / "Send it over"
- "Interested!" (when post offers a resource)
- "Count me in"

**Action triggered:**
1. Cross-reference Airtable resource library
2. Retrieve correct resource link for this post
3. Send LinkedIn DM via HeyReach with resource
4. Log delivery in Airtable lead tracker

---

### Category B — Hot Lead Signal
**Definition:** The commenter expresses a pain point, problem, or explicit interest
in consultation — indicating high purchase intent.

**Signal phrases (examples):**
- "I'm struggling with this exact issue"
- "Need a consultation on this"
- "This is my situation right now"
- "How do I fix this for my business?"
- "I've been dealing with this for months"
- "Can we talk about this?"
- "I need help with [specific problem]"

**Action triggered:**
1. Immediate Slack alert fired to client channel
2. Alert includes: commenter name, LinkedIn profile, full comment text, post URL
3. Team follows up manually with full context
4. If booking link is sent → Calendly monitoring activated

---

### Category C — Standard Engagement
**Definition:** General positive engagement, reactions, or comments that don't
indicate resource requests or consultation intent.

**Signal phrases (examples):**
- "Great post"
- "Thanks for sharing"
- "So true"
- "Valuable content"
- "Love this"
- Emoji-only responses
- Tag mentions of other users

**Action triggered:**
1. LeadShark automated reply sent
2. No further follow-up

---

## Classification Prompt Structure

```
You are an expert sales intent classifier for LinkedIn comments.

Analyze the following comment and classify it as one of:
- FREEBIE_REQUEST: commenter wants the resource offered in the post
- HOT_LEAD: commenter expresses a pain point or consultation interest
- STANDARD: general engagement with no resource request or buying signal

Post context: {post_content}
Comment: {comment_text}

Return JSON only:
{
  "classification": "FREEBIE_REQUEST" | "HOT_LEAD" | "STANDARD",
  "confidence": 0.0-1.0,
  "reasoning": "one sentence explanation"
}
```

---

## Booking Follow-up Logic

```
Trigger:  Booking link sent to a Hot Lead
Window:   1–2 days after link delivery
Check:    Query Calendly API for booking from prospect email

Day 1 check (24 hours after link sent):
  → Booking found: mark lead as converted, stop monitoring
  → No booking: continue monitoring

Day 2 check (48 hours after link sent):
  → Booking found: mark lead as converted, stop monitoring
  → No booking: fire Slack alert
    Alert includes:
      - Prospect name and LinkedIn profile
      - Original comment and date
      - Date booking link was sent
      - "No booking detected — follow up recommended"
  → Stop monitoring regardless of outcome
```

---

## Airtable Resource Library Schema

| Field | Type | Purpose |
|---|---|---|
| `resource_name` | Text | Human-readable name |
| `post_id` | Text | LinkedIn post this resource maps to |
| `resource_url` | URL | The actual freebie link |
| `resource_type` | Select | PDF, Template, Video, Course, Guide |
| `active` | Checkbox | Whether resource is currently offered |
| `delivery_count` | Number | Times this resource has been sent |
| `last_sent` | Date | Most recent delivery timestamp |

---

## Airtable Lead Tracker Schema

| Field | Type | Purpose |
|---|---|---|
| `commenter_name` | Text | LinkedIn display name |
| `linkedin_url` | URL | Profile link |
| `comment_text` | Text | Original comment |
| `classification` | Select | FREEBIE / HOT_LEAD / STANDARD |
| `resource_sent` | Checkbox | Whether freebie was delivered |
| `booking_link_sent` | Checkbox | Whether Calendly link was shared |
| `booked` | Checkbox | Whether call was scheduled |
| `processed_at` | Datetime | When system handled this comment |
