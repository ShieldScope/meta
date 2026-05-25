# Methodology Overview

This document describes the analysis approach used across ShieldScope tools.

---

## Passive and Static Analysis

ShieldScope tools perform passive, static analysis only.

**Passive** means the platform does not initiate active probing, scanning, or exploitation of submitted targets. For network-facing tools (/url, /exposure), the platform makes standard DNS and HTTP requests to resolve targets — it does not scan ports, inject payloads, or interact with services beyond reading public responses.

**Static** means submitted artifacts (files, scripts, emails, URLs) are analyzed by structure and observable content. Nothing is executed, rendered, or run. A file that would behave maliciously when opened is evaluated on its structural indicators, not its runtime behavior.

This scope is intentional. It means:
- Analysis is safe to perform on potentially malicious artifacts
- Results do not require execution context or dynamic instrumentation
- The platform can be used without risk to the analyst's environment

It also means:
- Runtime behavior is not observed
- Obfuscated payloads that only reveal themselves at execution time are detected on observable obfuscation patterns, not decoded content
- Novel techniques outside current detection scope will not be flagged

Both of these are true simultaneously. The scope is declared because the alternative — not declaring it — creates false confidence.

---

## Finding Classification

Every finding has two attributes: **confidence** and **severity**.

**Confidence** describes how the finding was determined. See [confidence-taxonomy.md](confidence-taxonomy.md) for the full definition.

**Severity** describes the potential impact of the finding:

| Severity | Meaning |
|---|---|
| HIGH | Structurally significant indicator. Warrants direct investigation. |
| MEDIUM | Meaningful pattern with notable false positive rate. Investigate in context. |
| LOW | Weak signal or best-practice deviation. Note and monitor. |
| INFO | Observation with no security implication. Context only. |

Severity reflects potential impact, not certainty. A HIGH-severity heuristic finding is a high-impact pattern — not a confirmed threat.

---

## Risk Verdicts

Tools that produce a summary verdict use the following bands:

| Verdict | Meaning |
|---|---|
| CLEAN | Confirmed benign. Requires: known-benign MIME type and no findings. |
| NEUTRAL | No findings, but the artifact type or content is not confirmed benign (e.g., unknown file type, archive). |
| LOW | Findings present with low aggregate weight. |
| MEDIUM | Findings present with moderate aggregate weight. |
| HIGH | Findings present with significant aggregate weight. |
| CRITICAL | Reserved. Not published in V1 pending calibration. |

CLEAN and NEUTRAL are not the same verdict. NEUTRAL does not mean safe — it means the platform found nothing and cannot confirm safety. The distinction matters for unknown or opaque artifact types.

---

## Score Calculation

Risk scores are additive across findings, weighted by severity. See [scoring-philosophy.md](scoring-philosophy.md) for weights and band thresholds per tool.

General principles:
- A finding should only contribute its weight once, even if multiple observations support it
- Compound risk is not charged twice: if a high-severity finding already accounts for a gap, a lower-severity finding for the same gap should not stack
- Heuristic findings contribute their weight; the confidence label informs interpretation, not the score itself
- Score is an operational signal, not a compliance checklist result

---

## What This Platform Is For

ShieldScope is designed for IT professionals and security practitioners assessing their own or their organization's security posture. Tools are calibrated for this use case:

- Results assume the analyst can interpret confidence and severity correctly
- Findings declare their scope and limitations inline
- The platform does not produce pass/fail verdicts implying complete coverage

It is not a replacement for a penetration test, a full SIEM, or dynamic malware analysis. It is a fast, passive, no-account-required tool for surface-level security posture checks.
