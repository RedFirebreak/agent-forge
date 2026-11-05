# gotcha

A registry of the traps your project has hit, written so the next person recognises one from the symptom
alone and doesn't lose the time again.

Every project collects traps that aren't bugs in its own code: the framework that silently drops a field,
the API whose timeout doesn't cover what you'd think, the toolchain flag that only matters on CI. They cost
an afternoon each, get fixed, and then cost someone else the same afternoon a month later. Coding agents
make this worse, because every new session starts from zero.

This skill keeps them in `docs/gotchas/`, one entry per trap:

```markdown
### HTTP-4
**A retry fires while the first request is still in flight, and the server records both.** The client's
timeout covers the connect phase only; a slow response never trips it, so the retry wrapper sees
"no response yet" and sends again. Set the read timeout explicitly, and make the retry idempotent with a
request key the server de-duplicates on.

*Source: `OBSERVED` (billing-service, 2026-09-28)*
```

The rules that make it work:

- **Symptom first.** The bold sentence is what you *see*, not the cause, so it can be matched before you
  know the answer.
- **Permanent IDs.** `HTTP-4` is cited from code comments, commits and other docs, so it never changes.
  Merged or moved entries are retired, never reused.
- **One index.** `docs/gotchas/README.md` lists every entry's symptom, grouped by area. You match against
  it, then open only the one file the match points to.
- **Stated once.** Each trap lives only in its own entry. The tables, index and counts are generated from
  the entries, so there is nothing to keep in step by hand.
- **Kept honest.** A small validator, plain Python with no dependencies, checks it all, and a hook runs it
  after every edit.

## Using it

Install it in Claude Code (other ways are in the [repository README](../../README.md)):

```text
/plugin install gotcha --marketplace RedFirebreak/agent-forge
```

Then just work. When a trap has cost real time,
say "that's a gotcha" or "write that down", or let Claude notice it. The skill:

1. **bootstraps** `docs/gotchas/` the first time, filling in the project-specific parts from your codebase
   and asking only when a guess would be shaky;
2. **checks for a duplicate** before minting an ID, and extends an existing entry when it's the same trap;
3. **files it** under the right area and prefix, creating a new area only when nothing fits, and saying so;
4. **writes the tables** and runs the validator until it prints `OK`.

It also handles promoting an open question once a live run settles it, and merging or retiring entries.

## The validator

`init` copies it into your project as `tools/check_gotchas.py`, which your project owns from then on:

```bash
python3 tools/check_gotchas.py            # validate; prints OK or every problem, each with what to fix
python3 tools/check_gotchas.py write      # write the tables, index and counts from the entries
python3 tools/check_gotchas.py next HTTP  # the next free ID in a prefix, counting retired ones
python3 tools/check_gotchas.py ids        # the highest ID in use per prefix
```

It checks entry order and format, source keys, prefix ownership, links and anchors, and leftover template
placeholders. It also scans every file git tracks for citations of retired IDs. `init` prints a
`PostToolUse` hook for `.claude/settings.json`, so a malformed entry fails as soon as it's written. The skill
asks you before adding it.

Requires Python 3.10 or later, and git for the citation scan.

## Options for bigger repositories

Both are a paragraph in your registry's README that the validator reads; the skill has the exact text.

- **Written on merge.** When several branches add entries at once (a team, or parallel agent sessions),
  the default branch writes the tables after every merge. Pull requests then never conflict in them. A
  ready GitHub Actions workflow ships with the skill.
- **Product prefixes.** For a monorepo whose products share one registry: IDs become `W-HTTP-3`, and the
  validator requires a product prefix on every one.

## Tell your CLAUDE.md about it

The skill adds one line pointing at the registry when it bootstraps. If you want more, paste this:

```markdown
## Gotchas

Known traps in the shared layer (framework, API, platform, toolchain) are recorded in `docs/gotchas/`,
one entry per trap under a permanent ID like `HTTP-4`.

**Before debugging anything that smells like a quirk rather than a bug in our code**, open
`docs/gotchas/README.md`, match what you're seeing against the symptom column of the index, and open only
the one area file that entry points to.

**When something has cost more than ~10 minutes and would bite another part of the project too**, use the
`gotcha` skill to record it before moving on, even mid-task.

**Cite by ID** wherever code works around a trap: `// See HTTP-4: read timeout must be set explicitly`.
Never cite an ID from the README's "Retired IDs" table.
```
