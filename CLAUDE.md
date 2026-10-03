# CLAUDE.md

This repository holds the **Recruiter Tracker** agent. It contains no
application code, only skills and a Routine prompt. See `README.md`.

- `.claude/skills/recruiter-row/` decides what counts as a recruiter email and
  how a row is built and appended. It is the single source of truth for those
  rules.
- `.claude/skills/recruiter-tracker-setup/` holds the one-time steps: the
  backfill into the Google Sheet, then creating the Saturday 15:20 Riyadh
  Routine.
- `routine/saturday-routine-prompt.md` is the standalone prompt each weekly
  run receives. Keep it in step with `recruiter-row`.

Rules that always apply:

- **The sheet is append-only.** Never delete, clear, overwrite, sort or
  re-order rows.
- **LinkedIn, Indeed and anything not from a specific named person are never
  counted** as recruiter emails.
- **This repository is public.** Never commit the sheet link, the user's email
  address, or recruiter names or emails.
