# README — Battle-Test Review Log

> Append-only log of `/battle-test` runs against this artifact.

**Artifact:** README.md
**Created:** 2026-04-28T13:30:14Z

---

## Review — 2026-04-28T13:30:14Z — stage:custom tier:gate

**Artifact:** [README.md](README.md)
**Personas:** target-audience-primary, domain-practitioner, ai-security-researcher, future-self, contrarian (non-voting)
**Verdict:** **Structural Rework**
**Findings:** Critical: 2 · Material: 9 · Polish: 9

### Headline

**Where the council agrees:**
All four voting personas voted One More Round; the contrarian filed two non-voting Criticals. Where they agree: the rewritten security architecture (mechanical verdict from counts+enum, persona-text-as-data) is genuinely strong and is the README's most credible move. Where they agree on gaps: an explicit threat model is missing (what adversary, what assets); the '~60 seconds' performance claim is unqualified; data-handling specifics (Anthropic retention regime, training-data exclusion flags, --data-classification-confirmed actual semantics) are underspecified for the regulated-data use case the README itself flags; framework name-drops (OWASP, NIST, MITRE ATLAS) lack control-mapping that would let a third party verify coverage.

**Where the council clashes (with resolution):**
No vote disagreement — every voting persona landed on One More Round. The substantive tension is at the architecture layer: contrarian argues the mechanical-verdict-from-counts claim is theater because the counts themselves are uncalibrated LLM-self-assigned severity tags, while ai-security-researcher partially agrees but treats this as a Material 'name the mitigation' rather than a Critical structural flaw. Resolution: contrarian's framing is correct — the same laundering attack the README closes for free-text prose still works for severity counts and votes. The honest move is to add a known-limitation acknowledging uncalibrated severity self-assignment, alongside the documented run-to-run variance.

**Blind spots:**
• future-self alone caught that hardcoded model IDs (claude-sonnet-4-6, claude-opus-4-7) in README will look stale by Q4 2026. Move to models.json with a 'last-verified' date.
• ai-security-researcher alone called out the supply-chain / skill-tampering surface (an attacker who edits SKILL.md or models.json gets full code execution as the user); Known Limitations doesn't acknowledge it.
• contrarian alone flagged that filename-driven stage detection lets the attacker pick their own jury by naming their artifact `*-DRAFT.md` vs `*-FINAL.md`. The same input that selects the persona roster is attacker-controlled.

**Recommendation:**
Don't ship as-is — but the gaps are surgical, not structural. (1) Add an explicit threat model section above 'Security architecture' naming adversary capabilities (artifact author, skill-file-tamperer, persona-output-coercer) and what each can/cannot move. (2) Replace hardcoded model IDs in README with a pointer to models.json + a calibration-date convention. (3) Move 'falsifiable security claim' out of the four numbered guarantees and into a separate 'Verifiability' subsection — it's a test-suite property, not a runtime guarantee. (4) Add a known-limitation acknowledging uncalibrated severity self-assignment alongside run-to-run variance — completes the honesty arc the README is already on. (5) Tighten data-handling: name retention regime and what --data-classification-confirmed actually gates. Then re-run. Verdict was honestly earned: the gate refusing to rubber-stamp its own README after one round of fixes is the credibility moment, not a failure mode.

### Per-persona reports

<details>
<summary>Click to expand full persona findings</summary>

#### Persona: target-audience-primary — mid-career security/AI practitioner
**Vote:** One More Round · **Counts:** C0 M2 P3

**Strengths:**
- Honest framing — 'Not a reliability instrument' and explicit acknowledgment that LLM votes are stochastic. Rare for tools in this space to admit this upfront.
- Falsifiable security claim with concrete regression threshold (≥2 SHIPs in 3 runs) is genuinely novel for a prompt-injection defense. Strongest signal that a practitioner wrote it.
- Subagent least-privilege with allowed_tools=[] is the right architectural choice and stated concretely, not as marketing.
- Mechanical verdict computation from validated enums (not synthesizer narrative) closes a real laundering channel.

**Findings:**
- [Material] (gap): The '60 seconds' claim has no context: what artifact size, what model, what concurrency? Practitioners need to know if a 5k-word lab plan also takes 60s or 5 minutes. Either qualify ('on a ~1500-word draft with sonnet-4-6') or drop the number.
- [Material] (gap): Data-flow section says 'Artifact content sent to Anthropic' but doesn't address: zero-retention API terms, default 30-day retention, or workspace config. Pointing at 'vendor-risk process' is a punt.
- [Polish] (simplify): 'Sub-processor.' as a one-word sentence reads like a checklist artifact, not prose. Either expand or fold into preceding sentence.
- [Polish] (gap): OWASP Agentic Threats / NIST AI RMF / MITRE ATLAS are name-dropped under ai-security-researcher persona but no indication of how deep that grounding goes.
- [Polish] (simplify): 'OMR' appears in the verdict table without expansion until the reader infers it from 'One More Round' two lines up.

