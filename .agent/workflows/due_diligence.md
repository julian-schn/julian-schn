---
description: Run full due diligence across the entire repo (typos, outdated info, structural issues, info leaks, link rot, etc.)
---

# /due-diligence

Performs a thorough audit of every file in the repository. Run this before any public release or major update.

## Step 1 — Inventory the repo

List all tracked, non-binary files so that every subsequent step operates on the same scope.

```bash
git -C . ls-files | grep -v -E '\.(png|jpg|jpeg|gif|pdf|ico|svg|woff|ttf|eot|zip|tar|gz)$'
```

Use the output as the file list for all steps below.

## Step 2 — Typos & grammar

Read every Markdown and plain-text file. For each file, check:

- Spelling mistakes (common English misspellings, accidental double words)
- Grammar issues (subject-verb agreement, punctuation, capitalisation)
- Informal / sloppy language in public-facing files (`README.md`, `projects.md`)

Flag every issue as a line-level finding: `[TYPO/GRAMMAR] <file>:<line> — "<original text>" → "<suggested fix>"`.

Known issues spotted in the current repo to verify/fix:
- `README.md` line 16: "afformentioned" → "aforementioned"
- `README.md` line 8: "inbound internn" → "inbound intern"

## Step 3 — Outdated or stale information

For every file, check:

- Dates, version numbers, and "current" status fields (e.g. `currently:`, `soon:`) — do they still reflect reality?
- Project statuses ("Ongoing", year ranges) — are they consistent between `README.md` and `projects.md`?
- URLs/links — do all hyperlinks resolve (use `curl -sI` head requests or the browser tool)?
- Any reference to a role, company, or university semester that may have passed

Flag as: `[STALE] <file>:<line> — <reason>`

## Step 4 — Accidental information disclosure

Read every file carefully for:

- Personal data: full addresses, phone numbers, private email addresses, national ID / tax numbers
- Credentials, API keys, tokens, secrets (check `.env*`, config files, comments in code)
- Private repository names, internal server names, or internal project codenames that shouldn't be public
- Sensitive file paths or internal infrastructure details

Also run:
```bash
git -C . log --all --oneline --diff-filter=D -- "*.env" "*.key" "*.pem" "*.secret" "*password*" "*token*"
```
to check if sensitive files were ever committed and removed (they may still be in history).

Flag as: `[DISCLOSURE] <file>:<line> — <description of sensitive data>`

## Step 5 — Structural & formatting issues

For each Markdown file:

- Verify heading hierarchy (no jumping from H1 → H4, no duplicate H1s)
- Check that all internal links (anchors) resolve to real headings
- Look for broken image references (files that don't exist in the repo)
- Identify orphaned files — files tracked in git but not referenced anywhere
- Check for inconsistent list styles (mixed `*` and `-` bullets in the same file)
- Look for trailing whitespace or mixed CRLF/LF line endings

Run:
```bash
git -C . ls-files | xargs file | grep "CRLF"
```

Flag as: `[STRUCTURE] <file>:<line> — <reason>`

## Step 6 — Consistency audit

Cross-check facts that appear in multiple places:

- Project names, descriptions, and tech stacks in `README.md` vs `projects.md` — are they identical or contradictory?
- Any social links or usernames (GitHub, LinkedIn, website) — are they consistent and all pointing to the right accounts?
- The Spotify embed UID — verify it matches the public profile

Flag as: `[CONSISTENCY] <file A>:<line> vs <file B>:<line> — <discrepancy>`

## Step 7 — Link rot check

For each external URL found in the file list, perform a HEAD request and verify it returns 2xx or 3xx:

```bash
# Example for extracting all URLs from markdown files
grep -rEoh 'https?://[^)"\s>]+' . --include="*.md" | sort -u
```

Then check each URL. Flag any that return 4xx/5xx or time out.

Flag as: `[DEAD LINK] <file>:<line> — <url> returned <status>`

## Step 8 — Compile findings & produce report

Collect all flagged findings from steps 2–7. Produce a structured report in the following format and write it to `due-diligence-report.md` at the repo root (add this file to `.gitignore` so it's never committed):

```markdown
# Due Diligence Report
Generated: <date>

## Summary
| Category | Count |
|---|---|
| Typos / Grammar | N |
| Stale Info | N |
| Disclosure Risk | N |
| Structural Issues | N |
| Consistency Issues | N |
| Dead Links | N |

## Findings

### 🔤 Typos & Grammar
...

### ⏳ Stale Information
...

### 🔒 Potential Information Disclosure
...

### 🏗 Structural Issues
...

### 🔄 Consistency Issues
...

### 🔗 Dead Links
...
```

## Step 9 — Fix issues (with approval)

present the report to the user via `notify_user`. For each category:

1. **Auto-fixable** (typos, grammar, trivial formatting): ask for approval to apply all at once
2. **Requires judgement** (stale info, disclosure risks): present each one individually and wait for user decision before editing
3. **Dead links**: suggest replacements or flag for manual review

Once approved, apply fixes file by file and confirm each change.

## Step 10 — Final pass

After all fixes are applied:
- Re-read every changed file to confirm the fix looks correct in context
- Ensure no new issues were introduced by the edits
- Run the link check (step 7) again on any file that had dead links fixed

---

> **Tip:** Run this workflow before pushing major changes, publishing a new blog post link, or before sharing the repo publicly.
