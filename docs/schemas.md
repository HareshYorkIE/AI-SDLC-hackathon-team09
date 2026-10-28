# Artifact schemas

Shapes for every JSON artifact in `output/<run-id>/`. Field values shown as `a | b` are enums.

## vague2tickets.config.json (repo root, not per-run)

Asked once, then reused for every run. Holds no secrets — auth lives in the Atlassian MCP connection.

```json
{
  "product": {
    "name": "Acme Billing",
    "competitors": ["optional, found by web search when empty"]
  },
  "jira": {
    "site": "https://acme.atlassian.net",
    "project_key": "ACME",
    "issue_types": { "epic": "Epic", "story": "Story", "task": "Sub-task" }
  },
  "confluence": {
    "site": "https://acme.atlassian.net/wiki",
    "space_key": "PROD",
    "parent_page_title": "PRDs"
  },
  "github": {
    "repos": ["org/repo"]
  }
}
```

## Source

Used wherever an item cites where it came from.

```json
{ "type": "quote", "text": "exact substring of transcript.md" }
{ "type": "decision", "id": "D1" }
```

## requirements.json (extractor)

```json
{
  "call": {
    "title": "string",
    "date": "YYYY-MM-DD",
    "participants": ["string"]
  },
  "requirements": [
    {
      "id": "R1",
      "statement": "short paraphrase",
      "quote": "exact substring of transcript.md",
      "speaker": "string",
      "kind": "stated | rejected | mentioned_only"
    }
  ],
  "contradictions": [
    {
      "topic": "string",
      "quotes": ["quote A", "quote B"],
      "requirement_ids": ["R2", "R3"]
    }
  ],
  "ambiguities": [
    {
      "text": "the ambiguous phrase",
      "quote": "exact substring",
      "why": "string"
    }
  ]
}
```

`rejected`: discussed and turned down. `mentioned_only`: referenced but never requested. Neither may become a ticket.

## questions.json (clarifier)

```json
[
  {
    "id": "Q1",
    "about": ["R1"],
    "question": "string",
    "why_it_matters": "string",
    "suggested_answer": "string",
    "evidence": {
      "requirement": [
        { "type": "quote", "text": "exact substring of transcript.md" }
      ],
      "competitors": [
        {
          "product": "string",
          "finding": "how they handle it",
          "pain_point": "string | null",
          "url": "https://..."
        }
      ]
    },
    "competitors_note": "string, required when competitors is empty: what was searched and why nothing applied",
    "reason": "why we recommend it, citing the quotes and competitor findings above",
    "alternatives": ["string"],
    "depends_on": ["Q1"],
    "raised_by": "extractor | gatekeeper | impact-analyst",
    "status": "open | answered | deferred"
  }
]
```

## decisions.json (skill)

```json
[
  {
    "id": "D1",
    "question": "Q1",
    "answer": "string, the user's words",
    "accepted_suggestion": true,
    "answered_by": "name",
    "at": "ISO 8601"
  }
]
```

## tickets.json (drafter)

```json
[
  {
    "id": "T1",
    "title": "string",
    "user_story": "As a <actor>, I want <capability> so that <benefit>",
    "acceptance_criteria": ["Given ... When ... Then ..."],
    "requirement_ids": ["R1"],
    "sources": [
      { "type": "quote", "text": "..." },
      { "type": "decision", "id": "D1" }
    ],
    "open_questions": ["Q4"],
    "provenance": {
      "generated_by": "vague2tickets",
      "run_id": "YYYY-MM-DD-feature-slug",
      "local_id": "T1"
    },
    "references": [
      {
        "kind": "jira",
        "relation": "changes | recreates | relates",
        "key": "DEV-12",
        "url": "string | null",
        "title": "string",
        "why": "why this prior issue is the same capability"
      }
    ]
  }
]
```

`provenance` is on every ticket. `generated_by` is always `vague2tickets`. `run_id` is the run folder name. `references` is `[]` when `transcript.md` has no matching `Related Jira` issue. A reference key must be copied from that heading. `changes`: the prior issue is this same capability and this ticket alters it. `recreates`: that prior issue is being replaced because the requirement changed. `relates`: same area, not a change of that issue.

## gate_report.json (gatekeeper)