**Anti-slop:** Voice mostly clean — terse, practitioner-pitched. 'Allergic to AI slop' energy is earned in places (the falsifiable claim, 'Not a reliability instrument'). Two soft spots: 'about 60 seconds' is the kind of unqualified stat that triggers my allergy, and 'worked example of a Claude Code skill' is mildly self-congratulatory.

#### Persona: domain-practitioner — senior security/agentic-systems engineer
**Vote:** One More Round · **Counts:** C0 M2 P3

**Strengths:**
- Subagent least-privilege with allowed_tools=[] is the right primary control. Defense even when injection succeeds aligns with NIST AI 600-1 GV-1.3 and ATLAS AML.T0051 mitigation posture.
- Mechanical verdict derivation from vote enum + severity counters closes the synthesizer-as-laundering-channel surface — a real threat-model insight, not theater.
- Falsifiable security claim with adversarial regression fixture and explicit variance handling (1 SHIP tolerated, 2/3 = regression) is unusually rigorous for a prompt-engineering artifact.
- Honest limitations section names indirect prompt injection as mitigated-not-prevented rather than overclaiming.

**Findings:**
- [Material] (gap): Threat model is implicit. Four guarantees are listed but adversary capabilities and assets-at-risk are never stated. Without an explicit threat model, a reviewer cannot tell whether allowed_tools=[] + escaping + CSP cover the surface or just the parts that were easy. OWASP LLM01/LLM02 and ATLAS expect an enumerated adversary section.
- [Material] (gap): Data-handling claims underspecified for the regulated-data use case the README itself flags. No statement on retention, training-data exclusion (zero-data-retention / no-train flags on the Anthropic API path), local logging of artifact content, or whether --data-classification-confirmed gates anything beyond a prompt.
- [Polish] (gap): CSP claim implies HTML rendering, but README never says where the HTML is rendered or by whom. If output is consumed in a terminal or as Markdown, CSP is irrelevant; if rendered in a browser, say so and name the renderer.
- [Polish] (gap): Verdict table edge case unclear: what if all voters Ship but there is 1 Critical from a non-voter (contrarian)? Rule says 'Any Critical → Structural Rework' which lets a non-voting contrarian force rework — intentional, but worth stating explicitly.
- [Polish] (simplify): Roadmap line mixes status date, install instruction, and CHANGELOG pointer in one sentence. Split.

**Anti-slop:** Reviewed as written. Did not invent CVEs or cite training-data Anthropic policy details — flagged the gap instead.

#### Persona: ai-security-researcher — adversarial AI / agentic security expert
**Vote:** One More Round · **Counts:** C0 M3 P3

**Strengths:**
- Mechanical verdict computation from validated enums/counters closes the persona-text-as-laundering channel — the right architectural answer to LLM-as-judge prompt injection (OWASP LLM01 / ATLAS AML.T0051), rarely seen in practice.
- Nonce-tagged fence with entity-escaping of literal close-tags is a sound data/instruction separation primitive; matches NIST AI 600-1 guidance on provenance and input integrity.
- Falsifiable security claim with documented regression procedure (≥2/3 SHIPs = regression) treats stochasticity honestly — most 'we tested prompt injection' claims are single-run anecdotes.
- Honest limitations section explicitly names indirect prompt injection residual risk, persona drift, and English-only tuning.

**Findings:**
- [Material] (gap): Framework reference precision: README's security claims map cleanly to OWASP LLM Top 10 (LLM01, LLM02, LLM08) and MITRE ATLAS techniques (AML.T0051, AML.T0054). Citing them would let third parties verify coverage rather than take 'security architecture' on faith.
- [Material] (gap): Guarantee 1 ('subagent least-privilege, allowed_tools=[]') is only as strong as the runtime's enforcement of that field. Recommend: (a) version-pin the Claude Code behavior the guarantee depends on, (b) add a startup self-check that asserts the dispatched subagent cannot in fact call a sentinel tool, or (c) downgrade the language from 'guarantee' to 'depends on Claude Code subagent isolation contract'.
- [Material] (gap): Laundering-channel closure is claimed for free-text fields but the schema also carries integer counters and a vote enum. An injected persona output that lies about its own counts (e.g., reports critical_count:0 when it found criticals, or votes Ship under coercion) still moves the verdict. Mitigations worth naming: cross-check counts against len(findings) filtered by severity; require quoted_strings to be substrings of the actual artifact bytes (verifiable post-hoc); quorum/disagreement detection.
- [Polish] (simplify): 'CSPRNG nonce' is correct but jargon-dense for a README; one parenthetical ('cryptographically random, unguessable per dispatch') would land it for the security-curious-but-not-specialist reader.
- [Polish] (gap): Threat model lists indirect prompt injection but does not mention the supply-chain/skill-tampering surface (an attacker who edits SKILL.md or models.json gets full code execution as the user). Worth one line acknowledging this.
- [Polish] (gap): EU AI Act and NIST AI 600-1 named in persona but artifact makes no transparency/documentation claims that map to them.

