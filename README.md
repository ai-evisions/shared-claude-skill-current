# Claude Code skill: `/current`

A single slash command that writes **`CURRENT.md`** — a controlled handoff snapshot of a project — so you can wipe the context window and resume the next session from facts *you* chose to keep.

Cross-team resource. Works in any project, any language, any stack.

---

## Why this exists

When a Claude Code session fills up the context window you have two options:

| | What happens | Who decides what survives |
|---|---|---|
| `/compact` | Claude auto-summarises the conversation and keeps going | Claude |
| `/clear` | Context is wiped completely | Nothing survives |

Neither preserves the expensive knowledge from a long session: the approach that **failed** and why, the rate limit you discovered the hard way, the trade-off you argued about for twenty minutes. An auto-summary may drop any of it, and git will never bring it back.

`/current` writes that knowledge to disk first, in a fixed structure. Then `/clear` is safe.

### The workflow that actually works

```
/current                       # write the handoff file
/clear                         # wipe the context window
read /path/to/CURRENT.md       # load your controlled context back
```

Two things people get wrong:

- **`CURRENT.md` is not loaded automatically.** After `/clear` it just sits on disk. You have to ask for it — that is why the skill prints the full path at the end, ready to paste.
- **`/current` + `/compact` is mostly redundant.** `/compact` does not empty the context, it replaces it with its own summary and carries on from there — so the session keeps running on the auto-summary you were trying to avoid. Pair `/current` with `/clear`, not `/compact`. (If you do stay on `/compact`, follow it immediately with `read /path/to/CURRENT.md` so the controlled version wins.)

---

## Install

**User level** — available in every project:

```bash
cp commands/current.md ~/.claude/commands/current.md
```

**Project level** — committed with the repo, shared with everyone working on it:

```bash
mkdir -p .claude/commands
cp commands/current.md .claude/commands/current.md
```

Start a new Claude Code session and `/current` shows up in the slash command list. No dependencies, no config, no API keys — it is a single markdown file.

---

## Usage

Run it at the **end** of a session, before you clear:

```
/current
```

Also worth running after any major change you would not want to re-explain — a finished refactor, a resolved bug hunt, a decision you argued through.

The skill prints the path it wrote and nothing else. It deliberately does not dump the file contents into the chat, because that would eat the context you are trying to reclaim.

---

## What lands in CURRENT.md

Handoff first, because that is what the next session needs to read before anything else:

- **Where to continue** — done and working / tried and NOT working / where I got stuck / TODO / gotchas / architectural decisions
- **Git state** — branch, last commit, uncommitted files
- **Project state** — overview, tech stack, structure, what works, what is in progress, what is not implemented
- **Operational** — configuration, how to run, production URLs

See [`examples/CURRENT.example.md`](examples/CURRENT.example.md) for a filled-in file.

### Update semantics

Re-running the skill does not blindly overwrite everything:

- **Gotchas** and **Architectural decisions** are *merged* — old entries are never deleted unless the current session proved them wrong. These accumulate across sessions and are the whole point of the file.
- **Handoff and status checklists** are regenerated each run. It is a snapshot, not a diary.
- **Static sections** (tech stack, structure, how to run) are left alone when nothing substantial changed. They duplicate `README.md` / `CLAUDE.md` and do not need churning.

### Design constraints baked into the skill

- **No project exploration.** The skill runs when the window is nearly full, so re-reading the whole codebase would defeat the purpose. It writes from session context and only verifies facts it is unsure about, plus one `git status` / `git log -1`.
- **Facts only** — no "successfully completed", no progress cheerleading, no evaluations.
- **Under 500 lines**, no emoji, quotes over summaries for key decisions.

---

## Limits

- Only as good as the session it was run in. If the session never established why something failed, the file will not invent it.
- Not a substitute for `CLAUDE.md`. `CLAUDE.md` holds standing rules; `CURRENT.md` holds current state. Different lifetimes.
- One file per project, in the project root. Multiple parallel workstreams in one repo will fight over it.

---

## Contributing

Branch, PR, no direct pushes to `main` (see the [house rules](https://github.com/ai-evisions/ai-evisions)). `commands/current.md` is the source of truth — if you change the template, update the install instructions and the example in the same PR.
