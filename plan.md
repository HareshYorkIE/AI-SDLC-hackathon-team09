# Vague Requirement → Verifiable Tickets

**Team:** 09 · **Members:** Prem, Purvesh, Haresh
**Challenge:** 02 · Stage: Planning · Time box: 2 hours

---

## 1. Problem

Requirements rarely arrive as clean specs. They arrive as Zoom transcripts, client emails, Slack threads, and one-liners like _"Add export."_ Today, a Product Manager turns these into tickets by hand, and four things go wrong:

- **Vague requirements slip through.** Words like "faster", "intuitive", or "ASAP" end up in tickets. They are only caught in backlog grooming or QA, days or weeks later, when fixing them is expensive.
- **Requirements get invented.** When information is missing, people fill the gap with their own assumptions. Nobody can later tell what the stakeholder actually asked for versus what was guessed.
- **Clarification stalls.** Open questions sit in someone's inbox. Nobody tracks who owes an answer, follow-ups are manual, and answers are scattered across email and chat.
- **Impact is discovered late.** A "small" change turns out to touch other services, teams, or screens, and that only shows up during development or QA.

## 2. Our solution

A tool that turns messy input into tickets **anyone can act on**, uses **gates** to reject anything vague or invented, and runs a **tracked clarification loop** with the client and PM until every question is answered and signed off.

> **We don't guess, and we don't invent.** Missing information becomes a clarifying question, never an assumption. And every question comes with a proposed answer, so stakeholders can simply confirm instead of starting from a blank page.

### What "verifiable" means

A ticket is verifiable when both of these are true:

1. **It traces back to the input:** every requirement is backed by an exact quote from the source, **or** by a recorded stakeholder answer on the clarification page.
2. **It is testable:** a QA engineer could write a pass/fail test from its acceptance criteria.

### Ticket statuses

| Status                | Meaning                                                                                            |
| --------------------- | -------------------------------------------------------------------------------------------------- |
| `ready`               | Traces to the input and passes every gate. A developer can start work                              |
| `needs_clarification` | Traces to the input, but is vague or incomplete. Comes with specific questions for the stakeholder |
| `rejected`            | Cannot be traced to the input. The requirement was invented                                        |

### PRD lifecycle statuses

| Status                   | Meaning                                                                    |
| ------------------------ | -------------------------------------------------------------------------- |
| `draft`                  | PRD generated from the input; gates run; open questions listed             |
| `awaiting_clarification` | Clarification page published; client and PM notified; reminders running    |
| `impact_review`          | Answers in; affected areas found by code scan are being confirmed by teams |
| `approved`               | Client and PM have both signed off; no open questions or conflicts         |
| `ticketed`               | Epics, stories, and tasks created for Dev, QA, and UI/UX with timelines    |

## 3. Scope

**In scope for the 2-hour build:** input ingestion, ticket generation, four quality gates, four test fixtures with planted defects, an answer key, an automated scorecard, and a **clarification-loop slice**: generating the clarification table (with a proposed solution for every question) and the stakeholder email, in dry-run mode.

**Designed here, built after the hackathon** (see Section 6 and Future Work): live Confluence publishing, real email sending, daily polling and reminders, GitHub/GitLab impact scanning, and Jira epic/story/task creation.

## 4. How it works (core pipeline)

```
Input (transcript / email / Slack / one-liner)
   │
   ▼
[1] EXTRACT   LLM lists every stated requirement + its exact source sentence
   │
   ▼
[2] DRAFT     LLM fills the ticket schema; missing info → open_questions
   │
   ▼
[3] GATES     Traceability · Vagueness · Completeness · Testability
   │
   ▼
[4] OUTPUT    Tickets with status, gate reasons, and clarifying questions
   │
   ▼
[5] CLARIFY   Open questions → clarification loop (Section 6)
```

### The four gates

