# Mermail Smart Digest workflows

Sequence the digest as discovery → rank → briefing. Keep it read-only end to end.

## Discover mailbox scope

- Call `list_mailboxes` and present the set (prefer `public_id`).
- Confirm the target: primary only, a named mailbox, or all mailboxes.
- Do not assume a single mailbox; a workspace can have several.

## Pull newest metadata

For each target mailbox:

```json
{
  "mailboxId": "MAILBOX_PUBLIC_ID",
  "query": { "folder": "inbox", "sortColumn": "date", "sortDirection": "DESC" },
  "limit": 50
}
```

Capture `next_cursor`. If present, the scan was truncated — say so in the briefing.

## Filter to the window

- Today, last 24h, this week, or "all unread" per the request.
- Split unread vs read; split direct mail vs mailing-list / bulk using
  `sender_authentication` and list headers when available.

## Rank by signal

Score each message:

1. Direct recipient (To, not Bcc/cc only).
2. Known / `sender_authenticated` sender.
3. Money, account, or deadline keywords in subject/snippet.
4. Calendar or meeting invites.
5. Recency (newest first within the window).

Newsletters, marketing, and notifications score low unless they carry an explicit
ask or deadline.

## Read bounded context

For the top N only, confirm intent with `get_email` / `get_email_context` /
`get_thread` and `metadata_only: true` when available. Respect the 1 MiB MCP
response limit; never download oversized attachments during a digest.

## Emit the briefing

- Lead with the most actionable item.
- Use buckets: "Needs action" (deadlines / direct asks / money / accounts),
  "Waiting on you" (thread replies), "FYI / newsletters".
- Include coverage metadata: mailboxes scanned, window, page truncation.
- Footer: "No write operations performed."
- Offer next steps as suggestions only; do not execute them.

## Recover from failure

- A timeout, transport error, or partial page returns a bounded report: list
  what was inspected and what failed; never auto-retry a write to "finish" the
  digest (there is no write to finish).
- If `get_email` is rate-limited, fall back to metadata already returned by
  `list_emails` and mark items "ranked on metadata only".
