---
name: recruiter-tracker-setup
description: One-time finishing steps for the Gmail → Google Sheets recruiter tracker, for a session that has BOTH the Gmail and Google Sheets connectors. It checks the sheet, backfills every thread already labelled "Recruiters" into the sheet (starting with two rows for the user to check), and then creates the weekly Routine for Saturdays at 15:20 Asia/Riyadh. Use when the user says "finish / set up the recruiter tracker", "run the recruiter backfill" or "create the recruiter routine".
---

# recruiter-tracker-setup

Earlier work already in place (session of 2026-10-03):

- **Gmail label `Recruiters`** exists. It was applied to every recruiter
  thread in the inbox's full history that passes the `recruiter-row` rules:
  366 threads (449 messages), 2017 to 2026. LinkedIn, Indeed, job boards and
  generic mailboxes were excluded.
- **The sheet is still empty.** That session had no Google Sheets connector.

Load and follow the **`recruiter-row`** skill for every counting, row-building
and sheet-writing decision.

## Step 0 – Preconditions

1. **Confirm the Google Sheets tools are available** (e.g. ToolSearch for
   "google sheets get_values"). If they are missing, stop and tell the user to
   do two things, then start a new session:
   - connect Google Sheets at https://claude.ai/customize/connectors
   - start a new session (connectors load at session start)
2. **Get the sheet link.** It is deliberately **not** stored in this public
   repository. Take it from the user's message, or ask for it.
3. **Read `.claude/skills/recruiter-row/SKILL.md`** and, if available, the
   `google-workspace` skill's `references/sheets.md`.

## Step 1 – Inspect the sheet (read only)

1. Call `get_spreadsheet` with `fields: ["sheets.properties"]` to get the tab
   names.
2. Call `get_values` on `'<first tab>'!A:H`.
3. Check the headers. If row 1 is empty, write the eight headers:
   `Date | Name | Company | Role | Location | Salary | Next step | Email link`.
4. Note how many rows already exist. They must never be touched.

## Step 2 – Backfill: start with two

1. Find the `Recruiters` label ID with `list_labels`. You need the ID for
   `label_thread`.
2. List every thread with `search_threads` using `label:Recruiters`, paging
   through all pages. There should be about 366. Search by the label **name**:
   in testing, `label:<id>` (e.g. `label:Label_1`) returned nothing.
3. Sort the threads **oldest first**.
4. Build rows for the **two oldest threads only**, using the `recruiter-row`
   algorithm (`get_thread`, `messageFormat: PLAIN_TEXT`), and append them.
5. **Show the user those two rows and wait for "continue".** This is the
   guide's "start with two" rule.

## Step 3 – Backfill: the rest

- Continue oldest to newest. Write about 25 rows per `update_values` call, and
  re-read the sheet before and after every write.
- If a thread fails the counting rules on a full read, skip it, leave its
  label alone, and list it in the report.
- Re-running is safe. The algorithm skips threads already in the sheet, so an
  interrupted backfill can simply be started again.
- **Cost:** 366 threads is a lot of reading.
  - Ignore signatures and disclaimers.
  - Parallel subagents are allowed **only if the user agrees** (the guide's
    "dynamic workflow" step). Give each subagent a batch of thread IDs and have
    it return rows as data.
  - The main session stays the **only writer** to the sheet.

## Step 4 – Create the weekly Routine

Call `create_trigger` (claude-code-remote) with:

| Field | Value |
|---|---|
| `name` | `Recruiter tracker — Saturdays 15:20 Riyadh` |
| `cron_expression` | `CRON_TZ=Asia/Riyadh 20 15 * * 6` |
| `create_new_session_on_fire` | `true` |
| `connectors` | `["Gmail", "Google Sheets"]` |
| `initiation` | `human_request` |
| `prompt` | The text block in `routine/saturday-routine-prompt.md`, with `<SHEET_URL>` replaced by the real link |

- If the result warns that connectors were not stored, say so plainly and pass
  on the remedy it names.
- Offer one test run with `fire_trigger`. **Ask first**, because it writes to
  the sheet.
- Tell the user they can watch the first run in Claude Code on the web.

## Step 5 – Report

- Rows written, and any threads skipped with the reason.
- The Routine's ID and next run time.
- Reminders:
  - recruiters who write to the user's *other* Gmail address are not seen
    unless that mail is forwarded to this inbox
  - keep the sheet private
