# adversarial-review

> The builder is the worst reviewer. Fresh eyes find bugs that context hides.

A Claude Code skill that spawns a **zero-context subagent** to adversarially review your code. Unlike normal code review — where the same Claude that wrote the code tries to review it — this skill gives the reviewer **no build history, no design rationale, no "what the author meant."** Just the code and the original ask.

The subagent assumes every line is wrong until proven correct.

## Why

Every AI coding agent has a fatal flaw: **builder's blindness.** The same Claude that writes code cannot reliably review it, because its context is polluted by intentions, design decisions, and rationalizations from the build session.

This skill exploits that weakness — by spawning a fresh Claude that was never in the room.

## Install

```bash
claude plugins install lesue1/adversarial-review
```

Or manually:

```bash
git clone https://github.com/lesue1/adversarial-review.git ~/.claude/skills/adversarial-review
```

## How it works

```
You finish writing code
  → adversarial-review auto-triggers
    → spawns a fresh subagent (zero context)
      → arms it: "Assume every line is wrong. Find reproducible bugs."
        → reports only findings with concrete reproduction steps
          → you decide what to fix
```

## What it catches

- Null/undefined crashes from edge cases
- Type confusion and boundary violations
- Race conditions and async mistakes
- State corruption and silent failures
- Logic errors the builder couldn't see

## What it doesn't catch

- Style issues and naming (different tool for that)
- Security vulnerabilities (different tool for that)
- Architecture problems (different tool for that)

**One skill, one job: find reproducible bugs that the builder's context hides.**

## Example

After building a period tracker app, adversarial-review found:

| Severity | Bug | Regular review caught? |
|----------|-----|----------------------|
| 🔴 CRITICAL | `toISOString()` returns UTC — all calendar dates wrong for UTC+8 | ❌ No |
| 🔴 CRITICAL | DB error → infinite loading spinner, app permanently broken | ❌ No |
| 🟡 MEDIUM | Changing start date after setting end date → end before start | ❌ No |

All three were invisible to the builder. All three had concrete reproductions.

## License

MIT
