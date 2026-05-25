# ShieldScope Confidence Taxonomy

Version 1.0 · May 2026

---

## What Confidence Measures

Confidence in ShieldScope findings measures **evidentiary certainty**: how certain the platform is that a finding accurately reflects observable reality.

Confidence does NOT measure:
- Severity or impact
- Exploitability
- Business risk
- Breach probability
- Completeness of coverage

These are separate dimensions. A finding can be OBSERVED (high certainty) and LOW severity. A finding can be INFERRED (lower certainty) and HIGH severity. Those combinations are valid and must be expressible without contradiction.

The separation is intentional. Conflating confidence with severity produces tools that treat "uncertain high-risk" the same as "certain high-risk" — which causes both over-response and under-response in practice.

---

## The Five Tiers

### OBSERVED

**Definition:** The finding is directly visible in the retrieved data. The user can independently verify it from the displayed raw evidence without ShieldScope computation.

**Characteristics:**
- Deterministic and reproducible
- No processing, expansion, counting, or interpretation required
- The user sees exactly what ShieldScope sees

**Examples:**
- HTTP header is absent from the response
- DMARC policy is `p=none` (the value is in the record)
- TLS certificate expiration date
- Redirect destination URL
- `eval(unescape(...))` appears in a script
- File extension is `.exe`

**Exclusion:** If reaching the conclusion requires counting, parsing, normalizing, comparing, or expanding data, it is not OBSERVED — it is DERIVED.

---

### DERIVED

**Definition:** The finding is calculated deterministically from observed data. The raw evidence is observed; the conclusion requires ShieldScope's processing to reach.

**Reproducibility test:** The user could reproduce it, but would need to follow ShieldScope's computation — they cannot read the conclusion directly from the raw evidence.

**Characteristics:**
- Deterministic (same inputs always produce same output)
- Requires computation, expansion, normalization, parsing, or structured comparison
- High confidence, but conceptually distinct from direct observation

**Examples:**
- SPF DNS lookup count (requires tracing the include chain)
- CSP directive weakness classification (requires parsing and evaluating the directive tree)
- DMARC enforcement readiness (computed from policy + alignment + DKIM presence)
- TLS version support classification (normalized from negotiated cipher suites)
- File entropy score
- ZIP uncompressed size ratio

**Exclusion:** If the computation involves assumptions about behavior, intent, or incomplete data, it crosses into INFERRED.

---

### HEURISTIC

**Definition:** The finding is based on pattern matching against known indicators. The pattern correlates with the behavior — it does not confirm it.

**Characteristics:**
- Pattern-based, not deterministic
- False positives are expected and must be acknowledged per finding
- Confidence bounded by pattern specificity and false positive rate

**Distinction from INFERRED:** HEURISTIC uncertainty comes from pattern specificity — the pattern is real, but matches too broadly. INFERRED uncertainty comes from data completeness — the data is real, but the conclusion extends beyond what was verified.

**Examples:**
- Suspicious URL structure (homoglyph characters, misleading subdomain depth)
- High script entropy consistent with obfuscation
- Email sender display name inconsistent with sending domain
- Double extension filename consistent with masquerading
- Phishing-style anchor text mismatch

**Required disclosure:** Every HEURISTIC finding must declare its false positive surface. "This pattern is also present in [legitimate context]" is part of the finding, not a footnote.

---

### INFERRED

**Definition:** The finding is a contextual conclusion based on incomplete visibility. The evidence is real; the conclusion extends beyond what was directly verified.

**Characteristics:**
- Evidence is real (observed or derived)
- Conclusion requires an assumption about context, behavior, or scope not fully present in the retrieved data
- Common in passive analysis where full system state is not observable

**Examples:**
- Probable infrastructure role from response header patterns
- CDN attribution from IP range and certificate subject names
- Delivery failure risk from SPF lookup count exceeding RFC limits (the count is DERIVED; the delivery behavior is INFERRED)
- Exposure implication from open port combined with service banner
- Behavioral interpretation from partial artifact structure

