---
description: Creates/updates CURRENT.md — unified snapshot of project state + session continuation notes. Use before /clear (end of session) OR after a major change. Start the next session with "read CURRENT.md".
---

# Role

You are a technical documentarian. Your job is to create (or update) `CURRENT.md` — the **single handoff file** that combines:

- **Session continuation** (per-session: where you stopped, where to continue)
- **Project state** (long-lived: what the project is, what works, structure, env)

One file, one source of truth. Replaces the classic `HANDOFF.md` / `resume.md` pattern.

---

# Your task

1. **Use the current session context as the primary source.** Everything you need (what was done, what failed, decisions) is already in the conversation. Read files ONLY to verify facts you are unsure about. NO broad exploration of project structure or code — this skill runs when the context window is nearly full, and exploration can exhaust it.
2. **Capture git state** — run `git status` and `git log -1 --oneline` (nothing more).
3. **Review the session history** — requests, actions, failed attempts, conclusions.
4. **Check remaining gaps** against `docs/SPEC.md` / `docs/PRD.md` / `docs/IMPLEMENTATION.md` — only if these files exist; skip otherwise.
5. **Create/update `CURRENT.md`** in the project root using the template below.

## Update rules (when CURRENT.md already exists)

- **Session handoff + status checklists** (Where to continue, Git state, Works / In progress / Not implemented): regenerate every time (snapshot, not a diary).
- **Gotchas + Architectural decisions**: MERGE — add new items, NEVER delete old items unless the current session explicitly invalidated them.
- **Static sections** (Project overview, Tech stack, Structure, Configuration, How to run): if they exist and nothing substantial changed, leave them untouched. Do not regenerate them each run — they duplicate CLAUDE.md/README and churn the file.
- **Language migration**: if the existing file is not in English, rewrite it into English — section headings to the template names below, and translate the content you carry over. Older versions of this skill produced Slovak and Czech headings (`Kde pokračovať`, `Čo funguje`, `Ako spustiť`) plus emoji, and a file left in that state keeps pulling the next session into the wrong language. This overrides the rule above: a section in the wrong language is a section that changed.

---

# Output format: CURRENT.md

```markdown
# CURRENT.md

Snapshot of the current project state + where to continue. Read me first before working.

**Last updated:** [YYYY-MM-DD HH:MM]

---

## Where to continue (session handoff)

**Last session:** [one-line description of the main task]
**Active skill/workflow:** [slash command that was driving the work, with arguments, e.g. `/academy-quiz 2.5.3` — or "none"]

### Done and working
- [concrete changes, files, functions — with file paths]

### Tried and NOT working
- [approach A] → why it failed → reverted / replaced with what

### Where I got stuck
[Concrete: what problem, what was verified, what is suspected. Max 5 sentences.]

### TODO for next session
1. [concrete next step]
2. [...]

### Important gotchas
- [knowledge you cannot read from the code — rate limits, edge cases]

### Architectural decisions (must not disappear)
[Discussion conclusions, trade-offs. This cannot be recovered from git.]

---

## Git state

- **Branch:** [current branch]
- **Last commit:** [hash + message]
- **Uncommitted:** [count of modified/untracked + key files]

---

## Project overview

**Name:** [name]
**Description:** [one-sentence description]
**Status:** [Development / MVP / Production]

---

## Tech stack

- **Frontend:** [technology or "none"]
- **Backend:** [technology]
- **Database:** [type]
- **Hosting:** [where deployed]

---

## Project structure

\`\`\`
[tree of the main files]
\`\`\`

**Key files:**
- `[file]` — [what it does]

---

## Works

- [x] [Feature] — [short description]

## In progress

- [ ] [Feature] — [what is done, what is missing]

## Not implemented yet

- [ ] [Feature from PRD]

---

## Configuration

**Environment variables (.env):**
\`\`\`
[NAME] — [what it is for]
\`\`\`

---

## How to run

\`\`\`bash
[commands: install, dev, build]
\`\`\`

---

## Production URLs (if any)

- **App:** [URL]
- **Admin/API:** [URL]
```

---

# Rules

- **English only** — write `CURRENT.md` in English no matter what language the conversation is in, and keep section headings exactly as the template names them. The file is read back as context by a later session; a mixed-language file drags that session into mirroring the wrong language.
- **Facts only** — no evaluations.
- **Current state** — what IS, not what SHOULD BE.
- **Concrete** — exact file names, commands, URLs.
- **Under 500 lines** — if it grows, shorten or move details to `docs/`.
- **NEVER summarize context** — quote key decisions verbatim, do not summarize.
- **No marketing** — no "successfully completed", "great progress". Facts only.
- **No emoji** anywhere in the file.

---

# After creating the file

1. Print its full absolute path on its own line (e.g. `/path/to/project/CURRENT.md`) so it is copy-paste ready.
2. Put a ready-to-paste **resume prompt** on the system clipboard, so the next session is one paste away. Copy this exact text, with the real absolute path filled in:

   ```
   Continuing a previous session. I ran /clear, so you have no history — but this is not a fresh start.

   1. Read <ABSOLUTE PATH TO CURRENT.md> first and treat it as your primary context. It records what was done, what was tried and failed, gotchas, architectural decisions and the TODO to pick up.
   2. Load the <SKILL> skill before continuing — the previous session was working under it.
   3. Do not re-explore the project beyond verifying what you actually need. The file is the handoff; trust it unless something contradicts it.

   Then tell me in two or three sentences where we are and what you are picking up next, and wait for my go-ahead before changing anything.
   ```

   Line 2 is conditional: include it **only** if a slash command or skill was actually driving the previous session (the same value you wrote into "Active skill/workflow"), and name it exactly, with its arguments. If none was, drop the line entirely and renumber.

   Write it to the clipboard with a quoted heredoc so nothing gets shell-expanded:

   ```bash
   cat <<'PROMPT' | pbcopy
   [the resume prompt above, path and skill already filled in]
   PROMPT
   ```

   Keep the heredoc body flush left in the command you actually run — a heredoc preserves leading whitespace, and indented lines would paste with the indentation attached.

   `pbcopy` is macOS. If it is unavailable, try `wl-copy`, then `xclip -selection clipboard` (Linux), then `clip.exe` (WSL). If none works, say so in one line and stop — the printed path from step 1 is the fallback.
3. Do NOT print the whole file content into the chat.
4. Tell the user the next steps: "Run /clear, then paste (Cmd+V) and hit Enter — the resume prompt is already on your clipboard." Mention that the clipboard is lost if they copy something else in between, and the path above is the fallback.
