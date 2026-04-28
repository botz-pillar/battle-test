---
description: Run a persona-driven council review on any artifact before you commit to it.
argument-hint: <path> [--stage <stage>] [--tier {iterate|gate}] [--dry-run] [--yes] [--data-classification-confirmed] [--no-html] [--no-markdown]
---

# /battle-test

Invoke the `battle-test` skill on `$1` (the artifact path) with the optional flags.

Load the skill at `skills/battle-test/SKILL.md` and follow its instructions exactly. Do not deviate from the skill's persona-selection logic, dispatch architecture (`allowed_tools=[]`, nonce-tagged fence, orchestrator pre-read), synthesizer-input fencing, or HTML escaping rules.

If `$1` is missing, print the usage message and exit:

```
Usage: /battle-test <path> [--stage <stage>] [--tier {iterate|gate}] [--dry-run] [--yes] [--data-classification-confirmed] [--no-html] [--no-markdown]

Examples:
  /battle-test draft.md
  /battle-test posts/2026-q2-launch.md --tier gate
  /battle-test outline.md --dry-run --json

For a first run, try the included sample:
  /battle-test examples/sample-blog-post.md
```