| Gate             | Type      | Check                                                                                                                               | On failure            |
| ---------------- | --------- | ----------------------------------------------------------------------------------------------------------------------------------- | --------------------- |
| **Traceability** | Code      | Every `source_quote` is an exact substring of the input, or links to a recorded stakeholder answer                                  | `rejected`            |
| **Vagueness**    | Code      | Acceptance criteria contain no weasel words (fast, easy, intuitive, better, seamless, robust, ASAP…) without a measurable threshold | `needs_clarification` |
| **Completeness** | Code      | Actor, capability, and at least one acceptance criterion are present                                                                | `needs_clarification` |
| **Testability**  | LLM judge | Could QA write a pass/fail test from the criteria? Returns yes/no + reason                                                          | `needs_clarification` |

Three of the four gates are **deterministic code**, not AI. This makes the core guarantee (no invented requirements) reliable and explainable: a requirement either has a real quote from the input or it doesn't.

### Ticket schema

```json
{
  "id": "T1",
  "title": "string",
  "user_story": "As a <actor>, I want <capability> so that <benefit>",
  "acceptance_criteria": ["Given ... When ... Then ..."],
  "source_quotes": ["exact substring of the input"],
  "open_questions": ["Q1", "Q2"],
  "gate_results": {
    "traceability": "pass | fail: reason",
    "vagueness": "pass | fail: reason",
    "completeness": "pass | fail: reason",
    "testability": "pass | fail: reason"
  },
  "status": "ready | needs_clarification | rejected"
}
```

## 5. Proving it works: fixtures and answer key

We wrote four realistic inputs, each with **deliberately planted defects**, and an answer key stating what the tool must catch.

| #      | Input                        | Planted defects                                                                                                                   |
| ------ | ---------------------------- | --------------------------------------------------------------------------------------------------------------------------------- |
| **F1** | One-liner: _"Add export."_   | No format, no actor, no done condition                                                                                            |
| **F2** | Client email                 | "Faster and more intuitive" dashboard; missing actor; "ASAP" with no scope                                                        |
| **F3** | Zoom transcript (3 speakers) | Speakers contradict each other on export format; ambiguous "it"; an idea discussed and rejected that must **not** become a ticket |
| **F4** | Slack thread                 | Login mentioned in passing but never requested (an invention trap); one clean valid requirement as a control                      |

### Answer key format

```json
{
  "F3": {
    "expected_ready": ["CSV export for admins"],
    "expected_flags": ["contradiction:format", "ambiguous:it"],
    "must_not_appear": ["dark mode"]
  }
}
```

### Scorecard metrics

| Metric                      | What it measures                                               | Target |
| --------------------------- | -------------------------------------------------------------- | ------ |
| **Defects caught**          | Planted defects flagged, out of the total                      | All    |
| **Leaks**                   | Forbidden (invented) requirements appearing as `ready` tickets | 0      |
| **Recall**                  | Valid requirements correctly turned into tickets               | All    |
| **Questions with proposal** | Open questions that come with a proposed solution              | All    |

The scorecard runs with one command (`python eval.py`), so results are reproducible.

## 6. Clarification loop: from open questions to approved PRD

Gates tell us **what** is unclear. The clarification loop makes sure it actually **gets answered**, by the right person, with a record of who said what.

```
[A] PRD DRAFT        PRD + open questions + proposed solutions
   │
   ▼
[B] PUBLISH          Create Confluence clarification page (table below)
   │                 ✉ Email client + PM with the page link
   ▼
[C] DAILY CHECK      Read the Confluence table every day
   │                 ✉ Reminder for each unanswered question (until answered)
   ▼
[D] RE-VALIDATE      Feed answers back into the gates
   │                 New gaps or conflicts? → new questions → back to [B] (round 2, 3…)
   ▼
[E] IMPACT REVIEW    Scan GitHub / GitLab code for affected areas
   │                 ✉ Notify owners of each affected area; their questions join the table
   ▼
[F] SIGN-OFF         Client + PM approve on the page
   │                 ✉ Final PRD summary sent to everyone
   ▼
[G] PLAN             Epics → stories → tasks for Dev, QA, UI/UX
                     with proposed timeline and impact areas
                     ✉ Plan shared with client, PM, and team leads
```

### Confluence clarification table

One page per PRD. The client or PM fills in the **Answer** and **Decision** columns; everything else is generated.

