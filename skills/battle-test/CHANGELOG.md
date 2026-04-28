# battle-test skill changelog

## 2026-04-28 — v1.0.0 (initial public release)

Public release at [github.com/botz-pillar/battle-test](https://github.com/botz-pillar/battle-test).

**Reference baseline:**

- `persona_model`: `claude-sonnet-4-6`
- `synthesis_model`: `claude-sonnet-4-6`
- Rates pinned in [`models.json`](models.json) (`rates_as_of` field is the single source of truth — bump when re-verifying against current Anthropic pricing)

**Architecture (frozen at v1.0.0):**

- 9 persona archetypes: `target-audience-primary`, `domain-practitioner`, `ai-security-researcher`, `instructional-designer`, `copywriter`, `policy-risk-reviewer`, `audience-skeptic`, `future-self`, `contrarian` (non-voting).
- Operational severity rubric in `SKILL.md` § Severity rubric — shared thresholds across all personas; mechanical verdict math computed from validated counts and votes.
- Subagent dispatch with `allowed_tools=[]`. Orchestrator pre-reads the artifact and passes it inline inside a per-dispatch nonce-tagged `<artifact_<nonce>>` fence (literal closing-tags pre-escaped).
- Synthesizer-input fencing: each persona payload wrapped in a fresh-nonce `<persona_payload>` fence, validated against strict schema before synthesis. Verdict computed deterministically from validated values, not from synthesizer prose.
- Mandatory HTML-entity escaping on all rendered persona/synthesizer strings; CSP `default-src 'none'`; no `<script>`/`<a href>`/`<img src>`/`style=` from artifact content.
- Cost estimator with real formula reading rates from `models.json`; `--dry-run`, `--dry-run --json`, `--yes`, `--data-classification-confirmed` flags.
- Provenance footer: `git_sha` + `dirty` flag + `content_sha256` + `attestation` + `models_json_sha` + `cost_estimated`/`cost_actual`.
- Falsifiable security claim via `examples/sample-adversarial.md` + documented response procedure in `examples/sample-adversarial-expected.md`.
- HTML report opens with an artifact-summary block and word/byte size pill so reports differentiate at a glance.
- Threat model section in README enumerates in-scope adversaries (artifact author / filename-controlled / persona-output coercer / output-render exploiter) and out-of-scope (skill-file tamperer, Anthropic-side compromise, network adversary).
- Stage-detection warning in pre-flight banner when stage is auto-detected from filename (defensive nudge against filename-controlled jury selection).

**Known limitations** (acknowledged in README):

- Indirect prompt injection (mitigated, not prevented).
- Persona drift across model versions.
- LLM-self-assigned severity tags — rubric is shared, but no external calibration mechanism.
- English-only tuning.
- Run-to-run variance in LLM votes.

**License:** MIT.

## Persona prompt revisions

When you tweak a persona prompt, log it here as: `<date> — <persona-id> — <one-line summary>`.
