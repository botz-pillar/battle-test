<!-- assets/readme-banner.png will be inserted here once rendered -->

# battle-test

A Claude Code skill that runs a council of personas over any artifact before you commit to it.

---

## What it is

`battle-test` is a Claude Code skill. Point it at any artifact — a blog draft, a lab plan, an architecture proposal, a contract section, anything you'd want a second opinion on — and it dispatches nine personas in parallel. Each one reads the artifact through their own lens, files findings tagged Critical / Material / Polish, and votes Ship / One More Round / Structural Rework. The votes don't always agree; the gate decides anyway. You get a markdown review log and an HTML report in about 60 seconds.

<!-- assets/launch-video.mp4 will be embedded here once rendered -->

## Why this exists / what skills are for

This repo is also a worked example of a Claude Code *skill*. If you've found yourself prompting Claude with the same scaffolding over and over for some workflow you keep doing, that's a skill waiting to be written. battle-test is mine for review-before-you-commit. Yours might be different — and the structure of this repo (skill file + persona prompts + templates + plugin manifest) is the shape that's worked for me. Read it, take what you want.

## Install

```
claude plugin marketplace add botz-pillar/battle-test
claude plugin install battle-test@battle-test
```

(Claude Code installs plugins via marketplaces. The first command adds this repo as a single-plugin marketplace; the second installs the skill from it.)