| Q#  | Question                       | Source quote  | Why it matters               | Our proposed solution                      | Alternatives        | Owner  | Answer | Decision                 | Status | Due date   | Last reminder |
| --- | ------------------------------ | ------------- | ---------------------------- | ------------------------------------------ | ------------------- | ------ | ------ | ------------------------ | ------ | ---------- | ------------- |
| Q1  | Which export format is needed? | "Add export." | Defines scope and test cases | CSV first (opens in Excel, smallest build) | XLSX, PDF           | Client |        | Accept / Modify / Reject | open   | 2026-10-30 | 2026-10-29    |
| Q2  | Who is allowed to export?      | "Add export." | Drives permissions and UI    | Admins only in v1                          | All logged-in users | PM     |        |                          | open   | 2026-10-30 | —             |

- **Proposed solution is mandatory.** Every question ships with our recommendation and the reason, so stakeholders can answer with one click ("Accept") instead of writing an essay.
- **Status values:** `open` → `answered` → `closed`, or `conflict` when the client and PM disagree, or an answer contradicts an earlier one.
- **Answers become sources.** Once a question is closed, its answer is a valid source for the Traceability gate, with a link to the Confluence row.

### Emails at every stage

| Stage         | Trigger                                   | To                     | Content                                                 |
| ------------- | ----------------------------------------- | ---------------------- | ------------------------------------------------------- |
| Publish       | Clarification page created                | Client, PM             | Page link, number of questions, due date                |
| Reminder      | Question still `open` at daily check      | Question owner (cc PM) | Only the unanswered questions, with proposed solutions  |
| Escalation    | Question open past due date + 2 days      | PM, delivery lead      | Blocked items and what they block                       |
| Round N       | New gaps or conflicts after re-validation | Client, PM             | What changed, remaining questions, conflicts to resolve |
| Impact review | Code scan finds affected areas            | Owners of those areas  | What changes, where, and questions for that team        |
| Sign-off      | All questions `closed`, no conflicts      | Client, PM             | Final PRD summary and approval request                  |
| Plan ready    | Epics, stories, and tasks generated       | Client, PM, team leads | Plan, timeline, and impact areas                        |

**Reminder policy (configurable):** check daily; remind the owner every day a question is open; escalate after the due date + 2 days. Reminders stop automatically once a row is `closed`.

**Human in the loop:** the first email of every round is shown to the PM for approval before it is sent, so the tool never emails a client something wrong.

### Impact analysis with GitHub / GitLab

Before sign-off, we check what else the PRD touches, so it doesn't surface during development.

1. **Map requirements to code:** search the repos (GitHub / GitLab code search) for the entities, endpoints, screens, and tables each requirement mentions.
2. **Find dependencies:** follow imports, API consumers, shared libraries, and DB schemas to find downstream services.
3. **Scan safely:** run the existing code scanning (CodeQL / GitLab SAST, dependency scanning) on affected modules to flag security or compliance risks the change could introduce.
4. **Find owners:** use `CODEOWNERS` and recent commit history to identify the team for each affected area.
5. **Clarify with each area:** add their questions to the same Confluence table (Owner = that team) and email them.

Output: an **impact map** per PRD listing area, repo/path, owner, risk level (low / medium / high), and whether that team has confirmed.

### Change after sign-off

If the client changes a requirement after approval, the PRD goes back to `awaiting_clarification` for **only the affected rows**, and the tool reruns impact analysis and the plan. Every version of the PRD is kept with a diff, so nobody loses track of what was agreed.

## 7. From approved PRD to work items

Only after **both** the client and PM sign off, the tool generates the delivery plan.

```
Epic                    one per major capability in the PRD
 └── Story              user story + acceptance criteria (passes all 4 gates)
      ├── Dev task      backend / frontend / data changes, with impact areas
      ├── QA task       test cases generated from the acceptance criteria
      └── UI/UX task    screens or flows that change, with design notes
```

### Work item schema

