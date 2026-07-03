# ShieldScope Roadmap

This document reflects current development priorities. It is not a commitment or release schedule — it signals direction.

Last updated: July 2026

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

## Completed (July 2026)

- ✅ Confidence taxonomy documentation — 5-tier model, published at github.com/ShieldScope/meta
- ✅ Scoring philosophy documentation — posture degradation model, published at github.com/ShieldScope/meta
- ✅ Methodology page — `/methodology` live, links to GitHub canonical docs
- ✅ Finding card redesign — 4-zone layout (verdict / analysis / evidence / confidence note) across Script, Email, File tabs
- ✅ Finding detail pages — `/sandbox/findings/{pdf-javascript, extension-mismatch, homoglyph-filename, vba-macro-stream}` live
- ✅ Example library — 5 one-click examples above tab bar on /sandbox
- ✅ Scan counter — trust bar item tracking total scans via SQLite, populated by `/api/stats`

---

## Near-Term

**More finding detail pages**  
Expand beyond the initial 4 file-inspection pages. Script and email tab findings are candidates:
- /sandbox/findings/eval-obfuscation
- /sandbox/findings/base64-payload
- /sandbox/findings/spoofed-display-name
- /sandbox/findings/missing-authentication

**Score calibration**  
Heuristic findings can stack in edge cases to produce misleading scores. A calibration pass is needed before the CRITICAL risk band is considered.

**OOXML macro detection improvement**  
Current `.docx`/`.xlsx` macro detection is extension-based only (V1). Full inspection of `vbaProject.bin` inside OOXML ZIP containers should be parity with OLE2 detection.

**Trust signals**  
- Public uptime status page (pending UptimeRobot account)
- Methodology version history on /updates

---

## Deferred / Under Consideration

- CRITICAL risk band for /sandbox (withheld pending score calibration)
- Confidence label tooltips in the UI
- Individual update deep-link pages (/updates/static-file-inspection, etc.)
- Module card hierarchy on the homepage (most used, recommended)
- /url enhancement: visual URL decomposition, redirect storytelling, tracking/privacy detection

---

## Not Planned

- Active scanning (port scanning, web crawling, content injection)
- User accounts or persistent scan history
- Public API
- Open-sourcing the backend detection engine
