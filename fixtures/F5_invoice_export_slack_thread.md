# F5 — Invoice export, filters, and audit (Slack thread)

Channel: #acme-invoices
Date: 2026-10-26

**Raj Patel** 09:01
Finance keeps asking for this: on the Invoices page, admins need to export the invoice list to CSV, capped at 5,000 rows per file.

**Raj Patel** 09:02
Before they export, they also need to filter by status — paid, unpaid, overdue — same three statuses as the invoices list.

**Priya Shah** 09:04
Got it. Anything on who can see this?

**Raj Patel** 09:05
Admins only, same as the Invoices page today.

**Dana Lee** 09:08
One more: every export needs to show up in the admin audit log, with the exporting user's email and a timestamp, within 5 seconds of the file finishing.

**Raj Patel** 09:10
Agreed, compliance asked for that one specifically.

**Dana Lee** 09:14
Separate thought — should we also email the exported file straight to the requester, instead of just downloading it? I'd need to check with IT security on attachment policy first, can't confirm today.

**Priya Shah** 09:15
Okay, we'll leave that open and follow up separately — don't block this sprint on it.

**Raj Patel** 09:18
Also, while we're in there, can we make the whole exports experience feel more polished and modern? It's a bit clunky right now.

**Priya Shah** 09:19
That one's too vague to scope — let's park it until there's something concrete.

**Raj Patel** 09:21
Fair enough. So for this sprint: the status filter on export, the 5,000 row cap, and the audit log entry.

## Related Jira

- DEV-42 — "Invoices page: add CSV export (legacy)" — Done — https://york-team-uvwkz4j6.atlassian.net/browse/DEV-42
