Create the PRD and tickets. Product: Acme Billing.
Do not search live Gmail, Slack, Jira, or Confluence. The only source is this Confluence page. Treat it as confirmed.

Title: Invoice export — client notes
Space: BILLING
URL: https://acme.example/wiki/invoice-export
Updated: 2026-10-25 by Dana Lee

Admins on the Invoices page need a CSV export of the invoice list.
Each file is capped at 5,000 rows.
The export must respect the current status filter: paid, unpaid, or overdue.
Only admins can run it, same as the Invoices page today.
Every export is written to the admin audit log with the exporting user's email and a timestamp, within 5 seconds of the file finishing.
Emailing the file to the requester is not decided. IT security has not confirmed attachment policy.
Do not redesign the page to feel more modern. That request was dropped.
