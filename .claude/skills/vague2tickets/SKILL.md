---
name: vague2tickets
description: Turn a client call, Slack thread, email thread, or vague requirement into a verified PRD and Jira work items (epics, stories, tasks). Open decisions are a grill — design tree, one frontier per round, a recommended answer, then wait — followed by gate checks, impact analysis, and PM approval. Use when asked to create a PRD, tickets, stories, or tasks from a call, meeting, transcript, Slack thread, client email, or vague requirement. If a run folder already exists, or the user says continue or resume, pick up at the first unfinished step.
---

# vague2tickets

Example request: _"Client mentioned bulk export in the Friday call 'Acme weekly sync'. Create the PRD and tickets."_

Rules, gates, run folder, and statuses live in `AGENTS.md`; artifact shapes in `docs/schemas.md`. Run the steps in order. Each step ends only when its **Done when** holds. After every step, append a line to `audit.log`, then update the Confluence progress page (below). Do that update before you wait on the user, so a closed chat still leaves the latest requirement on the page. Fixture mode does not write to Confluence.

Tracker: **Jira**. PRD destination: **Confluence**. Both need the Atlassian MCP connected (Cursor Settings → MCP, or Claude Code), signed into the real site.

## 1. Start the run

Pick `<run-id>` = `YYYY-MM-DD-<feature-slug>` (today's date). Create `output/<run-id>/` only if that folder does not exist. If it exists, resume (below). Do not start a second folder for the same feature on the same day. If the user said "fixture F<n>" for a file that exists under `fixtures/` (e.g. `F1`…`F5`, or any later one added there), read that file and run in **fixture mode**. Fixture mode is reproducible and needs no human: step 2 copies the fixture verbatim, step 4 marks every question `deferred` without asking, step 7 is skipped, step 10 auto-approves, and step 11 is skipped. It still ends at step 9's scorecard.

Read `vague2tickets.config.json` at the repo root (shape in `docs/schemas.md`). If it is missing (or has no `product.name`) and this is not fixture mode, ask the user once for: the client's product name (used to find its competitors), Jira site, project key, and the issue type names for Epic / Story / Task; Confluence space key and, optionally, a parent page title; GitHub repos (`owner/name`) to compare against. Write their answers to `vague2tickets.config.json` so later runs don't ask again.

**Done when:** the run folder exists, `audit.log` has its first line, and (outside fixture mode) the config file exists.

## Resume

Closing the chat loses the conversation. It does not lose the run. `output/<run-id>/` is the checkpoint. On "continue", "resume", or any request whose `<run-id>` folder already exists, read the files and `audit.log`, tell the user the last finished step, and start at the first unfinished one. Do not redo a step whose **Done when** already holds. Do not re-ask a question that already has a `D#`.

| First thing missing | Resume at |
| --- | --- |
| `transcript.md`, or the user never confirmed the source set | 2. Find the sources |
| `requirements.json` | 3. Extract |
| `questions.json`, or any question still `open` | 4. Clarify — ask only the open frontier |
| `prd.md` or `tickets.json` | 5. Draft |
| `gate_report.json` | 6. Gate |
| `impact.json` | 7. Impact analysis |
| `plan.json` | 8. Plan |
| `scorecard.md`, or its verdict is **FAIL** | 9. Evaluate |
| no `APPROVE <run-id>` line in `audit.log` | 10. PM approval |
| `APPROVE` recorded, but `published.json` has no Confluence page, or is missing an approved Jira issue | 11. Publish. Create or update the PRD page first. Search Jira for `v2t:<run-id>` and create only the issues that are not already there. Then add the page URL to each issue and the issue links back onto the page |
| `published.json` lists every approved issue | Stop. Show the links. Do not create them again |

Append `resume | <step> | <run-id>` to `audit.log` before continuing, then update the progress page to say the run resumed and which step is next.

## Progress page

One Confluence page per run, titled `PRD: <feature> (<run-id>)`. Create it at the end of step 1. Every later step updates that same page. Never create a second page for the same run.

The space is `confluence.space_key` in `vague2tickets.config.json`. If it is empty, ask once at step 1 and save the answer. If Confluence is disconnected or returns 404, append `confluence | skipped | <reason>` to `audit.log`, keep going, and say the page was not saved. The run folder is still the checkpoint.

Page status at the top is `In progress` until the PM approves, then `Approved`. Under it: last finished step, what is next, and the time.

Then include only the sections that already exist:

- **Sources:** each confirmed call, Slack thread, email, and Confluence page, with link and first line. Sources not yet confirmed are listed as "found, not confirmed".
- **Requirements:** each `stated`, `rejected`, and `mentioned_only` item, with its quote.
- **Clarifications:** every question, the suggested answer, why (client quote, then competitor and URL), and the outcome (`D#`, `open`, or `deferred`).
- **Decisions** from `decisions.json`.
- **PRD** sections already drafted.
- **Gate summary**, **impact**, **plan** (labeled proposal), and **scorecard** verdict, once those files exist.
- **Jira issues:** empty until publish. Step 11 fills the keys and URLs.

Record the page id and URL in `output/<run-id>/progress.json` the first time the page is created. Later steps update that page. Step 11 reads `progress.json` and updates it in place. It does not create another page.

## 2. Find the sources

Search every connected source for the named feature. Run the searches together. Do not stop after the first hit. A source the user named is the place to start, not the only place to look.

1. Resolve relative dates to absolute ones ("last Friday" → `2026-10-23`) and state the date you resolved.
2. Search each source that is connected. Skip one only when its MCP is disconnected or errors, and say which and why. The others still run.
   - **Call:** meeting tool (Fireflies, Zoom, Google Meet, Teams, Otter, Gong). Search by feature words, title, date, and participants. Read each matching transcript in full.
   - **Slack:** Slack MCP, read-only. Search the named channel, or the workspace if no channel was named. Read each matching thread in full. Do not send, schedule, or react.
   - **Email:** Gmail MCP, read-only. Search by feature words, sender, subject, and date. Read each matching thread in full, oldest message first. Do not send, draft, reply, label, or delete.
   - **Confluence:** configured space. Search pages that mention the feature. Read each matching page.
   - **Jira:** configured project. Search existing issues that mention the feature. Read each match. These are context so the same work is not created twice. They are not a new requirement.
3. Show one list. For each hit: source type, title or subject, date, people, link, and the first line. If nothing is connected or nothing matches, ask the user to paste the text or give a file path.
4. Ask the user to confirm the set. The default is every hit that mentions the feature. The user can drop a hit.
5. Save every confirmed conversation and page **verbatim** into `transcript.md`, each under its own heading: source type, title or subject, date, link, participants. A call keeps its transcript. Slack keeps each message as `**author** timestamp` plus the exact text. Email keeps each message as `**from** to **to** — date` plus the exact body. A confirmed Confluence page keeps its body. Strip signature blocks and quoted-reply chrome only when they duplicate an earlier message in the same thread. Never edit or summarize a body: traceability depends on it. Put confirmed Jira issues under a final heading `Related Jira`, with key, title, status, and URL only.

**Done when:** every connected source was searched or marked unreachable, the user has confirmed the set, and `transcript.md` holds the full unedited text of every confirmed conversation and page.

## 3. Extract

Dispatch the **extractor** subagent with the path to `transcript.md` and the feature the user named.

**Done when:** `requirements.json` exists and every quote in it is an exact substring of `transcript.md` (checked by the extractor with code).

## 4. Clarify — grill

1. Dispatch the **clarifier** with `requirements.json` (plus any `new_questions` from `gate_report.json` or `impact.json` on later rounds). It writes the design tree into `questions.json` (`depends_on` is the branch).
2. Read `.claude/skills/grilling/SKILL.md` and run the interview that way. The frontier is every `open` question whose `depends_on` are all `answered` or `deferred`. A question that still depends on an open question waits for a later round.
3. Ask the whole frontier in one round, in the grilling format. The recommended answer (➡️) is that question's `suggested_answer`. **Why** (💡) is its `reason`, followed by sources: the client quote (`evidence.requirement`), then each competitor finding as `<product>: <finding> — <url>`, or the `competitors_note`. In Cursor, send that same frontier once through AskQuestion: suggested answer first, marked "(Recommended)", then each alternative, then Defer. At most 5 questions in the round; the rest of the frontier waits until the next round. Then wait.
4. Look up facts yourself (transcript, competitors, code, config). The user answers decisions only.
5. Record each answer in `decisions.json` as a new `D#`, in the user's own words. Set that question to `answered` or `deferred`. If an answer opens a new gap, send only that branch back to the clarifier, then recompute the frontier.
6. Start the next round only after this round's answers are in `decisions.json`.

**Done when:** the frontier is empty — no question in `questions.json` is `open` — or 3 rounds have run and every question still `open` is marked `deferred`. Deferred questions keep their tickets at `needs_clarification`. Drafting (step 5) starts only after this. In fixture mode, skip the interview and mark every question `deferred`.

## 5. Draft

Dispatch the **drafter** with the run folder path.

**Done when:** `prd.md` and `tickets.json` exist, and every ticket and PRD line cites a source.

## 6. Gate

Dispatch the **gatekeeper** with file paths only (`transcript.md`, `decisions.json`, `tickets.json`). Give it no other context.

- If tickets are `needs_clarification` with `new_questions`, go back to step 4 for those questions, then redraft (step 5) and re-gate. Maximum 2 draft → gate retries.
- `rejected` tickets are dropped. Tell the user which ones and why.

**Done when:** `gate_report.json` exists, and no ticket's `needs_clarification` can be resolved by another round, because its questions are deferred or the retry limit has been reached.

## 7. Impact analysis

Use the GitHub MCP (read-only: search code, read file, list commits) on the repos in `vague2tickets.config.json`. For each `ready` ticket, find the current implementation and pass the matching paths and excerpts to the **impact-analyst**, along with `tickets.json` and `gate_report.json`. If GitHub MCP is not connected, say so and fall back to a local clone. If it returns `new_questions`, run one more step-4 round for them.

**Done when:** `impact.json` has a comparison (`missing`, `partial`, or `already_done`) and affected areas (or an explicit "none found") for every `ready` ticket, plus anything it could not scan.

## 8. Plan

Dispatch the **planner** with the run folder path. It assigns each task from the `hub.york.ie` team-directory MCP, or `fixtures/team_directory.json` if that MCP isn't connected — say which one was used.

**Done when:** `plan.json` has an epic → story → task breakdown for every `ready` ticket, with Dev and QA tasks on every story, a UI/UX task wherever a screen or flow changes, every task assigned to a person with a reason, and a proposed timeline.

## 9. Evaluate

Dispatch the **evaluator** with the run folder path (and `answer_key.json` in fixture mode).

**Done when:** `scorecard.md` exists with a verdict. If it is **FAIL**, fix the failing step and re-run it from there. Never request approval on a FAIL.

## 10. PM approval

Show the PM one summary:

- the PRD link (`prd.md`) and a one-paragraph overview
- the decisions made (`D#` with answers) and any deferred questions
- the ticket statuses and the rejected ones with reasons
- the impact areas, by risk
- the **exact preview** of what will be created: the Confluence page title and space, plus each epic, story, and task with title, discipline, estimate, dates, priority, assignee, and the Jira issue type it will become. Say whether assignees came from `hub.york.ie` or the simulated roster. Group tasks by assignee and show each person's queue in `assignee_sequence` order, numbered, so it's visible which task they do first when one person holds several. For each item, show `provenance` (generated by vague2tickets, run id, local id) and its `references`, or "none" when the array is empty.
- the scorecard verdict

Ask the PM to reply `APPROVE <run-id>`, ask for changes, or approve a subset (e.g. `APPROVE <run-id> E1 only`). Change requests go back to the step they affect. Record the approver, the time, and the approved scope in `audit.log`.

**Done when:** the PM has typed the exact approval phrase, or declined. Anything else is not approval.

## 11. Publish

### 11a. PRD to Confluence

Publish the PRD in the same step as the Jira issues. Do this before creating the issues, so every issue can link to the page.

1. Update the progress page from `progress.json`. If that file is missing, search the configured space for `PRD: <feature> (<run-id>)` and update that page. Create one only when neither exists. If `confluence.space_key` is empty, ask once, write it to the config, then continue. If the site has no Confluence (the API returns 404), stop, say so, and do not create the Jira issues yet.
2. Set the page status to `Approved` and add a version comment `v2t:<run-id>`. Do not create a second page.
3. Page body, in this order, citations kept as visible text (`[quote: "…"]`, `[D3]`):
   - **Overview, goals, users and roles** from `prd.md`.
   - **Requirements and acceptance criteria**, one section per ticket, with its ticket ID.
   - **Clarifications**: every question, why it matters, the suggested answer, why that answer (client quote, then each competitor as `<product>: <finding> — <url>`), the alternatives, and the outcome (`D#` plus the answer, or `deferred`). Mark unaccepted suggestions as proposals.
   - **Decisions log** from `decisions.json`.
   - **Out of scope** and **open questions**.
   - **Work items**: each epic, story, and task with its local ID. After the Jira issues exist, edit the page once and replace each local ID with the issue key and browse URL.
   - **Sources**: each confirmed call, Slack thread, email thread, and Confluence page, with its link.
4. Create or update the page under the configured parent, or at the space root. Record the title, space key, page id, and URL in `published.json` before creating Jira issues.

### 11b. Tickets to Jira

1. Search the configured project with JQL for the label `v2t:<run-id>` and for the existing tickets found in step 2. Never create an item that already exists; link to it instead.
2. Create only the approved scope, top-down: Epics first, then Stories linked to their epic (`Epic Link` / parent), then sub-tasks linked to their story. Use the issue type names from the config.
   - Set the Jira assignee from `plan.json`'s `assignee.email` when that email matches an existing Jira account on this site; otherwise leave the Jira assignee unset and name the intended assignee in the description instead. Either way, the Source section (below) always names them.
   - Set the Jira Priority field from `plan.json`'s `priority` when the project has a priority of that name; otherwise leave the default Priority and state the intended one in the description instead.
3. Give every issue (epic, story, and sub-task) a **Provenance** section, then a **References** section, then a **Source** section in its description, plus acceptance criteria and the Confluence PRD URL from step 11a. A sub-task lists the sources of its parent story. A `qa` sub-task's description also lists its `test_cases` and `edge_cases` from `plan.json`. A `ui_ux` sub-task's description also lists its `design_references` from `plan.json`, each labeled "existing pattern, not a requirement," or the none-found note when there are none.
   - **Provenance:** `Generated by vague2tickets (AI)`, the `run_id`, and the local ID (`E1`, `E1-S1`, `E1-S1-T2`). On a story or sub-task, also the parent ticket id (`T1`).
   - **References:** every entry in that item's `references`. A Jira reference is `relation`, key, title, and URL. A code reference is `relation`, repo, and path, labeled "existing code, not a requirement". When `references` is empty, write `none`. Also add a Jira issue link to each referenced key (`Relates`). If the link call fails, keep the description line and say which key did not link.
   The Source section has:
   - **Origin:** every confirmed source in `transcript.md` (call, Slack thread, email thread, Confluence page), each with its title, date, and link.
   - **Client quotes:** each quote the item cites, verbatim, with its speaker or author and timestamp.
   - **Decisions:** each `D#` the item cites: the question, the answer, who answered, when, and whether it accepted the suggestion.
   - **Why we recommended it:** for each accepted suggestion, its `reason` and its competitor findings (`<product>: <finding> — <url>`), marked "context, not a requirement".
   - **Assigned to:** the task's `assignee.name` and the `assignee.reason`, noting whether it came from `hub.york.ie` or the simulated `fixtures/team_directory.json`.
   - **Priority and queue:** the task's `priority`, and its `assignee_sequence` as "Do this #N in <name>'s queue for this run," naming the task before it (if `assignee_sequence` > 1) and the one after (if it exists), by title and key once that key is created.
   - **Run:** `<run-id>` and the local ID (`E1-S1`, `E1-S1-T2`).

   Add the labels `ai-generated` and `v2t:<run-id>`, and a label for the discipline (`dev`, `qa`, `ui_ux`, `devops`). `ai-generated` is how a reader filters every ticket this pipeline created, including a later run that recreates the same capability.

4. If a creation fails (bad issue type, permissions, etc.), stop, report exactly what was and wasn't created, and wait for the user — do not retry blindly.

Record every created or updated item (Confluence page and every Jira issue) in `published.json`.

**Done when:** `published.json` lists the Confluence page URL and every approved Jira issue with its key and URL, and you have shown the user the full list of links.
