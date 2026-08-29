# Mermail Smart Digest tool contract (read-only)

Read this reference when constructing MCP calls for the daily digest. The digest
is strictly read-only: it inspects mailboxes and messages but never mutates them.

## Native MCP envelope

Use the exact tool identifier exposed by the current host. Claude may expose
`Mermail:list_emails`; another host may use a different namespace or bare
`list_emails`. Do not manually add, strip, or invent a prefix. At the protocol
boundary the catalog name is bare.

Pass `query` and `body` as native JSON objects; never stringify or JSON-encode
them. Common fields are:

```json
{
  "mailboxId": "MAILBOX_PUBLIC_ID",
  "emailId": "EMAIL_ID",
  "query": {},
  "body": {}
}
```

Use `mailboxId` from `list_mailboxes`, preferably `public_id`. Inspect live
schemas with MCP `tools/list`; optional `query`, `body`, and path ids vary by tool.

## Read-only tool map

The digest uses only these tools. None of them change workspace or mailbox state:

| Class | Tools |
| --- | --- |
| Mailbox discovery | `list_mailboxes` |
| Message discovery | `list_emails`, `search_emails`, `get_email`, `get_email_context`, `get_thread` |

`list_mailboxes` is owned by workspace discovery (see `mermail-administer-workspace`
coverage); `list_emails`, `search_emails`, `get_email`, `get_email_context`, and
`get_thread` are owned by the `mermail-manage-inbox` domain. This digest reuses
them read-only and introduces no new tool ownership.

## Message discovery

Newest inbox metadata:

```json
{
  "mailboxId": "MAILBOX_PUBLIC_ID",
  "query": {
    "folder": "inbox",
    "unreadOnly": false,
    "sortColumn": "date",
    "sortDirection": "DESC"
  },
  "limit": 50
}
```

There is no `sort: "date_desc"` shortcut. Use `sortColumn: "date"` and
`sortDirection: "DESC"`. The host-qualified identifier (e.g. `Mermail:list_emails`)
is the exact tool identifier exposed by the current host. Do not manually add,
strip, or invent a prefix.

## Bounded content reads

For ranking, prefer metadata. When you need to confirm intent, read bounded
content with `get_email`, `get_email_context`, or `get_thread`:

- Pass `metadata_only: true` when you only need headers/sender/subject.
- `get_email_context` returns bounded conversation context; respect the 1 MiB
  MCP response cap and `content_omitted` markers.
- `next_cursor` on `list_emails` means more pages exist; report truncation rather
  than silently capping the scan.

## What this skill must NOT call

Never invoke any write, move, label, or delete tool from the digest:

- No `update_email`, `move_email`, `bulk_mark_emails_read`, `bulk_move_emails`.
- No `create_folder`, `update_folder`, `delete_folder`, `create_custom_label`,
  `update_custom_label`, `delete_custom_label`.
- No `download_attachment` on oversized bodies; no `delete_email`,
  `bulk_delete_emails`, `empty_trash`.
- No `prepare_destructive_action` — the digest performs no destructive operation.

If a follow-up action is needed, hand off to the owning skill and let the human
approve the external effect.