```json
{
  "tickets": [
    {
      "id": "T1",
      "gates": {
        "traceability": "pass | fail: reason",
        "vagueness": "pass | fail: reason",
        "completeness": "pass | fail: reason",
        "resolution": "pass | fail: reason",
        "testability": "pass | fail: reason"
      },
      "status": "ready | needs_clarification | rejected",
      "new_questions": ["question text the clarifier should turn into a Q#"]
    }
  ],
  "summary": { "ready": 0, "needs_clarification": 0, "rejected": 0 }
}
```

## impact.json (impact-analyst)

```json
{
  "comparisons": [
    {
      "ticket": "T1",
      "repo": "org/repo",
      "gap": "missing | partial | already_done",
      "current_behavior": "what the code does today",
      "requirement_gap": "what the ticket asks that the code does not do",
      "evidence": ["src/export/handler.py:42 — short quote from the file"],
      "prior_work": [
        {
          "kind": "code",
          "relation": "changes",
          "repo": "org/repo",
          "path": "src/invoices/List.tsx",
          "why": "list page exists; status filter does not"
        }
      ]
    }
  ],
  "areas": [
    {
      "ticket": "T1",
      "repo": "org/repo",
      "path": "src/export/",
      "owner": "@team-or-person (from CODEOWNERS or git history)",
      "why": "string",
      "risk": "low | medium | high",
      "scan_findings": [
        "existing CodeQL / SAST alert IDs or summaries in this area"
      ]
    }
  ],
  "design_elements": [
    {
      "ticket": "T1",
      "repo": "org/frontend-repo",
      "path": "src/components/Modal.tsx",
      "pattern": "existing modal with confirm/cancel actions",
      "evidence": "src/components/Modal.tsx:12 — <Modal onConfirm...>"
    }
  ],
  "new_questions": ["question text for the clarifier"],
  "not_scanned": ["repos or areas that could not be accessed, with reason"]
}
```

`design_elements` holds existing frontend patterns found for UI-touching tickets — context for the `ui_ux` task, never a requirement. Omit a ticket here (or note it in `not_scanned`) when it touches no UI or nothing matched.

`prior_work` is on every comparison. It lists a code path only when `gap` is `partial` or `already_done` (this ticket changes something that already exists). It is `[]` when `gap` is `missing`. The path must be one you read. It is context, not a requirement source.

## team_directory.json (fixture, simulates the hub.york.ie MCP team directory)

```json
{
  "team": [
    {
      "id": "u1",
      "name": "string",
      "email": "string",
      "discipline": "dev | qa | ui_ux | devops",
      "technologies": ["string"],
      "experience_years": 6,
      "seniority": "junior | mid | senior",
      "max_complexity": "low | medium | high",
      "current_load": 2
    }
  ]
}
```

## plan.json (planner)

```json
{
  "proposal": true,
  "epics": [
    {
      "id": "E1",
      "title": "string",
      "provenance": {
        "generated_by": "vague2tickets",
        "run_id": "YYYY-MM-DD-feature-slug",
        "local_id": "E1"
      },
      "references": [
        {
          "kind": "jira | code",
          "relation": "changes | recreates | relates",
          "key": "DEV-12",
          "url": "string | null",
          "repo": "string",
          "path": "string",
          "why": "string"
        }
      ],
      "stories": [
        {
          "id": "E1-S1",
          "ticket_id": "T1",
          "title": "string",
          "user_story": "string",
          "acceptance_criteria": ["string"],
          "sources": [{ "type": "quote", "text": "..." }],
          "provenance": {
            "generated_by": "vague2tickets",
            "run_id": "YYYY-MM-DD-feature-slug",
            "local_id": "E1-S1",
            "ticket_id": "T1"
          },
          "references": [
            {
              "kind": "jira | code",
              "relation": "changes | recreates | relates",
              "key": "DEV-12",
              "url": "string | null",
              "repo": "string",
              "path": "string",
              "title": "string",
              "why": "string"
            }
          ],
          "tasks": [
            {
              "id": "E1-S1-T1",
              "discipline": "dev | qa | ui_ux | devops",
              "title": "string",
              "description": "string",
              "provenance": {
                "generated_by": "vague2tickets",
                "run_id": "YYYY-MM-DD-feature-slug",
                "local_id": "E1-S1-T1",
                "ticket_id": "T1"
              },
              "references": [
                {
                  "kind": "jira | code",
                  "relation": "changes | recreates | relates",
                  "key": "DEV-12",
                  "url": "string | null",
                  "repo": "string",
                  "path": "string",
                  "why": "string"
                }
              ],
              "test_cases": [
                "string, qa tasks only: one per acceptance criterion"
              ],
              "edge_cases": [
                "string, qa tasks only: boundary values, invalid/missing input, empty/max-size data, repeated or concurrent actions, error/timeout states"
              ],
              "design_references": [
                {
                  "repo": "string",
                  "path": "string",
                  "pattern": "existing component or style this task reuses",
                  "note": "existing pattern, not a requirement"
                }
              ],
              "impact_areas": [
                {
                  "repo": "string",
                  "path": "string",
                  "owner": "string",
                  "risk": "low | medium | high"
                }
              ],
              "dependencies": ["E1-S1-T0"],
              "estimate_days": 2,
              "confidence": "low | medium | high",
              "proposed_start": "YYYY-MM-DD",
              "proposed_end": "YYYY-MM-DD",
              "priority": "Highest | High | Medium | Low",
              "assignee_sequence": 1,
              "assignee": {
                "name": "string",
                "email": "string",
                "source": "hub.york.ie | fixtures/team_directory.json (simulated)",
                "reason": "technology match, complexity fit, and why load or experience broke the tie"
              }
            }
          ]
        }
      ]
    }
  ]
}
```