```json
{
  "id": "E1-S2-T3",
  "type": "epic | story | task",
  "discipline": "dev | qa | ui_ux | devops",
  "title": "string",
  "description": "string",
  "acceptance_criteria": ["Given ... When ... Then ..."],
  "sources": ["source quote or Confluence answer link (Q#)"],
  "impact_areas": [
    {
      "repo": "string",
      "path": "string",
      "owner": "string",
      "risk": "low | medium | high"
    }
  ],
  "dependencies": ["E1-S1-T2"],
  "estimate_days": 2,
  "proposed_start": "2026-11-02",
  "proposed_end": "2026-11-04"
}
```

- **Proposed timeline:** estimates come from the LLM plus task size, ordered by dependencies (e.g. UI/UX design before frontend, Dev before QA). The PM adjusts it; it is a proposal, not a commitment.
- **Impact areas on every task:** each Dev task lists the repos, paths, and owners from the impact map, so nothing is changed by surprise.
- **Definition of Ready:** a story can only be created if it passes all four gates and has no open questions.
- **Export:** push to Jira (epic links, sub-tasks) or GitHub / GitLab Issues.

## 8. Tech stack

| Layer         | Choice                                                                   | Why                                              |
| ------------- | ------------------------------------------------------------------------ | ------------------------------------------------ |
| Language      | Python 3.10+                                                             | Fastest path for LLM work                        |
| LLM           | Claude API (`anthropic` SDK), temperature 0                              | Strong at extraction; stable, repeatable results |
| Schema        | Pydantic                                                                 | Malformed LLM output fails loudly, not silently  |
| Gates         | Plain Python                                                             | Deterministic and explainable                    |
| UI            | Streamlit                                                                | Live demo: paste input, see tickets              |
| Secrets       | python-dotenv                                                            | API keys stay out of the repo                    |
| Clarification | Confluence REST API (`atlassian-python-api`)                             | Page with table that client and PM already use   |
| Email         | SMTP / SendGrid, dry-run mode for the demo                               | Notifications at every stage                     |
| Scheduler     | Cron or GitHub Actions scheduled workflow                                | Daily check and reminders, no server to run      |
| Impact scan   | GitHub / GitLab APIs (`PyGithub`, `python-gitlab`), CodeQL / GitLab SAST | Find affected code, owners, and risks            |
| Tracker       | Jira REST API, GitHub / GitLab Issues                                    | Create epics, stories, and tasks                 |
| Dev tools     | Claude Code, Cursor, GitHub                                              | Parallel work across three people                |

Deliberately **not** used: agent frameworks, vector databases, databases, Docker. The Confluence page is the source of truth for answers, and a JSON file stores loop state. Extra infrastructure adds setup and failure points without improving this problem.

### Repository layout

```
vague2tickets/
├── schema.py          # Ticket, question, and work item models
├── prompts.py         # All LLM prompts
├── pipeline.py        # Extract + draft
├── gates.py           # Four gates + status logic
├── clarify/
│   ├── table.py       # Build clarification table + proposed solutions
│   ├── confluence.py  # Publish / read the Confluence page
│   ├── notifier.py    # Email templates + sending (dry-run by default)
│   └── reminders.py   # Daily check, reminders, escalation
├── impact.py          # GitHub / GitLab scan → impact map
├── planner.py         # Epics, stories, tasks, timeline
├── fixtures/          # F1–F4 inputs
├── answer_key.json    # Expected results
├── eval.py            # Scorecard
├── app.py             # Streamlit demo
└── export.py          # Jira / GitHub / GitLab export
```

## 9. Team and timeline

| Role                  | Owns                                                             |
| --------------------- | ---------------------------------------------------------------- |
| **A: Test owner**     | Fixtures, answer key, eval scorecard                             |
| **B: Pipeline owner** | Extraction, drafting, gates, prompts, proposed solutions         |
| **C: Product owner**  | Streamlit UI, clarification table + email preview, export, pitch |

