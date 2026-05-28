---
name: ux-rfc
description: Write a UX design RFC for sharing with designers via Notion. No code, no file paths — flows, states, and component descriptions only. MANDATORY TRIGGERS: /ux-rfc <name>, "write a UX RFC", "design doc for designers", "Notion design doc". DO NOT trigger for: technical RFCs (use /rfc), internal engineer docs, post-implementation reviews.
user_invocable: true
---

# ux-rfc (`/ux-rfc`)

Produces a designer-friendly UX RFC suitable for Notion. Contains user flows, screen states, and component behaviour — no code, no file paths.

## Required inputs

- Feature name or ticket ID
- Optional: existing JTBD file or user research notes

## Steps

### 1. Gather context (parallel)

- Read `jtbd/<feature-name>.md` — extract user jobs and success criteria
- Read `rfcs/<feature-name>.md` if it exists — extract any UX decisions already made
- Fetch the issue from your issue tracker if a ticket ID is given

### 2. Write the UX RFC

Structure the document for a designer audience:

```markdown
# UX RFC: [Feature Name]

**Date:** YYYY-MM-DD
**Author:**
**Ticket:** [TICKET-ID]
**Status:** Draft

---

## What we're building and why

[2–3 sentences. No jargon. What the user can do that they can't do today.]

---

## Users and jobs

| User type | Job to be done |
|-----------|---------------|
| [persona] | When …, I want …, so I can … |

---

## User flows

### Flow A: [Primary flow name]

1. User arrives at [entry point]
2. User sees [state description]
3. User does [action]
4. System shows [response]
5. User reaches [success state]

### Flow B: [Edge case / error flow]

1. …

---

## Screen states

### State 1: [Name] — [when this state appears]

- **Header:** [text]
- **Body:** [description of content]
- **Primary action:** [label and what it does]
- **Secondary action / empty state / error state:** [description]

### State 2: …

---

## Open design questions

- [ ] [Question for designer]
- [ ] [Question for designer]

---

## Out of scope

- [What we are not designing in this iteration]
```

### 3. Post to Notion (if requested)

If the user asks to post to Notion:
- Check if a Notion page for this feature already exists — **fetch first, verify it was created by you before modifying**
- If no page exists, create a new private page under the agreed design docs section
- Paste the UX RFC content

### 4. Offer next steps

> "UX RFC written. Share with designer for feedback. When implementation is ready, run `/ux-review <feature-name>` to check it against this doc."

## Handling edge cases

- **No JTBD file**: extract jobs from ticket description or ask one clarifying question before proceeding
- **Notion page exists but not created by you**: do not modify — tell the user and offer to create a new page instead
- **Feature is very visual**: suggest adding wireframe embeds or Figma links as placeholders in the Notion page

## Quality check

- [ ] No code snippets, file paths, or technical identifiers in the document
- [ ] Every screen state is named and has a trigger condition
- [ ] Open design questions are concrete, not vague ("what colour?" not "what should it look like?")
- [ ] Document reads cleanly to someone who doesn't know the codebase
