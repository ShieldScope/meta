# ShieldScope Scoring Philosophy

Version 1.0 · May 2026

---

## Anchor

**Score measures posture degradation, not breach probability.**

This sentence governs every decision in this document.

Scoring answers: *how much does the observed posture reduce resilience, authentication strength, trust clarity, or safe handling?*

It does not answer: *will this domain get breached? Is this file malware? Is this sender malicious?*

That distinction is not a hedge. It is an accuracy constraint. ShieldScope performs passive, static analysis with bounded visibility. Claiming breach probability from that evidence base would be false precision. Posture degradation is what the evidence can support.

---

## What the Score Represents

A ShieldScope score or risk band represents the cumulative weight of observed posture weaknesses within the tool's scope, as of the time of analysis.

A lower score (or higher risk band) means: more significant posture weaknesses were observed.  
A higher score (or lower risk band) means: fewer or less significant posture weaknesses were observed.

That is all it means.

---

## What the Score Does Not Represent

- **Breach probability.** Score is not a likelihood estimate of a security incident.
- **Overall security posture.** Each tool covers one surface. A clean result on one tool does not compensate for findings on another.
- **Complete coverage.** A clean passive scan means "nothing significant was observed within this tool's scope." It does not mean the surface is secure.
- **Runtime behavior.** For artifact analysis tools, score reflects structural indicators only. A file scoring LOW may behave maliciously when executed. A file scoring HIGH may be a false positive.
- **Mathematical certainty.** A score of 72 is not twice as safe as 36. It means fewer or less significant posture weaknesses were observed. Scores are directional, not absolute.

---

## Relationship Between Score, Severity, and Confidence

Severity and confidence are independent dimensions. Score is derived from severity. Confidence affects how a finding is explained — it does not automatically adjust the severity weight.

| Scenario | Severity | Confidence | Effect on score |
|---|---|---|---|
| Missing HSTS header | LOW–MEDIUM | OBSERVED | Contributes its severity weight |
| SPF delivery failure risk | HIGH | INFERRED | Contributes its severity weight |
| Phishing-style URL pattern | HIGH | HEURISTIC | Contributes its severity weight |
| Malformed DNS record | — | INDETERMINATE | Does not contribute to score |

A HEURISTIC finding does not become lower severity because it is uncertain. A LOW-severity finding does not score higher because it was directly observed. The confidence tier determines how the finding is explained and caveated — not what it contributes to the score.

**The one exception:** INDETERMINATE findings do not contribute to the score. When analysis completed but produced no stable conclusion, there is no finding to score. INDETERMINATE findings may affect the overall verdict (e.g., preventing a CLEAN result) but do not add deduction weight.

Every finding that contributes to the score must be traceable. If a user asks why the score is what it is, the UI and documentation should point to specific findings and their individual contributions. A score that cannot be explained from its findings is not defensible.

---

## Deduction Philosophy

**Start from a baseline.** Tools that use graded output (A+ to F, or 0–100) begin at the maximum and deduct. Tools that use risk bands begin at NEUTRAL and accumulate.

**Deduct for posture degradation, not for deviation from best practice.** A configuration that materially reduces resilience, authentication coverage, or safe handling is a deduction. A configuration that is non-ideal but not harmful is a note, not a deduction.

**Do not double-charge compound risk.** If one finding already accounts for a gap, a second finding for the same underlying gap does not contribute its full weight. Compound risk — where two findings together create a worse condition than either alone — is different and may stack, but the rationale must be explicit.

**Context changes weight.** Some findings carry different weight depending on what else is present. A deduction that fills the only gap in protection weighs more than the same finding when another mechanism compensates.

**Absence penalties require meaningful absence.** If ShieldScope cannot observe something, it should not penalize heavily unless the absence itself is meaningful. Absence of DKIM detection in a bounded selector check is not the same as confirmed absence of DKIM. The score should reflect what was confirmed, not what was not found.

**Floor is zero.** Scores do not go negative. Multiple severe findings compound to a floor of zero, not below.

**Informational findings do not deduct.** INFO-severity findings contribute zero to the score. They exist to provide context.

---

## Severity Definitions

Severity describes the magnitude of posture degradation a finding represents, independent of confidence.

| Severity | Definition | Score contribution |
|---|---|---|
| CRITICAL | Immediate abuse potential, severe trust failure, or dangerous handling condition. Authentication is broken or trivially bypassed. Active exploitation risk exists without additional prerequisites. | Reserved. Not published in V1. |
| HIGH | Finding materially reduces resilience or eliminates a layer of protection. By itself, it significantly degrades posture. | Substantial deduction |
| MEDIUM | Finding creates a meaningful gap that reduces posture in realistic scenarios. Mitigating factors may reduce impact. | Moderate deduction |
| LOW | Best practice deviation with limited practical posture impact. Worth noting; not structurally significant. | Minor deduction |
| INFO | Observation, confirmation, or context. No posture degradation. | Zero |

**On CRITICAL:** CRITICAL is not a label for "serious" or "concerning." It is reserved for findings where the posture failure is severe enough that the surface cannot be considered operationally trustworthy without remediation. Every CRITICAL finding published must meet this bar. The category being rare makes it meaningful.

---

## Module-Specific Scoring Notes

### /headers

Scoring reflects HTTP security header presence, configuration, and strength. Output is a letter grade (A+ to F).

Starting baseline: 100. Deductions applied per finding.

