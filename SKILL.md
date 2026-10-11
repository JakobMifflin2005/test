---
name: commit-message-write
description: Writes Conventional Commit messages from the staged diff only. Use when the user asks to write a commit message, draft a commit message, suggest a commit message, generate a conventional commit, or summarize staged changes for git commit. Reads git diff --staged before writing; refuses secrets; flags mixed changes, tests, migrations, API breaks, debug leftovers, and dependency edits.
---

# Commit message write

Produce one Conventional Commit message from the **staged** change. Do not invent unstaged work. See [examples](references/examples.md).

## Format

```
<type>(<scope>): <subject>

<body>
```

- **type**: `feat` | `fix` | `docs` | `test` | `refactor` | `chore`
- **scope**: optional, lowercase project area (`auth`, `api`, `db`, …)
- **subject**: ≤50 chars, imperative mood, no trailing period, does not restate the type
- **body**: blank line after subject; wrap at 72 chars; required unless the subject alone is enough; explain why and what the diff omits
- **breaking**: `!` after type or scope (`feat(api)!:`); explain the break in the body

## Required checks

Run `git diff --staged` (and `git diff --staged --stat` if helpful). Apply every row that matches:

| Finding | What the message does |
| --- | --- |
| Unrelated changes mixed together | Say so; propose how to split into separate commits |
| Tests added, changed, or removed | Name them in the body; give a reason if a test was removed |
| Database migration or schema change | Name the migration; state whether it is reversible |
| Public interface or API change | Describe old vs new behavior; mark breaking if callers break |
| Secret, credential, or key in the diff | **Refuse** to write a message; say what was found and where |
| Debug output or commented-out code left in | Point it out; ask whether it belongs in the commit |
| Dependency added, removed, or upgraded | Name the dependency and version change in the body |

## Steps (in order)

1. Confirm there is a staged change. If the index is empty, stop and say so.
2. Run `git diff --staged`. Read paths, hunks, and renames—not only the stat summary.
3. Scan for each required check above. Record every hit before drafting.
4. If a secret/credential/key is present: refuse; report path and kind; do not propose a message that would bless the commit.
5. Otherwise choose `type` (and optional `scope`) from the staged intent. Prefer one coherent theme; if themes conflict, call out the mix and suggest a split before or instead of a single message.
6. Write `subject` (≤50, imperative). Write `body` when anything non-obvious, any check fired, or a break exists. Wrap body at 72.
7. Output only the commit message (plus any refusal, split advice, or “does this debug belong?” questions). Do not run `git commit` unless the user asks.

## Output rules

- Never log or repeat secret values from the diff.
- Do not invent file names, test names, versions, or migrations absent from the staged diff.
- Prefer a precise scope over omitting one when the touched area is clear.
