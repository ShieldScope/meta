# ShieldScope Changelog

Mirrors [shieldscope.app/updates](https://shieldscope.app/updates).

Detection Methodology v1 · Started May 2026

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
