# testing-plan

A [Claude Code](https://claude.com/claude-code) skill that turns "now test it" into a written,
checkbox-style testing plan — and then has Claude **run that plan itself** in the real app,
checking each scenario against the UI, the network log, and the data layer, and handing you only
the steps it genuinely can't do.

## Why this exists

When an agent finishes a feature, "testing" tends to collapse into one of two failure modes:

- **It hands everything back.** "I don't have credentials, so please click through these 12
  steps" — without checking whether the browser is already signed in, the dev server is already
  running, or the database is readable.
- **It trusts the screen.** The UI updated, so it's declared working — while the request
  actually 500'd, or the row was written with the wrong value.

Unit and route tests don't close that gap either: they can't see real SQL/JSON writes, UI
wiring, side effects (notifications, audit logs, calendar invites), or cross-role effects.

This skill fixes both with one rule — **the human gets the residue, not the default** — and a
capability inventory that proves, with a real command, what Claude can and can't do before any
scenario is assigned to you.

## What it does

1. **Scopes the changes** from the plan/spec, the branch diff, or your description — one line
   per behaviour change.
2. **Takes a capability inventory** — running dev servers, browser automation, an existing
   signed-in session, read-only DB access (env var *names* only), network log, test upload
   files. Each is checked, not assumed.
3. **Writes the plan** to `docs/testing/YYYY-MM-DD-<feature>-testing.md`: global safety
   constraints, then scenarios with an Owner (Self / Needs you) and Expected results for UI,
   network, and data.
4. **Runs it immediately** — acts, verifies every layer, ticks boxes only with evidence. Bugs in
   the code under test are fixed test-first; pre-existing bugs are reported, not silently fixed.
5. **Cleans up and reports** — closes tabs, stops servers it started, lists test data it created
   (prefixed `QA-`), and reports Pass / Fail / Fixed / Needs you, plus exact steps for anything
   it couldn't verify.

Safety rails built in: test data only, read-only data checks, never prints secret values, drives
the UI instead of lifting auth tokens from the browser, and never commits.

See [SKILL.md](skills/testing-plan/SKILL.md) for the full workflow, plan template, and red flags.

## Install

Copy the skill directory into your Claude Code skills folder:

**User-wide** (available in every project):
```bash
git clone https://github.com/ErmisSolutions/claude-skill-testing-plan.git
cp -r claude-skill-testing-plan/skills/testing-plan ~/.claude/skills/testing-plan
```

**Project-scoped** (checked into one repo, shared with your team):
```bash
git clone https://github.com/ErmisSolutions/claude-skill-testing-plan.git
cp -r claude-skill-testing-plan/skills/testing-plan /path/to/your/project/.claude/skills/testing-plan
```

Then, in Claude Code, just ask:

> - "test this"
> - "create a testing plan for the invite flow and run it"
> - "QA the changes on this branch"
> - "do as much testing as you can yourself"

Claude Code will pick up the skill automatically based on its description — no slash command
needed. It also triggers at the end of an implementation plan's execution.

## Requirements

- [Claude Code](https://claude.com/claude-code)
- Browser automation (e.g. Claude in Chrome, Playwright MCP, or Chrome DevTools MCP) for UI
  scenarios — without it, UI scenarios fall to "Needs you".

Optional, used when installed:
- [Superpowers](https://github.com/obra/superpowers) — `systematic-debugging` for failures,
  and `writing-plans` / `executing-plans` / `subagent-driven-development` as the plans this
  skill tests.
- [doc-design](https://github.com/ErmisSolutions/claude-skill-doc-design) — header format for
  the saved plan.

## License

MIT — see [LICENSE](LICENSE).
