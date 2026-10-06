---
name: testing-plan
description: Use when a feature, fix, or implementation plan has been completed and needs verifying end to end; when asked to "test this", "create a testing plan", "QA the changes", or "do as much testing as you can yourself"; or when an implementation plan's execution (e.g. superpowers executing-plans / subagent-driven-development) reaches the end of its tasks.
---

# Testing Plan

## Overview

Writes a testing plan for a defined set of changes, in a task-by-task, checkbox style (the same
shape as `superpowers:writing-plans`, if you use it), and then **runs it**. It verifies behaviour the way a user meets it,
in the running app, with evidence read back from the network and the data layer. The person is
asked only for what genuinely needs them.

**Core principle:** the human gets the residue, not the default. Every scenario is yours until an
observable check proves you can't do it.

## Step 1 — Scope

Define "the changes" from the most specific source available, in this order: the implementation
plan and spec, then `git diff <merge-base>..HEAD` plus the working tree, then the user's description.
List each behaviour change in one line. Unit and route tests that already exist are the floor.
The plan tests what they can't: real SQL and JSON writes, UI wiring, layout, side effects
(notifications, audit, calendar) and cross-role effects.

## Step 2 — Capability inventory (before assigning anything to the human)

Check each item with a command or tool call. Don't assume.

| Check | How | If available |
|---|---|---|
| Dev servers already running | `lsof -nP -iTCP -sTCP:LISTEN`, curl the health route | Reuse them. Only start what's missing. The port must be in the API's CORS allow-list. |
| Browser automation | Load the browser tools and read the tab context | Drive the UI yourself |
| Already signed in | Navigate to the app and screenshot. Do this now: the result decides every UI scenario's Owner, so it can't wait until Step 4. | Use the session. No login needed. |
| Read-only data access | DB env var **names** only (`grep -oE '^DB_[A-Z_]+='`), `pg` / `psql` / Prisma via `dotenv -e` | Read state back after each action |
| Network log | Browser network-request tool | Check status codes per action |
| Test files for uploads | Create them in the scratchpad | Upload through the file-input tool |

A scenario goes to **Needs you** only when a check fails for a reason you can't clear. Examples:
credentials or MFA, someone else's inbox or calendar, a device you can't reach, production or
shared data, anything irreversible outside test data. Ask for all of these in one message, at
the point they block you, and keep running every other scenario meanwhile.

## Step 3 — Write the plan

Save it to `docs/testing/YYYY-MM-DD-<feature>-testing.md` (or next to the repo's existing plans
if it keeps them elsewhere), with the project's doc header convention — e.g. the `doc-design`
skill's header, if installed. Structure:

```markdown
# <Feature> Testing Plan
**Goal:** <one sentence>   **Changes under test:** <plan/spec path or commit range>
**Environment:** <servers reused/started, URL, signed-in role, test project/record>

## Global Constraints
- Touch only test data you create, prefixed `QA-`; never edit or delete pre-existing records.
- Data-layer checks are read-only.
- Never print secret values. List env var names only; never "redact" values with sed.
- Exercise the app through its UI, as a signed-in user would. Never pull auth tokens out of
  browser storage or cookies to call the API directly.
- No `git commit`. Code fixes are made test-first and staged; the user decides when to commit.

## Scenarios
### S1: <behaviour>   — Owner: Self | Needs you (<why>)
- [ ] **Action:** <exact UI/API steps>
- [ ] **Expected (UI):** <what renders>
- [ ] **Expected (network):** <endpoint → status>
- [ ] **Expected (data):** <rows/fields after>
```

Cover the golden path, each edge case the spec implies (duplicates, no-op, cleared values,
missing party or permission), and any regression in neighbouring features. The plan's Review
Focus items become scenarios. Test a narrow/mobile width **only when the user asks for it** — never
resize the browser window by default, and if you do resize, restore it before moving on.

Post a short chat summary (goal, a table of scenarios with their Owner, and what needs the
human), then **start running immediately**. Test steps on test data are not a permission
question.

## Step 4 — Run

For each scenario: act, then check every Expected layer, and tick the box only with evidence.
Take a screenshot before any coordinate click. After an async action, read the network log and
query the data layer.

- **Pass:** tick, one-line evidence.
- **Fail in the code under test:** find the root cause (`superpowers:systematic-debugging`, if
  installed) → failing test → fix → re-run the scenario → run the full suite.
- **Fail outside the change** (pre-existing): record it as a finding. Don't fix it without asking.
- **Can't tell** (for example, data that predates the change): prove which it is (timestamps
  compared with the server start time, a fresh repro) before ruling.

## Step 5 — Clean up and report

Close the tabs you opened and stop the servers you started. List every piece of test data you
created, and ask whether to remove it. Then report:

1. **Results table:** scenario → Pass / Fail / Fixed / Needs you.
2. **Fixed during testing:** what it was, which file, the test that proves it, and whether the fix is staged.
3. **Findings not fixed:** pre-existing issues, one line each.
4. **Not verified:** exact steps the human should run, and why you couldn't.
5. **Test data left behind.**

## Red flags — stop and re-check the inventory

- "I have no credentials, so the UI is yours." Did you check for an existing session?
- "The UI updated, so it works." What do the network log and the data layer say?
- "Say the word and I'll run it." Non-destructive test steps just run.
- "I'll quickly fix this pre-existing bug too." It's a finding; ask first.
- Printing a `.env` line in any form.
- "I'll grab the Bearer token from the browser and curl the API." Drive the UI instead.
- "Owner depends on a check I'll do later." Run the inventory check now.
