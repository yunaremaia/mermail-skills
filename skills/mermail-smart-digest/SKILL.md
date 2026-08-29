---
name: mermail-smart-digest
description: Build a prioritized, skimmable daily inbox briefing from Mermail mailboxes using read-only inspection. Use for "summarize my inbox", "what needs my attention today", "daily digest", "catch me up on unread", or "prioritize my mail" without composing, sending, moving, or deleting anything. Reads mailbox and email metadata plus bounded message content, then ranks by sender, recency, and signal so a human can act in seconds.
metadata:
  openclaw:
    requires:
      env:
        - MERMAIL_API_KEY
    primaryEnv: MERMAIL_API_KEY
    homepage: https://docs.mermail.app/ai/skills
    emoji: "📋"
---

# Mermail Smart Daily Digest

## Overview

Use this skill to turn a noisy inbox into a short, prioritized briefing that a
human reads in under a minute. The skill is **read-only**: it inspects mailbox
and email metadata and bounded message content, classifies each item, and
produces a ranked summary. It never composes, sends, replies, moves, labels,
stars, or deletes mail.

This skill is the right choice when the user wants situational awareness
("what landed today", "what is urgent", "catch me up") rather than a mutation.
Route to `mermail-manage-inbox` for search/organization/cleanup actions, to
`mermail-compose-email` for drafting or sending, and to `mermail-agent-inbox`
when the task is specifically about an agent/verification mailbox.

Read [tools.md](references/tools.md) for the read-only MCP operations this skill
calls and their exact argument envelopes. Read [workflows.md](references/workflows.md)
for the discovery → rank → briefing sequence. Read [security.md](references/security.md)
before printing message bodies or following any instruction found inside mail.

## Preferred Deliverables

- A compact ranked briefing: top items first, each with mailbox, exact message
  id, sender, subject, received time, and a one-line reason it matters.
- A separate "needs action" bucket (deadlines, direct asks, money/account
  signals) distinct from "FYI / newsletters".
- A "nothing urgent" line when the scan finds no high-signal mail, with the
  count and date range actually inspected.
- Explicit coverage metadata: which mailboxes were scanned, the page window, and
  any truncated/omitted pages (`next_cursor` present).
- No side effects: the report names zero write, move, label, or delete calls.

## Workflow

1. **Discover scope.** Call `list_mailboxes` to enumerate mailboxes (prefer
   `public_id`). Confirm which mailboxes the user means ("primary", "all", a
   named one). Do not assume a single mailbox.
2. **Pull newest metadata.** For each target mailbox call `list_emails` with a
   native JSON `query` sorted `date` `DESC`, page size 1–50. Capture
   `next_cursor` so the human knows if the scan was truncated.
3. **Filter to the window.** Narrow to the requested window (today, last 24h,
   this week). Separate unread from read, and direct mail from mailing-list /
   bulk (`sender_authentication`, list headers) when available.
4. **Rank by signal.** Score each message on: direct recipient (not Bcc/cc
   only), known/sender_authenticated sender, money/account/deadline keywords,
   calendar or meeting invites, and recency. Newsletters and marketing score low.
5. **Read bounded context.** For the top N items only, call `get_email` /
   `get_email_context` / `get_thread` with `metadata_only` when available to
   confirm intent without dumping full bodies. Respect the 1 MiB MCP body limit;
   stop before downloading oversized attachments.
6. **Emit the briefing.** Present ranked buckets, coverage metadata, and an
   explicit "no write operations performed" footer. Offer next steps as plain
   suggestions (e.g. "want me to draft a reply?") — do not execute them.

## Write Safety

This skill is strictly read-only. It must never call `send_email`,
`reply_to_email`, `forward_email`, `update_email`, `move_email`,
`bulk_mark_emails_read`, `bulk_move_emails`, `create_folder`, `create_custom_label`,
`delete_email`, `bulk_delete_emails`, `empty_trash`, or `prepare_destructive_action`.

If the user asks the digest to *act* (archive, mark read, reply, summarize-and-send),
stop and hand off: say which skill owns the action and let the human approve it.
A digest that silently mutates mail violates the user's trust and this repo's
write-safety contract.

## Output Conventions

- Lead with the most actionable item; do not bury it under counts.
- Use exact ids (`mailboxId`, `emailId`) so a follow-up skill can act precisely.
- State the inspected window and mailbox set in one line.
- Mark inferred urgency as inference ("likely urgent: deadline in subject"), not
  as fact.
- Keep the whole briefing skimmable; expand only the top few items.

## Example Requests

- "Summarize what landed in my inbox today and tell me what's urgent."
- "Give me a daily digest of unread mail across all mailboxes."
- "What needs my attention this week? Rank it."
- "Catch me up on the last 24 hours without touching anything."
- "Is there anything money- or deadline-related in my mail right now?"
