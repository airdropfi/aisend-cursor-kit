---
name: check-deliverability
description: Check what happened to emails sent through AISend and whether addresses are worth sending to. Use when the user asks if an email was delivered, bounced, opened or clicked, wants a summary of recent mail, or wants to validate an address before sending.
license: MIT
compatibility: Needs the AISend MCP server connected and signed in. Read-only except that validate_email uses the account's monthly validation quota.
---

# Check delivery and validate addresses

This skill sends nothing.

## Steps

1. For a recent-mail overview, call `list_emails` (filter with `status` or `search`; page with `nextOffset` when present).
2. For one email, call `get_email_status` with its id.
3. Report status exactly as returned: queued, sending, sent, delivered, failed, bounced or complained, plus opened and clicked. Never say "delivered" unless the status says so, and do not guess causes the tools did not return.
4. To check an address before sending, call `validate_email`. It checks format, MX records, disposable providers and role addresses. Each call counts against the monthly validation quota, so use it for a handful of addresses, not long lists.
5. If the user wants to act on a problem (for example resend), switch to the `send-an-email` skill and get approval for the exact message again.
