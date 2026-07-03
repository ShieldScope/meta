# ShieldScope Changelog

Mirrors [shieldscope.app/updates](https://shieldscope.app/updates).

Detection Methodology v1 · Started May 2026

---

## July 2026 — Example Library

**Module:** Sandbox  
**Type:** Usability · Education

Added a "Try an example" strip above the tab bar on /sandbox. Five one-click examples load pre-built sample artifacts and run the scan immediately — no file upload or copy-paste required.

- **Script — Base64 Payload**: JavaScript dropper using `eval(atob(...))` obfuscation, a cookie exfiltration beacon, and an encoded C2 config fetch. Fires obfuscation chain, eval, and base64 detectors.
- **File — Homoglyph Filename**: PDF loaded under the filename `rеsumе.pdf`, where both `е` characters are Cyrillic U+0435. Fires the homoglyph filename detector.
- **File — PDF with JavaScript**: Minimal PDF containing `/JavaScript` and `/OpenAction` markers. Fires the PDF JavaScript execution detector.
- **File — VBA Macro Document**: `.docm` file (OOXML ZIP) containing `word/vbaProject.bin` with valid OLE2 header. Fires the VBA macro stream detector.
- **File — Extension Mismatch**: Windows PE executable (MZ magic bytes) with a `.pdf` extension. Fires the extension mismatch detector at HIGH severity.

File examples are fetched from `/sandbox/examples/` and injected via the File API. No files leave the browser session; no data is stored.

---

## July 2026 — Scan Counter & Infrastructure

**Module:** Platform  
**Type:** Infrastructure · Transparency

Three infrastructure changes shipped:

**Scan counter.** A persistent scan counter now tracks total scans across all tools — /headers, /mail, /url, /exposure, and /sandbox (all endpoints). The current count appears as a fifth item in the trust bar on the homepage. Stored in a local SQLite database (`/var/www/shieldscope/data/stats.db`) via a thread-safe Python module (`backend/stats.py`). The counter is never tied to user identity or session data. Hidden when count is zero.

**Cache-Control headers.** Nginx now sends `Cache-Control: no-cache, must-revalidate` on all responses. Previously, HTML pages could be served from browser caches after a deploy. This ensures the current version propagates without a hard refresh.

**Content Security Policy update.** The `script-src` directive now allows `static.cloudflareinsights.com`, resolving a browser console error from the Cloudflare Web Analytics beacon. The scan counter script was moved from an inline `<script>` block to an external file (`/static/js/stats.js`) to comply with the existing `script-src 'self'` policy — no `unsafe-inline` was introduced.

---

## July 2026 — Deep-Linkable Finding Pages

**Module:** Sandbox  
**Type:** Documentation · SEO

Added four standalone reference pages for /sandbox file inspection findings. Each page covers: what the finding is, how ShieldScope detects it, the confidence model applied, and how to respond.

- `/sandbox/findings/pdf-javascript` — HIGH · HEURISTIC
- `/sandbox/findings/extension-mismatch` — HIGH/MEDIUM/LOW (tiered by detected type)
- `/sandbox/findings/homoglyph-filename` — HIGH · OBSERVED
- `/sandbox/findings/vba-macro-stream` — MEDIUM/HIGH · OBSERVED

All four added to `sitemap.xml` (priority 0.6, changefreq monthly).

---

## July 2026 — Finding Card Redesign

**Module:** Sandbox  
**Type:** Transparency · Accuracy

Redesigned finding cards across the /sandbox Script, Email, and File tabs to make the analytical chain explicit.

Each card now presents four distinct zones:

- **Verdict** — severity badge and confidence tier shown together in the header, so risk level and analytical certainty are readable before any explanation
- **Analysis** — the finding description: what was detected and why it matters
- **Evidence** — the raw extracted artifact (matched string, structural indicator, or pattern) shown as a labeled code block, separate from the conclusion, so analysts can verify the basis of the finding directly
- **Confidence note** — plain-language definition of the confidence tier, inline with every finding card

The confidence tier label alone was not sufficient for an analyst to calibrate response. The inline note now declares what each tier means operationally: OBSERVED findings include "Directly visible in retrieved data. No inference required." HEURISTIC findings include "Pattern-based — false positives possible. Verify before acting."

Two previously-defined confidence tiers are now surfaced in the UI: DERIVED (calculated deterministically from observed data) and INDETERMINATE (evidence retrieved, no stable conclusion possible). Both were part of the formal taxonomy but were not previously rendered in finding cards.

---

## May 2026 — Static File Inspection

**Module:** Sandbox  
**Type:** Detection

Added the File tab to /sandbox. Performs static inspection of uploaded files without executing them.

Detection coverage includes:
- Extension/MIME mismatch (executable or archive content with benign extension)
- Double extension patterns (e.g., `invoice.pdf.exe`)
- Embedded JavaScript in PDF files
- PDF launch actions and embedded file objects
- Office macro indicators (VBA macro streams in OLE2 containers)
- Compressed archive inspection: executable, script, and macro payloads in ZIP files
- Nested ZIP detection
- Homoglyph filenames (lookalike Unicode characters in the filename)

Risk verdict: CLEAN (confirmed benign MIME + no findings), NEUTRAL (unknown or archive type + no findings), or LOW/MEDIUM/HIGH based on weighted findings.

Limits: 10 MB file size, 100 ZIP entries, 50 MB uncompressed. No recursive archive extraction in V1.

---

## May 2026 — Email Confidence Classification

**Module:** Sandbox (Email tab)  
**Type:** Detection · Accuracy

Introduced a per-finding confidence classification to the email artifact analyzer. Each finding now carries one of two labels:

- **OBSERVED** — the indicator is structurally present in the artifact (e.g., a domain is literally in the header)
- **HEURISTIC** — the finding is inferred from a pattern that correlates with malicious behavior but is not definitively confirmed

This distinction is published in every finding card so analysts can calibrate their response accordingly. OBSERVED findings warrant direct action; HEURISTIC findings warrant investigation.

---

## May 2026 — Trust Layer

**Module:** Platform  
**Type:** Transparency · Infrastructure

Added the following supporting pages to establish platform transparency:

- **/about** — operator identity, analysis philosophy, what passive means
- **/security** — responsible disclosure policy, /.well-known/security.txt (RFC 9116)
- **/guide** — documentation for all tools including email artifact analysis

Updated /privacy to disclose Cloudflare Web Analytics (cookie-free, no personal data). Removed previously false claim that the platform collects no analytics. Added /terms with Acceptable Use Policy.

---

## May 2026 — Internet Exposure Analyzer

**Module:** Exposure  
**Type:** Detection

Launched /exposure: subdomain discovery and external attack surface mapping using Certificate Transparency log queries.

Checks per discovered host:
- Open port detection (common service ports)
- Service fingerprinting (HTTP/HTTPS banner, headers)
- TLS certificate validity and expiration
- Dangerous port exposure (RDP, SMB, databases)

All analysis is passive. No active probing, no banner grabbing beyond standard HTTP responses.

---

## May 2026 — Initial Launch

**Module:** Platform  
**Type:** Infrastructure

Initial public launch of ShieldScope with four tools live:

- /headers — HTTP security header analysis with graded output (A+ to F)
- /mail — SPF, DKIM, and DMARC authentication analysis
- /url — URL transparency and redirect chain analysis
- /password — Password strength check and HIBP breach exposure lookup (client-side k-anonymity)

All tools: passive analysis only, no active scanning, no account required, no data stored.
