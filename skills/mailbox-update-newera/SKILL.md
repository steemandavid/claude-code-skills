---
name: mailbox-update-newera
description: Refresh the New Era to-do list and project status from the synced Outlook mailbox. Use when David asks to go through his mailbox, update the to-do list, check what he is waiting on a reply for, or where his projects stand.
---

# mailbox-update-newera

Produces two dated files from the synced `david.steeman@neweratech.com` OST:

- `reports/todo-YYYY-MM-DD-HHMM.md` — today's tasks, chase list, tasks per project
- `reports/status-YYYY-MM-DD-HHMM.md` — per-project progress, issues, blockers

Project directory: `/home/john/claudecode/projects/NewEraOutlook-interface`.
Run everything with `.venv/bin/python` from there.

## Hard rules (context budget — see CLAUDE.md)

The backend has ~45K usable context. Therefore:

- **`scan.py` is the only bulk mail read of the run.** Never call `mailbox.py
  folders/list/search` without a narrow filter, never call `recent.py` here.
- Pull full bodies for **at most ~10** messages, via
  `.venv/bin/python digest.py "<folder-fragment>" <idx>`, and only where the
  digest snippet is genuinely not enough to decide what the task is.
- Do not re-read `scan.py`, `digest.py`, `mailbox.py` or `README.md` after a
  compaction — their behaviour is described here.

## Steps

1. **Find the baseline.** Newest `reports/todo-*.md`, else newest root
   `todo-*.md`. Take the `scanned mail since` / generation timestamp from its
   header → that is `--since`. If no prior file exists, use 7 days back.
2. **Read the inputs:** `projects.md` (taxonomy) and `chase-ignore.md`.
3. **Scan:**
   ```
   .venv/bin/python scan.py --since <ISO> --sent-days 30
   ```
   Sections returned: `HEADER` (OST freshness), `NEW`, `SENT-UNANSWERED`,
   `DRAFTS`, `COUNTS`. Add `--max-new N` if the digest is truncated and the
   remainder matters.
   If `HEADER` says `*** STALE ***`, carry that warning into both output file
   headers and mention it in the terminal summary — but continue, and tell
   David how to force a sync (see *Forcing a mailbox sync* below).
4. **Deep-read selectively** (≤10 messages) with `digest.py`, using the
   `[folder#idx]` handles printed by the scan.
5. **Read the previous todo and the previous status file** so you can diff.
6. **Classify every item to a project** using `projects.md`: the mail's Outlook
   folder (`Inbox/Projects/<name>`) is the strongest signal, then `aliases`,
   then `people`. Anything unmatched goes to `## Unassigned`, and you
   **auto-append** a proposed entry to `projects.md`:
   ```
   ## <proposed name>
   aliases: <the terms that would have matched>
   people: <senders seen>
   folders: <Outlook folder if any>
   status: proposed        # auto-added YYYY-MM-DD, review me
   ```
   Never rename or delete existing entries; only append.
7. **Write the two files** (templates below) into `reports/`, then **publish
   them to the fileshare** so they are readable from the work PC at
   `\\192.168.1.165\fileshare\outlook\reports\`:
   ```bash
   cp reports/todo-<stamp>.md reports/status-<stamp>.md /storage/fileshare/outlook/reports/
   cp reports/todo-<stamp>.md   /storage/fileshare/outlook/reports/todo-latest.md
   cp reports/status-<stamp>.md /storage/fileshare/outlook/reports/status-latest.md
   chmod 664 /storage/fileshare/outlook/reports/*.md
   ```
   The dated copies build up a history; `todo-latest.md` / `status-latest.md`
   always point at the newest run, so David can keep one file open on the
   laptop. Mention the UNC path in the terminal summary.
8. **Report in the terminal in ≤10 lines:** counts, the Today list, the chase
   list, proposed project additions, staleness warning if any. No essay.

## Judgement rules

- **Today section** is deadline-driven: due today or overdue, chases that have
  crossed the threshold, and direct asks to David still unanswered. No
  calendar data exists in the OST — never invent meeting times; if an item
  depends on a meeting, say "check your calendar".
- **Chase threshold:** deadline stated in the mail wins; otherwise 3 working
  days (weekends excluded, public holidays ignored). `scan.py` applies this.
- Split the chase list into **"waiting on them"** and **"I promised X"** — the
  scan flags both (a mail where David committed to send something is an
  unanswered thread too, but it is *his* action).
- Mark every item `[new]`, `[updated]`, `[done]` versus the previous file, and
  strike through finished items. Carry unchanged open items forward verbatim
  with `*(carry-over)*` so nothing silently disappears.
- Language: **English**, but quote Dutch/French mail fragments verbatim.
- Keep task lines one-liners with the evidence date, e.g. `(Bart 9/9)`.

## Output templates

`reports/todo-YYYY-MM-DD-HHMM.md`:

```markdown
# To-do — david.steeman@neweratech.com
Generated <date time> · OST synced <time> (<n> min ago)
Supersedes <previous file> · scanned mail since <since>

## 🔴 Today
- [ ] ...

## ⏳ Waiting on a reply
- [ ] **<subject>** → <recipient>, sent <date>, **<n> working days open**,
      asked "<verbatim quote>" — deadline: <stated|none> · <thread-matched|subject-only>

## ✋ I promised / owe someone
- [ ] ...

## Drafts still unsent
- [ ] ...

## Per project
### <Project>
- [ ] ...
### Unassigned
- [ ] ...  ← appended to projects.md as `status: proposed`

## Done since last run
- ~~...~~
```

`reports/status-YYYY-MM-DD-HHMM.md`:

```markdown
# Project status — david.steeman@neweratech.com
Generated <date time> · previous: <previous status file>

## <Project>
**State:** one line.
**Changed since <date>:** ...
**Issues:** ...
**Blocked on:** <who / what / since when>
**Next action:** <what> — <owner> — <by when>

## Completed since last run
## Unassigned threads
```

## Forcing a mailbox sync

The OST is pushed by a scheduled task **on the Windows work laptop**; it
cannot be triggered from john-ai (no WinRM, and the laptop must be awake on
the home LAN). On the laptop, in an **elevated** PowerShell / cmd:

```powershell
schtasks /Run /TN "Sync mailbox to john-ai"          # fire the task now
schtasks /Query /TN "Sync mailbox to john-ai" /V /FO LIST   # last run + result
& C:\Scripts\sync-mailbox.ps1                        # or run the script directly
```

Closing Outlook first gives a clean copy (a VSS snapshot taken mid-flush can
produce an unreadable OST). Verify from john-ai:

```bash
ls -l --time-style=+%F\ %H:%M /storage/fileshare/outlook/*.ost
```

A ~3.6 GB copy takes a few minutes over SMB; the swap is atomic, so the old
copy stays readable until the new one lands.

## Files

| File | Role |
|---|---|
| `scan.py` | the scan; `--since`, `--sent-days`, `--grace-days`, `--max-new`, `--snippet` |
| `projects.md` | project taxonomy (aliases / people / folders); auto-appended `status: proposed` entries |
| `chase-ignore.md` | threads never to flag as unanswered |
| `digest.py` | full body of one message: `digest.py "<folder>" <idx>` |
| `skill-spec-mailbox-update-newera.md` | design rationale |
| `reports/` | generated files, mirrored to `/storage/fileshare/outlook/reports/` (share `fileshare`, writable, user `john`) |
