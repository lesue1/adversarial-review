---
name: adversarial-review
description: >
  The builder is the worst reviewer. When the user finishes writing code, adding a feature,
  fixing a bug, or says "review this"/"check my code"/"looks good?", spawn a FRESH subagent
  with ZERO context from the build session. The subagent's job is adversarial: assume every
  line is wrong until proven correct. Only report findings that can be reproduced with
  concrete inputs. Do NOT trigger for trivial changes (typos, comments, formatting only).
---

# adversarial-review

> The builder is the worst reviewer. Fresh eyes find bugs that context hides.

## When to fire

Auto-trigger when the user finishes any non-trivial code work. Also fire directly when the user says "检查一下""帮我看看""审一下""review""check my code"。

Do NOT fire for: typo fixes, comment edits, formatting-only changes.

## What it does

1. **Capture the work** — the files changed + the user's original ask
2. **Spawn a fresh subagent** — same model, zero conversation history, brand new context
3. **Arm it with adversarial orders** — see `references/reviewer-prompt.md`
4. **Feed results back** — only reproducible findings, each with a concrete input that triggers the bug

## The subagent's mission

```
You did not build this code. You have no idea what the author intended.
Your job is NOT to be helpful. Your job is to BREAK things.

For every line: "What input would make this fail?"
For every function: "What edge case did they forget?"
For every assumption: "What if this assumption is wrong?"

If you cannot reproduce a bug with concrete inputs, it doesn't count.
```

## Core rules

- **Reproducible or it didn't happen.** No "this looks suspicious." Show the exact input that breaks it.
- **Fresh context only.** The subagent gets the code + the user's original ask. Nothing else. No build history, no rationale, no "what the author meant."
- **Verified correct is a valid result.** If the code survives adversarial review, say so. "I tried X, Y, Z to break it and couldn't" is a strong signal.
- **Don't review the plan. Review the artifact.** The code as it exists, not the code as it was intended.

## Output format

```markdown
# 🔴 Adversarial Review

## Reproducible bugs
| # | File:Line | Trigger | Expected | Actual |
|---|-----------|---------|----------|--------|
| 1 | auth.ts:42 | Login with empty password | Error message | 500 crash |

## Verified safe (tried and couldn't break)
- auth.ts:token refresh — tested expired token, malformed token, concurrent refreshes
- ...

## Edge cases not covered (not bugs, but gaps)
- What happens when the user uploads a >100MB file?
```

Then ask: "修哪些？"

## Gotchas

See `references/gotchas.md` for false-positive patterns and stealth-bug patterns.

## References

- `references/reviewer-prompt.md` — the exact prompt to inject into the fresh subagent
- `references/gotchas.md` — patterns that look like bugs but aren't, and vice versa
