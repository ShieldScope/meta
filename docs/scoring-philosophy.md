# Scoring Philosophy

This document describes how ShieldScope tools calculate risk scores and assign verdicts.

---

## General Model

Most ShieldScope tools use a weighted additive scoring model:

1. Each finding has a point weight determined by its severity
2. Weights accumulate additively across findings
3. The total score maps to a risk band (verdict)
4. The verdict is the user-facing output

Score is a signal, not a measure of compliance. A score of 0 does not mean an artifact is safe — it means the platform found nothing within its current detection scope.

---

## Severity Weights

Weights are calibrated per tool because the meaning of "high severity" differs across analysis contexts. General bands:

| Severity | Typical weight range |
|---|---|
| HIGH | 30–45 points |
| MEDIUM | 15–25 points |
| LOW | 5–12 points |
| INFO | 0 points |

Exact weights per finding are determined by the specificity and false positive rate of the indicator. A finding that almost never appears in benign content (e.g., PDF launch action) carries more weight than a finding that frequently appears in legitimate artifacts (e.g., a script using `eval`).

---

## Stacking Rules

Findings stack additively, with two constraints:

**1. No double-charging compound risk**  
If one finding already accounts for a gap, a second finding for the same gap does not add its full weight. Example: if a DMARC record is missing (high weight), the absence of DMARC enforcement (which requires DMARC to exist) does not stack as a separate finding.

**2. Context-dependent weighting**  
Some findings carry different weights depending on what else is present. Example (from /mail): SPF softfail carries lower weight when DMARC is enforcing, because DMARC enforcement compensates for SPF's transitional policy.

---

## Risk Bands

### /sandbox (File, Script, Email, URL tabs)

| Band | Score threshold | Meaning |
|---|---|---|
| CLEAN | 0 + confirmed benign MIME | No findings, and file type is confirmed non-executable/non-document |
| NEUTRAL | 0 + unknown or archive MIME | No findings, but file type is ambiguous or an archive |
| LOW | 1–24 | Minor indicators present |
| MEDIUM | 25–49 | Meaningful indicators present |
| HIGH | 50+ | Significant indicators present |
| CRITICAL | Withheld in V1 | Reserved pending calibration |

The CLEAN vs. NEUTRAL distinction reflects that "no findings" does not confirm safety for all file types. An unknown binary with no detectable patterns is not the same as a `.txt` file with no detectable patterns.

### /mail (Email Authentication Analyzer)

| Grade | Score | Meaning |
|---|---|---|
| A+ | 95–100 | Strong enforcement. SPF strict, DMARC enforcing, DKIM present. |
| A | 85–94 | Secure overall. Minor gaps. |
| B | 70–84 | Authentication configured. Enforcement improvements recommended. |
| C | 50–69 | Meaningful gaps. Monitoring-only DMARC or SPF softfail without backing. |
| D | 30–49 | Authentication unreliable or materially weakened. |
| F | 0–29 | Broken or dangerously misconfigured. |

Starting score is 100. Findings deduct from it. Grade reflects enforcement posture, not RFC compliance checklist completion. A domain that is operationally secure with minor RFC deviations should not receive a D.

### /headers (HTTP Security Header Analyzer)

Letter grade (A+ to F) based on presence, configuration, and strength of security headers. Each header contributes to or deducts from the overall grade based on its security value and current configuration state.

---

## What Scores Are Not

- **Not a compliance certification.** A HIGH verdict on a submitted file does not confirm it is malware. An A+ on /mail does not certify the domain against all spoofing scenarios.
- **Not a penetration test finding.** Scores reflect passive, static analysis of observable surface. They do not reflect exploitability, exposure to a specific threat actor, or runtime behavior.
- **Not comparable across tools.** A score of 40 in /sandbox and a score of 40 in /mail are calculated differently and describe different things. Do not compare scores across tools.

---

## Grade Philosophy (from /mail)

Grades reflect enforcement posture and authentication health, not compliance checklist completion.

Before finalizing a grade, check: does the grade match the narrative? If the posture summary sounds like a C but the score gives a D, verify — the narrative describes the security reality. Score arithmetic can produce unintuitive results when findings cluster at the boundary. The operational picture takes precedence.

Avoid grade inflation from stacking: multiple low-severity informational findings should not compound into a high-severity score. Each finding should carry the weight it deserves independently, and only stack when the risks genuinely compound.