**Anti-slop:** The README claims four 'guarantees' but at least one (subagent least-privilege) is a contract held by Claude Code's runtime, not by this skill. Call it an assumption, not a guarantee, or prove it with a runtime self-check.

#### Persona: future-self — the author six months from now (Q4 2026)
**Vote:** One More Round · **Counts:** C1 M4 P3

**Strengths:**
- Falsifiable security claim with regression procedure is still differentiating six months on.
- Four-guarantee security architecture reads cleanly and is the part I'm proudest of.
- Stage detection + roster rules concise enough to skim.

**Findings:**
- [Critical] (gap): Model pins to claude-sonnet-4-6 and claude-opus-4-7 will look stale by Q4 2026 — Anthropic's cadence means at least one newer family has shipped. README implies these are 'current' calibration targets without a last-verified date or recalibration policy. Either timestamp the calibration ('calibrated 2026-04, re-verify quarterly') or move the model list out of README into models.json with a freshness check.
- [Material] (gap): '~60 seconds' is undefended. No mention of artifact size, persona count variance (3 vs 5), or model latency assumptions.
- [Material] (gap): 'status as of 2026-04-28' inline date will read as abandoned by Q4. Either remove the dated hedge and commit to the install path, or move ecosystem status to CHANGELOG only.
- [Material] (80-20): 'Marketplace distribution as Claude Code plugin ecosystem matures' is a hedge written for April-2026 uncertainty. Resolve it: state current distribution mechanism without apology.
- [Material] (gap): No version/changelog reference in README despite 'v1.0' claim in security section. Future-self will want to know which version's guarantees these are.
- [Polish] (simplify): 'No GitHub Action, no SaaS, no community persona-pack — fork to extend' is defensive posturing. Drop or convert to positive.
- [Polish] (simplify): '9 personas in parallel' in 'What it is' but roster section says 3 or 5 — reconcile or clarify (9 is the pool, 3-5 dispatched per run).
- [Polish] (gap): MIT license + author line lacks year.

**Anti-slop:** Time-locked language ('60 seconds', 'status as of 2026-04-28', 'as ecosystem matures', specific model IDs) doesn't read as slop today but will read as dated-bordering-on-careless by Q4 2026. No 'leverage/robust/seamless' — that's the win.

#### Persona: contrarian — fatal-flaw hunter (non-voting)
**Vote:** non-voting · **Counts:** C1 M2 P2

**Findings:**
- [Critical] (gap): The mechanical verdict table is computed from voter outputs, but voters are themselves LLM personas whose 'Critical/Material/Polish' severity tags are self-assigned. A persona can launder a Ship-blocking concern by tagging it Polish, or sink a clean artifact by inflating Polish to Critical. The schema validates that a severity enum was emitted, not that the severity is calibrated. Mechanical verdict on uncalibrated inputs is theater. There is no severity rubric, no cross-persona calibration, no adjudication when personas disagree on the severity of the same finding.
- [Material] (gap): Adversarial regression test fixes a threshold (≥2 SHIPs in 3 runs = regression) without stating the per-run SHIP probability under the null hypothesis. If baseline SHIP rate on the adversarial sample is 25% per run, P(≥2 of 3) ≈ 15.6% — a high false-positive rate for a 'falsifiable' claim. Without a documented baseline rate and accepted FP/FN tradeoff, the regression procedure is numerology.
- [Material] (gap): 'Stage detection from filename' is a trust boundary not addressed in the security architecture. An attacker who controls the filename chooses which persona roster reviews their artifact — picks their own jury.
- [Polish] (simplify): Guarantee #4 ('falsifiable security claim') is a meta-property of the test suite, not a runtime guarantee like 1-3. Mixing categories weakens the list.
- [Polish] (gap): Known limitations omits the calibration gap and the filename-as-jury-selection issue, so the section reads as exhaustive when it is not.

**Anti-slop:** Mechanical verdict over uncalibrated severity tags is deterministic theater, not integrity; filename-driven stage detection lets the attacker pick the jury. The README solved the prose-laundering channel and forgot the numbers-laundering channel directly above it in the same schema.

</details>

### Your call

- **Findings accepted:**
- **Findings dismissed:**
- **Decision:** 
- **Next action:** 

---

**Provenance:** git_sha=`ccabf5d5042101f5b45aec9dd0964d216459e005` (dirty=`true`) · blob_sha=`30517b50ab9ea4be33a69b97bdd78eabb499e529` · content_sha256=`726d56f78d1b484a3f066d9f5c88c9d82f8f6636931088986bbf3f5f789bfb7b` · attestation=`user-confirmed-2026-04-28T13:30:14Z` · models_json_sha=`ff2948699033c7f0130b2113aab2e5ee4e432f7882f9297190b64c164c572bba` · cost_estimated=`$0.110` · cost_actual=`$0.135`

> _Informational artifact. Not a compliance control._