- Header completely absent: deduction scales with the header's role. Missing HSTS on a public site is more significant than missing Permissions-Policy.
- Header present but misconfigured: deduction is typically smaller than absent, but not zero — a misconfigured header can be worse than an absent one if it creates a false sense of protection.
- Header present and correct: no deduction.
- CSP: directive-level analysis. `unsafe-inline` on `script-src` is a higher-severity deduction than `unsafe-inline` on `style-src`. These are not equivalent.

Grade reflects posture across all headers collectively. Avoid grade inflation from stacking: many low-severity findings should not compound to a failing grade if the overall posture is defensible.

### /mail

Scoring reflects email authentication posture across SPF, DKIM, and DMARC. Output is a letter grade (A+ to F).

Starting baseline: 100. Deductions applied per finding.

Contextual weights apply:
- SPF softfail (`~all`) weight depends on what backs it: lower deduction if DMARC is enforcing (DMARC compensates), higher deduction if DMARC is absent (SPF is the only mechanism).
- DMARC `p=none` is an operational choice, not a negligent one — appropriate during initial deployment. Deduction reflects monitoring-only posture, not misconfiguration.
- DKIM absence is bounded: the selector check covered N selectors. Finding weight is calibrated accordingly. Do not apply a "DKIM not configured" penalty equivalent to confirmed absence.

Grade reflects enforcement posture and authentication health, not RFC compliance checklist completion. A domain can be operationally secure with minor RFC deviations and should not receive a failing grade for them.

### /url

Scoring reflects URL transparency and observable risk indicators. Output is a risk band (CLEAN / NEUTRAL / LOW / MEDIUM / HIGH).

Additive from findings. Each finding contributes its weight to the total.

CLEAN requires: no findings and the URL structure presents no anomalies. NEUTRAL is the default for unresolved or opaque URLs with no findings. LOW through HIGH reflect cumulative finding weight.

HEURISTIC findings (phishing pattern, typosquatting suspicion) contribute their full weight but must declare their false positive surface in the finding card.

### /password

Two independent signals: strength band and breach exposure. These are not combined into a single score.

Breach exposure: OBSERVED from HIBP. A count above zero is factual. The implication for password reuse is a recommendation, not a scored finding.

Strength band: DERIVED from entropy computation. The band (Weak / Fair / Strong / Very Strong) is a posture indicator — it does not factor in targeted attacks, credential stuffing lists, or contextual guessing.

No single combined score is produced. Presenting breach count and strength separately preserves interpretability.

### /exposure

Scoring reflects external attack surface observability. Output is per-host risk assessment plus an aggregate surface summary.

Findings stack per host. Dangerous port exposure compounds with service fingerprinting. An open RDP port is more significant when the host shows no indication of being a jump server.

INFERRED findings (infrastructure role, probable purpose) contribute their weight but should be declared as inferred in any explanation. Do not present infrastructure role conclusions as confirmed.

Subdomain count is not itself a scored finding. A large number of subdomains is informational. Risk comes from what those subdomains expose, not from their quantity.

### /sandbox (URL, Script, Email, File tabs)

Scoring uses additive weight from findings. Output is a risk band (CLEAN / NEUTRAL / LOW / MEDIUM / HIGH).

**CLEAN vs. NEUTRAL distinction:** CLEAN requires a confirmed-benign artifact type plus no findings. NEUTRAL means no findings but the artifact type is ambiguous or cannot be confirmed benign. This distinction is intentional and must not be collapsed — "no findings on an unknown binary" is not the same as "no findings on a plain text file."

**CRITICAL is withheld in V1** pending calibration. The maximum published verdict is HIGH.

Heuristic findings contribute their full weight. The finding card must declare the false positive surface. The score does not auto-reduce for HEURISTIC confidence.

INDETERMINATE findings (rare in artifact analysis) do not contribute to score. They may prevent a CLEAN verdict.

---

## Edge Cases and Non-Penalties

These situations do not warrant deductions, regardless of how they appear.

**Intentional operational choices in recognized deployment phases.** DMARC `p=none` during initial rollout, SPF `~all` during migration, relaxed DKIM alignment as the recommended default — these are not negligence. The finding should acknowledge the intent.

**Absence of optional mechanisms with no clear coverage gap.** Features that are optional, widely skipped for legitimate reasons, and whose absence does not create a meaningful gap are informational at most.

**Tool scope limitations.** If ShieldScope did not check something, it does not penalize for it. A selector not in the checked set is not a finding.

**Retrieval failures.** If a check could not complete due to timeout, rate limiting, or unreachable endpoint, no deduction applies. The scan state is surfaced; no finding is published.

**High score does not mean "secure."** Conversely, a finding that scores LOW is not trivial — it represents an observed posture weakness. Score level is not a proxy for whether remediation is warranted.

---

## Example Interpretation

**Scenario:** /mail returns a score of 61 (grade C).

The findings contributing to this score might be:
- DMARC policy is `p=none` — MEDIUM: monitoring posture only, no enforcement (-12)
- SPF is `~all` with no DMARC enforcement — HIGH: softfail carries full weight without DMARC backing (-18)
- No DKIM selectors detected in checked set — MEDIUM: bounded check, but absence reduces authentication layering (-9)

Total deduction: 39. Score: 61.

**What this means:** The domain's email authentication posture shows meaningful gaps. SPF softfail without DMARC backing means authentication enforcement is absent for unauthenticated senders. DMARC in monitoring mode provides visibility but not protection.

**What this does not mean:** This domain is being actively spoofed. This domain will experience a breach. DKIM is definitely not configured.

The score explains the posture. It does not predict the outcome.