> **Before running on your own artifacts, see [Data flow & sub-processors](#data-flow--sub-processors) below.**

## First run

Try it on the included sample first — see what a real council critique looks like before pointing it at your own work:

```
/battle-test examples/sample-blog-post.md
```

The sample is deliberately mid-tier (looks fine on first read; has hidden slop, weak claims, vague CTA). The council will surface real findings.

To preview without dispatching any Claude calls:

```
/battle-test examples/sample-blog-post.md --dry-run
```

## Example output

See [`examples/sample-review-output.html`](examples/sample-review-output.html) for what running on the sample produces.

> *Generated, not handwritten. The output ships unedited.*

## The 9 personas

| Persona | Role lens |
|---------|-----------|
| `target-audience-primary` | Generic primary-audience template — customize for your domain |
| `domain-practitioner` | Subject-matter expert in your field |
| `ai-security-researcher` | Adversarial AI / agentic security expert (OWASP Agentic Threats, NIST AI RMF, MITRE ATLAS) |
| `instructional-designer` | Adult-learning / curriculum design |
| `copywriter` | Direct-response / community voice |
| `policy-risk-reviewer` | Regulatory / legal / brand-risk lens (directional only — not legal advice) |
| `audience-skeptic` | Reader who's been burned by similar content before |
| `future-self` | You, six months from now, rereading your own work |
| `contrarian` | Fatal-flaw hunter (non-voting) |

## How it works

1. **Stage detection** — the skill picks a stage from the filename (or `--stage` if you override).
2. **Roster selection** — picks 3 personas (iterate tier) or 5 (gate tier, includes the contrarian).
3. **Parallel dispatch** — each persona runs in a separate subagent with `allowed_tools=[]`. The artifact is fenced inside a per-dispatch nonce-tagged tag so the skill can't be tricked by directives in the artifact itself.
4. **Synthesis + verdict** — a synthesizer reads the validated structured outputs (not the personas' free-form text) and produces a headline + the deterministic verdict.

## Severity + verdict

Each finding is tagged:

- **Critical** — would materially break the artifact or fail a hostile scan. Mandatory fix.
- **Material** — meaningful gap or weakness. Should fix before publish.
- **Polish** — refinement that improves quality but isn't blocking.

The deterministic gate:

| Verdict | Rule |
|---------|------|
| **Ship** | All voters Ship AND zero unresolved Critical AND ≤1 unresolved Material |
| **One More Round** | Mixed votes OR ≥2 unresolved Material findings (no Critical) |
| **Structural Rework** | Any unresolved Critical OR ≥2 voters Structural Rework |

> The gate function is deterministic given persona votes; the persona votes themselves are LLM-generated and have run-to-run variance. Run twice on the same artifact and you may see different finding counts and (rarely) a different verdict. The gate stops the obvious slop from shipping; it doesn't claim to be a reliability instrument.

## Customize it

The personas in `skills/battle-test/personas/` are markdown prompt files. Edit them. The two you'll customize first:

- `target-audience-primary.md` — your specific audience profile. The more specific, the sharper the findings.
- `domain-practitioner.md` — a senior practitioner in your field. Replace the generic template with your domain (medical writing, legal drafting, fiction, GTM copy, security engineering — whatever fits).

The `personas/AUTHORING.md` file is intentionally not here. The persona file contract is inline at the top of each `.md` — copy any of them as a starting point and rewrite. Five sections: Identity, Vocabulary, Triggers, Anti-triggers, Anchor date.

## Data flow & sub-processors

When you run battle-test, the artifact's full content is sent to Anthropic (an external AI sub-processor) as input to each persona subagent.

**Anthropic is a sub-processor of any artifact you point this skill at.**

If your artifact contains regulated data (PII, PHI, CUI, attorney-client material, ITAR, financial records, customer-confidential information), confirm that your organization permits sending that data to Anthropic before running. Battle-test does not make that determination for you.

For HIPAA / PCI / FedRAMP / EU AI Act / GDPR / CUI workflows: run your vendor-risk process on Anthropic before adoption. Anthropic's DPA and trust information: <https://www.anthropic.com/legal>

**Residency:** calls dispatch via the Claude Code session you're already authenticated to. Whatever residency your Claude Code config + Anthropic contract establish is what applies. Battle-test does not constrain or override that.

**This tool is informational, not a compliance control.** It produces a timestamped, hash-stamped record suitable for inclusion in a publishing or review-workflow audit trail. It does not satisfy a regulatory control on its own.

The pre-flight banner asks you to confirm the artifact is approved for third-party AI processing before each run. The `--yes` flag skips the prompt; pair it with `--data-classification-confirmed` in scripted use, otherwise the skill warns you on stderr.

## Security architecture

Four numbered guarantees. All four ship in v1.0.

1. **Subagent least-privilege.** Persona subagents are dispatched with `allowed_tools=[]`. Even on full prompt-injection success against a persona, the resulting subagent has no tools — no `Read`, no `Bash`, no network. The orchestrator pre-reads the artifact bytes and passes them inline.
2. **Nonce-tagged input fence.** Per-dispatch CSPRNG nonce on the `<artifact_<nonce>>` fence. Any literal `</artifact_<nonce>>` in the artifact is HTML-entity-escaped before interpolation, so the artifact cannot break out of the fence.
3. **Strict HTML escaping on all output.** Every interpolated string in the HTML companion (artifact quotes, persona findings, headline text — everything) passes through HTML-entity-escape. CSP `default-src 'none'`, no `<script>`, no `<a href>` from artifact, no `<img src>` from artifact, no `style=` attributes on interpolated content.
4. **Falsifiable security claim.** `examples/sample-adversarial.md` ships canonical prompt-injection payloads. `examples/sample-adversarial-expected.md` documents the expected verdict (Structural Rework) and a regression-response procedure. The kit's security claim is testable, not just asserted.

The synthesizer (which combines persona outputs into the verdict) gets the same data-vs-instructions treatment: persona payloads are wrapped in their own nonce-tagged `<persona_payload>` tags with strict-schema validation; the verdict is computed from the validated counts + votes, not from the synthesizer's free-form text.

## Known limitations

- **Indirect prompt injection.** A poisoned artifact can shift persona behavior. The kit's defenses (least-privilege subagents, input fencing, output escaping, falsifiable testing) limit blast radius and make regressions detectable, but do not absolutely prevent a sophisticated injection from biasing findings. Treat third-party content with appropriate skepticism. Defense-in-depth: read the artifact yourself before handing it to any AI tool.
- **Persona drift across model versions.** Personas are calibrated against `claude-sonnet-4-6` (default) and `claude-opus-4-7`. When the model versions change, persona behavior may shift. The CHANGELOG tracks the reference baseline.
- **English-only tuning.** Personas work on non-English artifacts but are calibrated against English. Results on other languages will be uneven.
- **Run-to-run variance.** LLM votes are stochastic. The gate is deterministic given the votes, but the votes themselves can shift slightly between runs.

## Roadmap

These aren't promises — they're the directions the kit is most likely to grow.

- An interactive `/battle-test --init` wizard for first-time customization.
- Per-persona model overrides in `models.json`.
- Listing on the Claude Code plugin marketplace when it ships.

No GitHub Action. No SaaS. No community persona-pack contribution structure — fork the repo if you want to extend it.

## License + author

MIT. Maintained by [Josh Botz](https://www.linkedin.com/in/joshthebotz/) (Pillar Security).
