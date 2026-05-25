# ShieldScope Roadmap

This document reflects current development priorities. It is not a commitment or release schedule — it signals direction.

Last updated: May 2026

---

## Currently Live

All six tools are live at [shieldscope.app](https://shieldscope.app).

| Tool | Status |
|---|---|
| /headers — HTTP Security Header Analyzer | Live |
| /mail — Email Authentication Analyzer | Live |
| /url — URL Transparency Analyzer | Live |
| /password — Password Strength & Breach Exposure | Live |
| /exposure — Internet Exposure Analyzer | Live |
| /sandbox — Suspicious Artifact Analyzer (URL, Script, Email, File) | Live |

Supporting pages live: /guide, /updates, /about, /security, /privacy, /terms, /contact

---

## Near-Term

**Confidence taxonomy documentation**  
Publish a formal definition of the OBSERVED vs. HEURISTIC confidence classification used across /sandbox and other tools. Currently signaled per-finding in the UI; this makes the model explicit and auditable.

**Scoring philosophy documentation**  
How risk scores are calculated: weight model, band definitions (CLEAN / NEUTRAL / LOW / MEDIUM / HIGH), rationale for thresholds.

**Methodology page (/methodology)**  
A dedicated page explaining the static-only analysis approach, what the tools can and cannot verify, and how to interpret findings correctly.

**Finding card redesign**  
Improve the visual hierarchy of finding output: clearer confidence signal placement, "why this matters" in context, better separation of verdict from detail.

---

## Medium-Term

**Finding detail pages**  
Deep-linkable reference pages per finding type:
- /sandbox/findings/pdf-javascript
- /sandbox/findings/extension-mismatch
- /sandbox/findings/homoglyph-filename
- /sandbox/findings/vba-macro-stream

Each page covers: what the finding is, why it matters, how to respond. Useful to practitioners investigating specific attack patterns.

**Example library**  
Curated safe samples users can load directly into sandbox tools: homoglyph filename, macro-enabled Office document, nested archive, deceptive PDF, spoofed email header, malicious anchor mismatch. Educational and interactive.

**Trust signals**  
- Public uptime status page
- Scan activity counter (if instrumented)
- Methodology version history (started at [/updates](https://shieldscope.app/updates))

---

## Deferred / Under Consideration

- CRITICAL risk band for /sandbox (withheld in V1 pending calibration)
- Score calibration pass for heuristic stacking in edge cases
- Confidence label tooltips in the UI
- Individual update deep-link pages (/updates/static-file-inspection, etc.)
- Module card hierarchy on the homepage (most used, recommended)

---

## Not Planned

- Active scanning (port scanning, web crawling, content injection)
- User accounts or persistent scan history
- Public API
- Open-sourcing the backend detection engine
