# Known Limitations

ShieldScope performs passive, static analysis only. This document describes the scope boundaries and known gaps for each tool. Understanding these limits is part of using the platform correctly.

Last updated: May 2026

---

## Platform-Wide

**No execution.** Nothing submitted to ShieldScope is executed, rendered, or run. Analysis is purely structural and pattern-based. A file that would behave maliciously when opened is analyzed by its structure, not its behavior.

**No active scanning.** ShieldScope does not initiate connections to submitted targets, crawl pages, scan ports, or interact with services beyond standard DNS and HTTP resolution. The /url and /exposure tools make outbound requests to resolve submitted targets, but do not probe or interact further.

**No persistent storage.** Submitted targets are processed in memory and not stored. Results are not retained, logged, or retrievable after the session.

**False positives are possible.** Heuristic detections are pattern-based. A finding marked HEURISTIC means the pattern correlates with malicious behavior — it does not confirm it. Treat heuristic findings as prompts for investigation, not verdicts.

**False negatives are possible.** These tools check known patterns and observable structure. Novel obfuscation, zero-day techniques, or indicators outside the current detection scope will not be flagged.

---

## /headers — HTTP Security Header Analyzer

- Analysis reflects the HTTP response at the time of the scan. Headers may differ across endpoints, subdomains, or behind authentication.
- CDN and load balancer layers may inject or strip headers not representative of the origin server.
- Content-Security-Policy analysis evaluates common unsafe directives. Complex or custom CSP policies may not be fully characterized.
- HSTS preload status is not checked against the Chrome/Firefox preload list — the header is evaluated in isolation.

---

## /mail — Email Authentication Analyzer

- **DKIM:** Checks a bounded set of common selectors (e.g., `default`, `google`, `selector1`, `selector2`, `mail`, `dkim`). Custom or provider-specific selectors outside this set will not be detected. Absence of a DKIM finding does not confirm DKIM is unconfigured.
- **SPF:** Analyzes the published SPF record via DNS. Third-party sender detection is based on known `include:` patterns and is not exhaustive.
- **DMARC:** Reflects current DNS-published policy. Organizational domain DMARC lookup is performed for subdomains without their own record.
- All results reflect current DNS state. DNS propagation delays may cause stale readings immediately after a configuration change.
- Results describe authentication posture for the submitted domain. They do not predict receiver behavior — receiving systems apply their own filtering policies.

---

## /url — URL Transparency Analyzer

- Follows redirect chains up to a fixed depth. Chains longer than this limit are truncated.
- Does not render JavaScript or execute client-side redirects. Meta-refresh and JS-based redirections will not be followed.
- Passive reputation signals (domain age, registrar, hosting ASN) are indicators, not verdicts. Newly registered domains used legitimately will trigger the same signals as newly registered phishing domains.
- Does not dereference shortened URLs to their final destination beyond what HTTP redirects expose.

---

## /password — Password Strength & Breach Exposure

- **Breach check:** Uses the Have I Been Pwned k-anonymity API. The first 5 characters of the SHA-1 hash are sent to HIBP — the full password never leaves the browser. Results reflect the HIBP database at query time and are not comprehensive of all breaches.
- **Strength analysis:** Entropy-based scoring. Does not check password lists, dictionary attacks, or contextual guessing (e.g., using the target's name). A high entropy score does not guarantee resistance to targeted attacks.
- Entirely client-side. Nothing is submitted to ShieldScope servers.

---

## /exposure — Internet Exposure Analyzer

- Subdomain discovery uses Certificate Transparency logs (via HackerTarget). Coverage is limited to subdomains with publicly issued TLS certificates. Subdomains without certificates, or with certificates not logged to public CT logs, will not appear.
- Port detection is limited to a defined set of common service ports. Non-standard ports are not checked.
- Service fingerprinting is based on HTTP/HTTPS response headers. Services that do not respond to standard HTTP probes are not fingerprinted.
- TLS analysis reflects the certificate chain at query time. Certificate pinning and HPKP are not evaluated.
- Results describe what is externally visible from public infrastructure. Internal or VPN-only services are not in scope.

---

## /sandbox — Suspicious Artifact Analyzer

### URL tab
- Pattern-based analysis of the URL structure and observable properties. Does not dereference or visit the URL.
- Homoglyph detection covers common Unicode substitution patterns. Novel or compound lookalike encodings may not be detected.

### Script tab
- Static pattern matching on submitted script content. Does not execute or interpret the script.
- Obfuscated code that evaluates to malicious content at runtime will be flagged on observable obfuscation patterns (e.g., `eval`, `unescape`, encoded strings), not on the decoded payload.
- Coverage is optimized for JavaScript and PowerShell. Other script types may produce partial results.

### Email tab
- Analyzes submitted email headers and body structure. Does not fetch linked content or resolve embedded URLs.
- DKIM signature verification is not performed — header presence and structure are evaluated, not cryptographic validity.
- Confidence classification (OBSERVED / HEURISTIC) is per-finding. See [docs/confidence-taxonomy.md](docs/confidence-taxonomy.md).

### File tab
- **Size limit:** 10 MB. Files larger than this cannot be processed.
- **ZIP limit:** 100 entries maximum, 50 MB uncompressed total. Archives exceeding these limits are partially analyzed.
- **No recursive extraction:** Nested archives are detected (fi-zip-nested) but not recursively extracted. Content inside a nested archive is not analyzed.
- **No execution:** Files are not opened, rendered, or run. A PDF with embedded JavaScript is flagged on structural indicators, not on what the JavaScript would do when executed by a reader.
- **MIME detection:** Based on file magic bytes. Files with no recognizable magic bytes are classified as unknown/neutral.
- **Office files:** OLE2-format Office files (`.doc`, `.xls`, `.ppt`) are inspected for VBA macro streams. OOXML-format files (`.docx`, `.xlsx`) macro detection is limited to extension-based indicators in V1.
- CRITICAL risk band is withheld in V1 pending calibration. The highest published verdict is HIGH.
