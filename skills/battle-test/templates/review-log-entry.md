
## Review — {{datetime}} — stage:{{stage}} tier:{{tier}}

**Artifact:** [{{artifact_path}}]({{artifact_path}}) ({{artifact_size}})

**Reviewed:** {{artifact_summary}}

**Personas:** {{personas_list}}
**Verdict:** **{{verdict}}**
**Findings:** Critical: {{critical_count}} · Material: {{material_count}} · Polish: {{polish_count}}

### Headline

**Where the council agrees:**
{{agreement_synthesis}}

**Where the council clashes (with resolution):**
{{clash_synthesis}}

**Blind spots:**
{{blind_spots}}

**Recommendation:**
{{recommendation}}

### Per-persona reports

<details>
<summary>Click to expand full persona findings</summary>

{{per_persona_blocks}}

</details>

### Your call

_Filled in by the artifact owner after reading. Format:_

- **Findings accepted:** _(list which findings you'll fix; reference by severity + persona)_
- **Findings dismissed:** _(list which findings you're rejecting and one-line reason — these will not resurface in future runs)_
- **Decision:** ship now / revise this iteration / structural rework
- **Next action:** _(what you do next — e.g., "fix the two Material findings, re-run /battle-test, then publish")_

---

**Provenance:** git_sha=`{{git_sha}}` (dirty=`{{dirty}}`) · blob_sha=`{{blob_sha}}` · content_sha256=`{{content_sha256}}` · attestation=`{{attestation}}` · models_json_sha=`{{models_json_sha}}` · cost_estimated=`{{cost_estimated}}` · cost_actual=`{{cost_actual}}`

> _Informational artifact. Not a compliance control._

---
