Create the PRD and tickets for invoice export. Product: Acme Billing.
Do not search live Gmail, Slack, Jira, or Confluence. These four pasted sources are the full confirmed set.

--- Gmail ---
From: Dana Lee <dana@acme.example>
To: Priya Shah <priya@york.example>
Date: 2026-10-21
Subject: Export

Can we add export? Needs to feel modern and be ready ASAP.

--- Slack #acme-invoices, 2026-10-24 ---
Raj Patel 10:03
On the Invoices page, admins must export the invoice list to CSV, capped at 5,000 rows per file. Before export they filter by status: paid, unpaid, overdue. Admins only.

Dana Lee 10:08
Every export goes in the admin audit log with the user's email and a timestamp, within 5 seconds of the file finishing.

Priya Shah 10:15
The "modern / ASAP" line from email is too vague. Park it.

--- Confluence: Invoice export notes, 2026-10-25, Dana Lee ---
https://acme.example/wiki/invoice-export
Same scope as Slack: CSV, 5,000 rows, status filter, admins only, audit log within 5 seconds.
Emailing the file to the requester is still open. IT security has not answered.

--- One-liner, Jira comment, Dana Lee, 2026-10-20 ---
Add export.

Related Jira: DEV-42 — Invoices page: add CSV export (legacy) — Done — https://york-team-uvwkz4j6.atlassian.net/browse/DEV-42
