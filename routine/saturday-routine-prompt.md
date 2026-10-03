# Saturday routine prompt

This is the exact text the weekly Routine sends to a **fresh** Claude Code
session every **Saturday at 15:20 Asia/Riyadh**. A fresh session may not have
this repository's skills loaded, so the prompt carries the full rules.

`<SHEET_URL>` is a placeholder. The setup session swaps in the real Google
Sheet link when it creates the Routine. The link is kept out of this public
repository on purpose.

Keep this file in step with `.claude/skills/recruiter-row/SKILL.md`.

---

```text
Weekly recruiter-tracker run. Use ONLY the Gmail and Google Sheets connectors.
Absolute rules: never delete, clear, overwrite, sort or re-order anything in
the sheet; never edit an existing cell; in Gmail never archive, trash, mark
read/unread, reply or send. The only allowed changes are (a) appending new
rows at the bottom of the sheet and (b) adding the Gmail label "Recruiters".

Sheet: <SHEET_URL>
Columns A–H: Date | Name | Company | Role | Location | Salary | Next step | Email link

STEP 1 – Label. Call list_labels and get the id of the label "Recruiters"
(create it if missing).

STEP 2 – Find candidates. search_threads with query
  newer_than:8d -in:sent -in:drafts -in:chats
and page through ALL results (pageSize 50, follow nextPageToken). Also search
  label:<Recruiters label id> newer_than:8d
for follow-ups in threads that were already labelled. Previews only show the
oldest messages, so call get_thread (messageFormat PLAIN_TEXT) on every thread
you might count.

STEP 3 – Decide if each thread counts. Count it ONLY if all three are true:
 1. It is sent by a specific, named person who writes in the first person and
    signs with their own name. Personal mailboxes count: first.last@, flast@,
    a first name (otis@) or initials (cc@, kk@) when the body is signed by a
    named person.
 2. That person is a recruiter: agency consultant, headhunter or in-house
    talent acquisition.
 3. It is about work for the user: a role or several roles, a weekly bulletin
    of roles from the recruiter's own address, a referral request, a CV
    request, an interview arrangement, or a catch-up about their job search.
NEVER count:
 - LinkedIn or Indeed in any form, even if it quotes a named recruiter.
 - Other job boards and platforms: CV-Library, Totaljobs, Reed, Glassdoor,
   Monster, Jobsite, CWJobs, Gradintelligence, applicant-tracking systems.
 - Generic mailboxes, even at real agencies: info@, information@, admin@,
   contact@, hello@, survey@, jobs@, careers@, recruitment@, marketing@,
   gold@, CPE@, datacontroller@, compliance@, member@, noreply@, no-reply@,
   notifications@.
 - Mail from a named person that is not about work for the user: GDPR or
   privacy notices, salary surveys, company news, newsletters without roles.
 - Scam "work from home / personal assistant" offers from free webmail.
 - Anything the user sent.
Already-agreed cases:
 - Weekly bulletins from a named recruiter count (Role "Multiple roles –
   weekly bulletin").
 - Roles outside the user's field from a named recruiter count.
 - Referral requests count.
If unsure, do NOT count the thread; list it as "uncertain" in your report.
Add the "Recruiters" label to every thread that counts.

STEP 4 – Read the sheet. get_spreadsheet with fields ["sheets.properties"]
gives the first tab's title. Then get_values on '<tab>'!A:H. Remember the
number of filled rows and every value in column H (Email link). If row 1 is
empty, write the 8 headers first. If the headers differ, map by header name
and do not rename anything.

STEP 5 – Build rows (append-only). Gmail links look like
…#all/thread-f:<thread>|msg-f:<message>.
 a) Thread not in the sheet (no column-H value contains its thread-f id):
    add ONE row.
    - Date: the FIRST recruiter message in the thread.
    - Next step: the latest state of the thread.
    - Email link: the viewUrl of the NEWEST recruiter message in the thread.
 b) Thread already in the sheet: find the newest message in the thread whose
    viewUrl is in column H. If recruiters sent messages after it, add ONE
    follow-up row: Date and Email link from the newest new message; Next step
    starts with "Follow-up: ". Otherwise skip the thread.
Never make a row from the user's own replies.
Columns:
 - Date: YYYY-MM-DD in Asia/Riyadh time.
 - Name: recruiter's full name as signed.
 - Company: the recruitment firm's trading name.
 - Role: job title or titles plus a short descriptor, with the client in
   brackets if named. For several roles, up to 3 then "+N more".
 - Location: as stated, otherwise "Not stated".
 - Salary: exactly as stated with currency and unit, otherwise "Not stated".
   Write "See attachment" if it is only in an unreadable attachment.
 - Next step: at most 12 words, imperative, from the email's ask. Note if the
   user already replied.
 - Email link: the message's Gmail viewUrl, exactly as returned.
Never invent anything. Add " (inferred)" to any value you inferred.

STEP 6 – Append. Re-read '<tab>'!A:H just before writing. Then call
update_values on the exact range A<last+1>:H<last+n>, oldest row first. Start
every text cell in columns B–H with an apostrophe ('). Write the Date without
an apostrophe.

STEP 7 – Verify. Re-read the tab. Check that the previous rows are all still
there (the count did not drop) and the new rows are present. If anything is
wrong, STOP and report it; never "fix" anything by deleting or clearing.

STEP 8 – Report briefly:
 - rows added (Name — Company — Role), and how many were follow-ups
 - threads labelled
 - uncertain emails skipped, with reasons
 - anything inferred
If nothing new arrived this week, say "No new recruiter emails this week".
```