`test_cases` and `edge_cases` apply only to `qa` tasks; `design_references` applies only to `ui_ux` tasks, copied from that ticket's `design_elements` in `impact.json` (or an explicit none-found note in `description` when there are none). Omit the fields that don't apply to a task's discipline.

`provenance` is on every epic, story, and task. `generated_by` is always `vague2tickets`. `references` is the ticket's Jira references plus that ticket's `prior_work` from `impact.json`. Use `[]` when neither exists. A Jira reference keeps `key` and `url`. A code reference keeps `repo` and `path`. Do not invent either.

`priority` is the task's importance (`Highest`/`High`/`Medium`/`Low`), from its complexity and how many other tasks depend on it. `assignee_sequence` is that task's 1-based position in its assignee's own queue, ordered by `proposed_start` then `priority`. Sequence numbers restart at 1 for each distinct `assignee.email`; they do not compare across people. When one person holds every task of a discipline, this is the only field that says which one they do first.

## progress.json (skill)

Written the first time the Confluence progress page is created. Later steps update that same page.

```json
{
  "title": "PRD: Roster CSV export (2026-10-28-roster-csv-export)",
  "space_key": "PROD",
  "page_id": "string",
  "url": "string",
  "status": "in_progress | approved"
}
```

## published.json (skill)

```json
{
  "confluence_page": {
    "title": "string",
    "space_key": "PROD",
    "page_id": "string",
    "url": "string",
    "action": "created | updated"
  },
  "jira_issues": [
    {
      "local_id": "E1",
      "issue_type": "Epic",
      "key": "ACME-101",
      "url": "string",
      "action": "created | linked_existing"
    },
    {
      "local_id": "E1-S1",
      "issue_type": "Story",
      "key": "ACME-102",
      "url": "string",
      "action": "created | linked_existing"
    },
    {
      "local_id": "E1-S1-T1",
      "issue_type": "Sub-task",
      "key": "ACME-103",
      "url": "string",
      "action": "created | linked_existing"
    }
  ]
}
```

## answer_key.json (fixtures)

```json
{
  "F4": {
    "expected_ready": ["Admins filter invoices by status"],
    "expected_flags": [],
    "must_not_appear": ["login", "Google SSO"]
  },
  "F5": {
    "expected_ready": ["string, one per ticket that should reach ready"],
    "expected_needs_clarification": ["string, one per ticket that should be blocked by an unresolved question"],
    "expected_flags": ["vague:<word> | open:<topic> | missing:<field> | contradiction:<topic> | ambiguous:<word>"],
    "expected_references": [
      { "ticket_about": "string, enough to match the ticket by meaning", "key": "DEV-42", "relation": "changes | recreates | relates" }
    ],
    "must_not_appear": ["string"],
    "notes": "string, what this fixture is targeting and why"
  }
}
```

`expected_ready` and `must_not_appear` are required; the evaluator's core fixture metrics (Recall, Leaks) always run on them. `expected_needs_clarification`, `expected_flags`, `expected_references`, and `notes` are optional per fixture — add them when the fixture targets something specific (a gate, a field) beyond plain ready/rejected classification. `expected_needs_clarification` checks the Resolution gate: each named ticket must exist with status `needs_clarification` and a non-empty `open_questions`. `expected_references` checks `tickets.json`: the matched ticket's `references` must contain an entry with that `key` and `relation`.
