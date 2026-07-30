# CLAUDE.md — shared-claude-skill-current

## What this repo is

Distribution repo for a single Claude Code slash command: `/current`, which writes a `CURRENT.md` session handoff. There is no application here — the deliverable is `commands/current.md`, a prompt file. Nothing runs, nothing builds, there are no dependencies.

## Source of truth

`commands/current.md` is the artifact. Everything else documents it.

It is meant to be **byte-identical to what people have installed** at `~/.claude/commands/current.md` (or `.claude/commands/current.md`). When editing it:

- Keep changes surgical. The file is a prompt sent to a model verbatim; small wording changes shift output more than they look like they should.
- Update `README.md` (install instructions, section list, update semantics) and `examples/CURRENT.example.md` in the same PR if the template structure changes.
- Do not add project-specific or client-specific content. This skill runs in every kind of repo.

## Conventions

- **English** for all files, including the template and its output.
- **No emoji** anywhere, including the template's section headings — the skill explicitly forbids them in its own output.
- **No secrets**, ever. Nothing here should require any. `.gitignore` covers the usual suspects as a safety net.
- **Branch + PR**, never a direct push to `main` (agency house rule, see https://github.com/ai-evisions/ai-evisions).

## Testing a change

There is no test suite. Validate by using it:

1. Install the modified `commands/current.md` at user level.
2. Run `/current` at the end of a real, long session in an actual project.
3. Check the output against the rules in the skill's own "Rules" section — facts only, no marketing language, no emoji, under 500 lines, gotchas and architectural decisions preserved rather than overwritten.

A change that has not been run against a real session has not been tested.