| Time      | A                                                         | B                                            | C                                             |
| --------- | --------------------------------------------------------- | -------------------------------------------- | --------------------------------------------- |
| 0:00–0:10 | **All:** agree on schema and definitions; verify API keys |                                              |                                               |
| 0:10–0:35 | Write F1–F4 + answer key                                  | Extract + draft (on F1)                      | Streamlit shell                               |
| 0:35–1:00 | Eval script                                               | Four gates + status logic                    | Status colors, gate reasons, questions in UI  |
| 1:00–1:10 | **All:** integration, all four fixtures end to end        |                                              |                                               |
| 1:10–1:35 | Analyze failures                                          | Tune prompts; proposed solution per question | Clarification table + email preview (dry-run) |
| 1:35–1:45 | **All:** final eval run; capture scorecard                |                                              |                                               |
| 1:45–2:00 | **All:** code freeze; rehearse demo                       |                                              |                                               |

## 10. Bonus features (if time allows)

- **Tracker-ready output:** export `ready` tickets as Jira CSV or GitHub Issues JSON, ready to import.
- **Developer-agent check:** a second LLM acts as a developer and lists any questions it would need answered before building a ticket. An empty list means the ticket is truly actionable.
- **Live Confluence page:** publish the clarification table to a real Confluence space.
- **Conflict detector:** flag when the client's answer contradicts the PM's answer or an earlier source quote.

## 11. Demo (3 minutes)

1. **"Add export."** → `needs_clarification` with three specific questions. _We don't guess._
2. **Clarification table + email preview** → every question comes with our proposed solution; the client just accepts or edits.
3. **F4 Slack thread** → the login trap is not turned into a ticket; the real requirement comes back `ready`. _We don't invent._
4. **Scorecard** across all four fixtures.
5. **Judge's turn:** type any vague requirement and watch the gates respond.

## 12. Where this fits in the SDLC

| Today                                                  | With this tool                                          |
| ------------------------------------------------------ | ------------------------------------------------------- |
| Original wording is lost in notes                      | Every ticket links to an exact source quote or answer   |
| Vague words caught in grooming or QA (late, expensive) | Caught in seconds, at planning                          |
| Days of back-and-forth to clarify                      | One tracked page, proposed answers, automatic reminders |
| Gaps filled with assumptions                           | Invented requirements rejected                          |
| Impact discovered during development                   | Affected code and owners found before sign-off          |
| Tickets written by hand                                | Epics, stories, and tasks per discipline, with timeline |

## 13. Future work

- **Full clarification loop live:** Confluence publishing, real email, daily scheduler, and escalation (designed in Section 6).
- **Impact analysis:** GitHub / GitLab code search, dependency graph, CodeQL / SAST, and `CODEOWNERS` routing.
- **Existing PRD merge:** load current PRDs from Confluence and flag contradictions with new requests instead of silently overwriting.
- **Delivery planning:** epics, stories, and tasks with timelines, pushed directly into Jira, Linear, or GitHub / GitLab.
- **Slack / Teams notifications** alongside email, for teams that live in chat.
- **Status dashboard:** open questions per PRD, average answer time, and who is blocking.
- **Estimation from history:** use past velocity from Jira to make proposed timelines more realistic.

## 14. Risks and mitigations

| Risk                                         | Mitigation                                                                   |
| -------------------------------------------- | ---------------------------------------------------------------------------- |
| LLM returns malformed JSON                   | Pydantic validation + one automatic retry                                    |
| Non-deterministic results between runs       | Temperature 0; deterministic code gates                                      |
| Over-flagging valid requirements             | The F4 control requirement measures false positives                          |
| Live demo failure                            | Pre-recorded scorecard screenshot and CLI fallback                           |
| Wrong or spammy email to a client            | PM approves the first email of each round; dry-run by default; daily cap     |
| Stakeholders don't answer                    | Daily reminders, escalation after due date, proposed solutions to cut effort |
| Client and PM give conflicting answers       | Row marked `conflict`; both notified; sign-off blocked until resolved        |
| Code scanning exposes source code            | Read-only tokens; scan runs inside our CI; only file paths go to the LLM     |
| Impact analysis misses or over-reports areas | Each affected team confirms its area before sign-off                         |
| Timeline estimates are unrealistic           | Labeled as proposals; PM adjusts before sharing                              |