---

### INDETERMINATE

**Definition:** ShieldScope retrieved data, but the evidence does not support a stable conclusion. Analysis ran and completed; the result is contradictory or unresolvable.

**Distinction from retrieval failure:** INDETERMINATE means analysis produced ambiguous results. Retrieval failure (timeout, unreachable endpoint) is a scan state — no analysis occurred, so no confidence label applies.

**Characteristics:**
- Data was retrieved successfully
- Multiple valid interpretations exist with no basis for preferring one
- Publishing a finding with a directional confidence label would be misleading

**Examples:**
- Conflicting DNS answers from multiple resolvers
- Malformed but partially parseable DNS record
- CDN or shared certificate with multiple subject names that do not resolve the ownership question
- Redirect chain inconsistency (A → B → A, or conflicting indicators at the destination)
- Multiple SPF records present (RFC-violating; receiver behavior is undefined)

**Usage rule:** INDETERMINATE findings should be published when the ambiguity itself is operationally significant — multiple SPF records is a misconfiguration worth surfacing regardless of whether a verdict can be reached. They should not be published when the ambiguity would only confuse without informing.

---

## Retrieval Failure States

Retrieval failures are scan states, not confidence tiers. When no analysis occurs, no confidence label applies.

| State | Meaning |
|---|---|
| `not_checked` | Check was not attempted (outside current scope, or conditionally skipped) |
| `unreachable` | Target did not respond |
| `timeout` | Response did not complete within the allowed window |
| `unsupported` | Target type or format is not handled by this check |
| `rate_limited` | Upstream source returned a rate limit response |
| `blocked` | Request was refused by the target or an intermediary |

These states appear in finding metadata. They do not produce a confidence label, a severity, or a score contribution.

---

## Multi-Layer Confidence

Some findings involve multiple reasoning steps, each with its own confidence tier.

**Weakest-link rule:** The headline confidence is the weakest tier in the chain. Not an average. The weakest necessary reasoning step governs what the user sees as the primary confidence signal.

**Audit trail:** The card shows both the headline confidence and the reasoning chain so the user can see where uncertainty was introduced.

```
Headline: INFERRED
Chain:    OBSERVED → DERIVED → INFERRED
```

**Full example:**

Finding: "SPF configuration may cause delivery failures for legitimate senders"

| Step | What happened | Confidence |
|---|---|---|
| SPF record retrieval | Record is present and directly readable | OBSERVED |
| Lookup count expansion | Count computed by tracing the include chain | DERIVED |
| Delivery failure risk | Failure depends on receiver behavior, not fully in scope | INFERRED |
| **Headline** | | **INFERRED** |

This approach prevents hiding the weakest step behind a stronger label while still showing the full reasoning path.

---

## What Confidence Is Not

| Not this | Use instead |
|---|---|
| A numeric certainty percentage | A tier describing evidence origin and verification model |
| A proxy for severity | Severity is a separate, independent dimension |
| A measure of exploitability | Not in scope for this taxonomy |
| A color code | Color may accompany the label but does not replace it — textual semantics are primary |
| A single scalar for multi-step findings | Use the multi-layer model with the weakest-link rule |

---

## Migration Note

ShieldScope currently publishes findings with two confidence labels: OBSERVED and HEURISTIC (introduced May 2026 in the /sandbox Email and File tabs).

This taxonomy supersedes that model.

**Migration path:**
- Existing OBSERVED findings: remain OBSERVED
- Existing HEURISTIC findings: audit for reclassification — some may be INFERRED (contextual conclusions from partial visibility) rather than HEURISTIC (pattern matching against known indicators)
- DERIVED, INFERRED, INDETERMINATE: new tiers, applied on first use per tool
- Retrieval failure states: applied retroactively where applicable, replacing prior `unknown` usage where the distinction is meaningful
