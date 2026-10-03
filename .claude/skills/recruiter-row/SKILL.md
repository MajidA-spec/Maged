---
name: recruiter-row
description: Decide whether a Gmail email counts as a recruiter email, and turn recruiter emails into rows for the user's "Recruiters" Google Sheet (Date, Name, Company, Role, Location, Salary, Next step, Email link) without ever deleting or changing existing rows. Use whenever screening Gmail for recruiters, applying the "Recruiters" Gmail label, backfilling the Recruiters sheet, or running the weekly recruiter routine.
---

# recruiter-row

The written procedure for turning one recruiter email into one sheet row.
It encodes the user's own corrections, so follow it exactly. When the user
asks to change a behaviour ("update the recruiter-row skill so it never does
X again"), edit this file **and** `routine/saturday-routine-prompt.md` so the
weekly routine stays in step.

## 1. Does the email count as a recruiter email?

Count it **only if all three** are true:

1. **It comes from a specific, named person.** A human who writes in the first
   person and signs with their own name. Personal mailboxes count:
   `first.last@`, `flast@`, a first name (`otis@`, `ravena@`) or initials
   (`cc@`, `kk@`) **when the body is signed by a named person**.
2. **That person is a recruiter**: agency consultant, headhunter, or in-house
   talent acquisition.
3. **It is about work for the user**, for example:
   - one role or several roles
   - a weekly bulletin of roles sent from the recruiter's own address
   - a referral request for a role
   - a CV request
   - an interview arrangement
   - a catch-up or follow-up about the user's job search

**Never count** (the user's rule: "LinkedIn, Indeed or any email not coming from
a specific person is not to be counted as a recruiter"):

- **LinkedIn**: any `linkedin.com` sender, including InMail notifications and
  job alerts, even when the notification quotes a named recruiter.
- **Indeed**: any `indeed.com` sender.
- **Other job boards and platforms**, which send automated mail: CV-Library,
  Totaljobs, Reed, Glassdoor, Monster, Jobsite, CWJobs, Gradintelligence, and
  applicant-tracking systems (`hr-manager.net`, `talentech`, "Candidate Centre"
  mails).
- **Generic or role mailboxes**, even at a real agency: `info@`,
  `information@`, `admin@`, `contact@`, `hello@`, `survey@`, `jobs@`,
  `careers@`, `recruitment@`, `marketing@`, `cpe-marketing@`, `gold@`, `CPE@`,
  `datacontroller@`, `compliance@`, `member@`, `noreply@` / `no-reply@`,
  `notifications@`.
- **Mail from a named person that is not about work for the user**: GDPR or
  privacy notices, salary surveys, company news, newsletters with no role in
  them.
- **Scam or unsolicited "work from home / personal assistant" offers** from
  free webmail addresses with no firm behind them.
- **Anything the user sent themselves** (`in:sent`).

**Decisions already applied to this inbox.** Keep applying them consistently:

| Case | Decision | Role column |
|---|---|---|
| Weekly role bulletin from a named recruiter's own address ("Hi all, below is this week's bulletin…") | Count | `Multiple roles – weekly bulletin` |
| Monthly vacancy round-up from a named consultant's address (e.g. `firstname.lastname@email.<agency>`) | Count | `Multiple roles – monthly round-up` |
| Role outside the user's field (e.g. software developer) from a named recruiter | Count. The sender rule is what decides | As stated |
| Referral request ("do you know anyone…") from a named recruiter | Count | The role they are filling |
| Recruiter asking whether the user is *hiring* | Count | `N/A – asked if you are hiring` |

If you are unsure, **do not count it**. List it under "uncertain" in the run
report so the user can decide.

## 2. How many rows: the append-only algorithm

Rows are only ever **added**. Earlier rows are never edited.

For every counted thread, compare the thread's message links with the sheet's
`Email link` column. Gmail links look like
`…#all/thread-f:<thread>|msg-f:<message>`.

1. **The thread is not in the sheet yet** (no row contains its
   `thread-f:<thread>`): append **one** row.
   - Date: the **first** recruiter message in the thread.
   - Name, Company, Role, Location, Salary: the substance of the thread.
   - Next step: the thread's **latest** state.
   - Email link: the link of the **newest** recruiter message in the thread.
2. **The thread is already in the sheet**: find the newest message in the
   thread whose link appears in the sheet.
   - If recruiters sent messages **after** that one, append **one** follow-up
     row. Its Date and Email link come from the newest new message, and Next
     step starts with `Follow-up: `.
   - Otherwise skip the thread.

This makes every run safe to repeat: running the same week twice adds nothing
the second time. Never create rows for the user's own replies.

## 3. Filling the eight columns

| Col | Header | How to fill it |
|---|---|---|
| A | Date | `YYYY-MM-DD` in **Asia/Riyadh** time. |
| B | Name | The recruiter's full name as signed. If there is no signature, build it from the address (`jane.doe@` → `Jane Doe`). |
| C | Company | The **recruitment firm's** trading name as written in the signature (e.g. `frlondon.co.uk` signs as "Fawkes & Reece"). The hiring client goes in Role, if named. |
| D | Role | Job title or titles plus a short descriptor, e.g. `Senior QS – data centre, 16+ month contract`. Put the client in brackets if named. For several roles, list up to 3 then `+N more`. |
| E | Location | As stated, including Remote or Hybrid if mentioned. Otherwise `Not stated`. |
| F | Salary | Exactly as stated, with currency and unit: `£75k + package`, `£45–50/hr`, `£400/day`, `Negotiable`. Otherwise `Not stated`. If the salary is only in an attachment you could not read, write `See attachment`. |
| G | Next step | At most 12 words, imperative, taken from the email's ask: `Send CV + availability`, `Reply to book a call`, `Refer a colleague (referral fee)`. If the user already replied, say so: `You replied 12 Mar – awaiting recruiter`. |
| H | Email link | The Gmail `viewUrl` of the message, exactly as the Gmail connector returns it. |

**Never invent.** If a value is inferred rather than stated, add ` (inferred)`.
Ignore signatures, disclaimers and tracking links when reading a body.

## 4. Writing to the Google Sheet: append only

The user's instruction is: "Don't delete any line items, collect them all."

- **Never** delete, clear, overwrite, sort, filter in place, re-order, or insert
  rows above existing data. Never edit an existing cell.
- Read the tab (`get_values` on `'<Tab>'!A:H`) **immediately before** each
  write. The first free row is the last filled row + 1.
- Write with `update_values` to the exact range `A<first>:H<last>`, up to about
  25 rows per call.
- Start every text cell in columns B–H with an apostrophe (`'`), so Sheets does
  not turn `1/2`, `£45-50` or `00123` into dates or numbers. Write the Date
  (column A) without an apostrophe so it becomes a real date.
- If row 1 is empty, write the eight headers first. If the headers differ from
  the expected eight, map by header name and do not rename anything.
- After each write, read the tab again and confirm two things:
  - the earlier rows are still there and unchanged in count
  - the new rows are present
  
  If anything looks wrong, **stop and report it**. Never try to repair the sheet
  by deleting or clearing.

## 5. Gmail housekeeping

- Apply the Gmail label **`Recruiters`** to every counted thread. Look up its
  ID with `list_labels`, and create the label if it is missing.
- Never remove labels, archive, trash, mark read or unread, reply, or send.

## 6. Report at the end of every run

- Rows added, as `Name — Company — Role`, plus how many were follow-up rows.
- Threads labelled.
- Uncertain emails not added, each with a one-line reason.
- Anything assumed or inferred.
