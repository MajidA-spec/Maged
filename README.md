# Recruiter Tracker

An AI agent that collects every email from a real, named recruiter into the
user's **Recruiters** Google Sheet. One row per recruiter contact, with these
columns:

`Date | Name | Company | Role | Location | Salary | Next step | Email link`

It follows the *Gmail Recruiter Tracker Guide* (2 Oct 2026), with three rules
from the user:

1. **LinkedIn, Indeed, or any email not from a specific named person is not a
   recruiter.**
2. **The routine runs every Saturday at 5:25 PM (Asia/Riyadh), changed from 3:20 PM at the user's request.**
3. **Never delete any line items.** The sheet is append-only, and every
   recruiter contact is collected.

## How it maps to the guide

| Guide step | What it is here | Status |
|---|---|---|
| 1 · Workflow saved as a skill | [`.claude/skills/recruiter-row/SKILL.md`](.claude/skills/recruiter-row/SKILL.md): counting rules, row format, append-only writing | Done |
| 2 · Labelling recruiter mail | The user chose to have Claude apply the Gmail label **`Recruiters`** itself (no n8n). The label exists and covers the full history: 366 threads from 152 named recruiters at 40 firms, 2017–2026 | Done |
| 3 · Backfill of old emails | [`.claude/skills/recruiter-tracker-setup/SKILL.md`](.claude/skills/recruiter-tracker-setup/SKILL.md): reads `label:Recruiters` into the sheet, starting with two rows | Done 2026-10-03: 365 rows (2 test rows + 363 backfilled), 1 thread skipped as a GDPR notice |
| 4 · Weekly Routine | [`routine/saturday-routine-prompt.md`](routine/saturday-routine-prompt.md), cron `CRON_TZ=Asia/Riyadh 25 17 * * 6`. It fires into the setup session, which holds the Gmail and Google Sheets connectors | Created 2026-10-03 |

## Finishing the setup (one-time)

1. Connect **Google Sheets** at <https://claude.ai/customize/connectors>, with
   the same Google account that owns the sheet.
2. Start a **new** Claude Code session on this repository. Connectors only load
   when a session starts.
3. Send: *"Finish the recruiter tracker setup. Sheet: \<your sheet link\>"*.

The setup skill does the rest:

- checks the sheet
- writes two rows for you to check
- after you say continue, backfills the rest

**The Saturday Routine** (17:25, Asia/Riyadh) fires back into the setup
session, because that session holds the Gmail and Google Sheets connectors.
On this account, Claude cannot attach connectors to a Routine that starts a
fresh session each run. **Do not archive the setup session**, or the Routine
stops working.

If you ever want a fresh session per run instead, create the Routine yourself
in the claude.ai Routines UI with these settings:

- **Schedule:** Saturdays at 17:25, time zone Asia/Riyadh.
- **Session:** a fresh session on each run.
- **Connectors:** Gmail and Google Sheets.
- **Prompt:** the text block in `routine/saturday-routine-prompt.md`, with
  `<SHEET_URL>` replaced by your sheet link.

## Screening decisions you may want to change

These are recorded in the `recruiter-row` skill. Ask Claude to update the skill
to change any of them.

- **Weekly or monthly bulletins** sent from a named recruiter's own address
  **count**, because the sender is a specific person.
- **Roles outside quantity surveying** (e.g. .NET developer) from a named
  recruiter **count**.
- **GDPR notices, salary surveys and company news** **do not count**, even
  from a named person.
- **Initials or first-name mailboxes** (`cc@`, `kk@`, `otis@`) **count** when
  the email is signed by a named person.

## Known limits

- **Second inbox.** Since 2017–2020 you have asked recruiters to use a
  different Gmail address. That inbox is **not** connected, so recruiters
  writing there, possibly including the ones you care about most, are not
  seen. Two fixes:
  - forward that inbox to this one (Gmail → Settings → Forwarding), or
  - connect that account instead
- **Attachments.** If a salary appears only in a PDF the connector cannot
  read, the row says `See attachment`.
- **Session-bound Routine.** The first run (Saturday 2026-10-03, 17:25) had
  Gmail and Sheets access and found no new recruiter emails. It only keeps
  working while the setup session exists, so do not archive that session.

## Privacy

This repository is **public**. The sheet link, the Gmail address and all
recruiter names are kept out of it on purpose. The sheet link lives only in
the Routine's stored prompt and in your own copy of it. Keep the sheet itself private: it holds other
people's names and contact details.
