# Confidence Taxonomy

ShieldScope uses a two-tier confidence classification applied per finding. This document defines what each tier means and how to use it.

---

## The Two Tiers

### OBSERVED

The indicator is structurally present in the artifact.

OBSERVED means: the finding is based on something directly readable in the submitted content. The platform did not infer or extrapolate — it found the thing.

Examples:
- A PDF file contains an embedded JavaScript stream (`/JavaScript` object in the PDF structure)
- An email header contains a `From:` domain that does not match the `Return-Path:` domain
- A URL contains a homoglyph character (a Unicode lookalike substitution is present in the hostname)
- A script contains a literal `eval(unescape(...))` call

OBSERVED findings warrant direct action. If the indicator is present, it is present.

---

### HEURISTIC

The finding is inferred from a pattern that correlates with malicious behavior, but is not structurally confirmed.

HEURISTIC means: the platform detected a pattern associated with the behavior, not the behavior itself. The correlation is meaningful; the confidence is bounded.

Examples:
- A filename contains a double extension (`.pdf.exe`) — consistent with masquerading, but double extensions exist in legitimate software packaging
- A script uses heavily chained string operations — consistent with obfuscation, but also present in minified legitimate code
- An email sender's display name contains a company name not matching the sending domain — consistent with spoofing, but also present in forwarded or alias mail

HEURISTIC findings warrant investigation, not immediate conclusion. The pattern is a signal. Context determines whether it is a threat.

---

## Why This Distinction Matters

Without confidence labels, a finding list conflates two different types of claims:
1. "This thing is present" — a structural fact
2. "This pattern suggests a thing may be present" — an inference

Conflating them causes two failure modes:
- **Over-response:** Treating a heuristic pattern as confirmed leads to false positives being acted on as threats
- **Under-response:** When analysts learn that some findings are uncertain, they sometimes discount the entire list — including OBSERVED findings that are certain

The confidence label is per-finding so each item is interpreted correctly. An OBSERVED finding next to a HEURISTIC finding is not the same type of claim, and the label makes that explicit.

---

## Confidence and Severity Are Independent

A finding can be any combination:

| Confidence | Severity | Example |
|---|---|---|
| OBSERVED | HIGH | PDF with embedded launch action — structurally present, high risk |
| OBSERVED | LOW | Executable file with a common extension — structural fact, low additional signal |
| HEURISTIC | HIGH | Double-extension filename on what appears to be an executable — high-risk pattern, not confirmed |
| HEURISTIC | LOW | Minor header anomaly in an email consistent with forwarding — low signal, benign explanation likely |

Severity describes potential impact. Confidence describes how certain the platform is that the indicator exists. Both matter. Neither overrides the other.

---

## Scope of Application

The OBSERVED / HEURISTIC taxonomy currently applies to:
- /sandbox Email tab (per-finding)
- /sandbox File tab (per-finding)
- /sandbox Script tab (per-finding)

It is being extended to all tools in subsequent releases. Tools currently without per-finding confidence labels declare their overall scope in [KNOWN-LIMITATIONS.md](../KNOWN-LIMITATIONS.md).
