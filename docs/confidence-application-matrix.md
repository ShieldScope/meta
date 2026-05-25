# Confidence Application Matrix

Version 1.0 · May 2026

Reference document for how the ShieldScope confidence taxonomy applies across each tool and module. Enforces consistent semantics and prevents per-tool drift.

Read [confidence-taxonomy.md](confidence-taxonomy.md) first. This document applies those definitions — it does not redefine them.

---

## Cross-Tool Rules

These apply everywhere, without exception.

**Permitted tiers per finding type:**

| Finding type | Permitted | Prohibited |
|---|---|---|
| Value directly in retrieved data | OBSERVED | HEURISTIC, INFERRED |
| Count or score computed from data | DERIVED | OBSERVED |
| Pattern match against known indicators | HEURISTIC | OBSERVED, DERIVED |
| Conclusion beyond retrieved data | INFERRED | OBSERVED, DERIVED |
| Analysis ran, result contradictory | INDETERMINATE | All others |
| No data retrieved | *(scan state only)* | All confidence tiers |

**Global prohibitions:**
- Never assign HEURISTIC to a deterministic check. If header presence/absence is a yes/no from the HTTP response, it is OBSERVED.
- Never assign INFERRED where direct evidence is available. Do not downgrade to avoid appearing overconfident.
- Never use INDETERMINATE as a fallback for low confidence. It means analysis completed and produced contradictory results — not "we are not sure."
- Never publish a conclusion of "malicious," "dangerous," or "compromised" at any confidence tier. Describe what was found. Let the analyst conclude.
- Never use numeric percentages alongside or instead of confidence tiers.

---

## /headers — HTTP Security Header Analyzer

**Applicable tiers:** OBSERVED, DERIVED, INDETERMINATE  
**Not expected:** HEURISTIC, INFERRED

HTTP headers are deterministic. Present or absent. Value is what it is. There is no pattern matching and no behavioral inference in header analysis.

| Finding | Confidence | Trigger |
|---|---|---|
| Header is absent from response | OBSERVED | Header not present in HTTP response |
| Header is present | OBSERVED | Header present, value directly readable |
| Header value is a specific string | OBSERVED | Value read directly from response |
| CSP directive weakness classification | DERIVED | Parsing directive tree, evaluating unsafe directives |
| HSTS max-age computation | DERIVED | Numeric parsing and comparison against threshold |
| Header grade contribution | DERIVED | Weighted scoring across multiple headers |
| Conflicting header values (multiple instances) | INDETERMINATE | Two or more instances of the same header with different values |

**Prohibited usages:**
- HEURISTIC: header checks are yes/no, never pattern-based
- INFERRED: do not infer what a header "probably means for the site" — state what it is

**Edge cases:**
- CDN-injected headers that differ from origin: the finding reflects what ShieldScope received, not what the origin sends. The finding should note "reflects response at the CDN layer" if detectable. Confidence remains OBSERVED — we observed what we received.
- Headers present but with empty values: OBSERVED (present, empty). Not the same as absent.

---

## /mail — Email Authentication Analyzer

**Applicable tiers:** OBSERVED, DERIVED, HEURISTIC, INFERRED, INDETERMINATE  
**Not expected:** (all five are valid here)

DNS-based analysis. Most findings are OBSERVED or DERIVED. HEURISTIC applies to third-party sender identification. INFERRED applies to delivery behavior conclusions. INDETERMINATE applies to RFC-violating DNS states.

| Finding | Confidence | Trigger |
|---|---|---|
| SPF record present / absent | OBSERVED | DNS query result |
| SPF policy value (`~all`, `-all`, `+all`) | OBSERVED | Value read directly from record |
| DMARC record present / absent | OBSERVED | DNS query result |
| DMARC policy (`p=none`, `p=reject`) | OBSERVED | Value read directly from record |
| DMARC alignment mode (`adkim`, `aspf`) | OBSERVED | Value read directly from record |
| DKIM selector found | OBSERVED | DNS query returned a key record |
| SPF lookup count | DERIVED | Include chain expansion and recursive DNS resolution |
| DMARC enforcement readiness | DERIVED | Computed from: policy + alignment + DKIM presence |
| TLS certificate for mail server | DERIVED | Certificate chain parsing and normalization |
| Third-party sender identification | HEURISTIC | Match against known `include:` domain patterns — not exhaustive |
| SPF softfail delivery risk | INFERRED | Receiver behavior depends on local filtering policy |
| Delivery failure from lookup overflow | INFERRED | RFC requires permerror, but receiver enforcement varies |
| Multiple SPF records | INDETERMINATE | Two or more SPF records found — RFC 7208 violation, receiver behavior undefined |
| Malformed SPF record (partial parse) | INDETERMINATE | Record present but not fully parseable |

**Prohibited usages:**
- OBSERVED for DKIM absence: "DKIM not configured" cannot be OBSERVED. The check covers a bounded selector set. The correct finding is "not detected in N checked selectors." The confidence reflects what was checked, not what exists.
- INFERRED for SPF record values: the record value is OBSERVED. Only the behavioral consequence (what receivers will do) is INFERRED.
- HEURISTIC for DMARC policy: `p=none` is directly readable. Its adequacy as a security posture is DERIVED (scoring) or INFERRED (impact), not HEURISTIC.

**Edge cases:**
- DKIM absence: scope must be declared inline. "No common DKIM selectors detected. Custom selectors are not covered by this scan." The result is a bounded OBSERVED (checked N selectors, found none) plus a scope declaration. It is not INDETERMINATE.
- DMARC organizational domain fallback: when a subdomain has no DMARC record and the org domain record is used, note that the finding reflects the org domain policy. Confidence remains OBSERVED for the record value.
- SPF `include:` pointing to an inaccessible domain: the inaccessible record is a retrieval failure (`unreachable`) for that specific chain node. The overall lookup count finding becomes INDETERMINATE if the chain cannot be resolved.

---

## /url — URL Transparency Analyzer

**Applicable tiers:** OBSERVED, DERIVED, HEURISTIC, INFERRED  
**Not expected:** INDETERMINATE (URL analysis is typically resolvable or a retrieval failure)

Redirect chains are observed directly. Structural analysis of the URL is observed or derived. Risk interpretation is heuristic or inferred.

| Finding | Confidence | Trigger |
|---|---|---|
| Redirect destination URL | OBSERVED | HTTP response Location header |
| Final URL after chain | OBSERVED | Last response in the redirect chain |
| Redirect chain length | DERIVED | Count of hops followed |
| Domain registration age | DERIVED | WHOIS date parsed and compared against threshold |
| Homoglyph character present in hostname | OBSERVED | Unicode lookalike character is structurally present |
| URL structural pattern matches known phishing structure | HEURISTIC | Pattern match against known deceptive URL structures |
| Typosquatting suspicion | HEURISTIC | Edit distance comparison against known brand domains |
| Newly registered domain risk implication | INFERRED | Age is DERIVED; risk interpretation requires assumption about intent |
| Redirect chain classified as evasion | INFERRED | Chain is OBSERVED; evasion interpretation is contextual |

**Prohibited usages:**
- OBSERVED for "suspicious URL": the structure may be OBSERVED, the suspicion is HEURISTIC
- INFERRED for redirect destination: the destination is directly observed
- HEURISTIC for domain age: age is computed, not pattern-matched

**Edge cases:**
- Homoglyph detection: the character's presence is OBSERVED. The finding "this may be intended to impersonate X" is HEURISTIC — declare the false positive surface (internationalized domain names exist legitimately).
- Redirect to a different domain: OBSERVED (destination is directly observed). Interpretation as suspicious is HEURISTIC. Do not conflate these in the same confidence label.
- JavaScript or meta-refresh redirects: these are not followed. The finding "redirect chain may continue beyond this point" is INFERRED from the detected JS or meta tag.

---

## /password — Password Strength & Breach Exposure

**Applicable tiers:** OBSERVED, DERIVED  
**Not expected:** HEURISTIC, INFERRED, INDETERMINATE

Entirely client-side. HIBP returns a count. Strength is computed. No pattern matching, no behavioral inference.

| Finding | Confidence | Trigger |
|---|---|---|
| Breach count from HIBP | OBSERVED | API returned a match count for the hash prefix |
| Password not found in breaches | OBSERVED | API returned no match for the hash prefix |
| Entropy score | DERIVED | Computed from character set and length |
| Strength band classification | DERIVED | Entropy score mapped to band thresholds |

**Prohibited usages:**
- INFERRED for breach absence: "not found" is OBSERVED within HIBP's database scope. Do not upgrade it to "safe" — declare the scope (HIBP database at query time).
- HEURISTIC for any finding: password analysis here is deterministic

**Edge cases:**
- HIBP API unavailable: retrieval failure (`unreachable`). No confidence label. No breach finding published.
- Password found in breaches: the count is OBSERVED. The implication "this password should not be used" is a recommendation, not a confidence-labeled finding.

---

## /exposure — Internet Exposure Analyzer

**Applicable tiers:** OBSERVED, DERIVED, HEURISTIC, INFERRED  
**Not expected:** INDETERMINATE (network responses are typically determinate)

Subdomain discovery is observation-bounded. Service fingerprinting is derived. Infrastructure role conclusions are inferred.

| Finding | Confidence | Trigger |
|---|---|---|
| Subdomain exists | OBSERVED | CT log record confirmed |
| Port is open | OBSERVED | Connection established |
| HTTP/HTTPS service responds | OBSERVED | HTTP response received |
| TLS certificate expiration date | OBSERVED | Certificate field directly readable |
| Service type from response headers | DERIVED | Header parsing and normalization |
| Dangerous port classification | DERIVED | Port number compared against known-dangerous port list |
| TLS version support | DERIVED | Negotiated from connection, normalized |
| CDN attribution from IP range | DERIVED | IP matched against known CDN ASN/range table |
| CDN attribution from response patterns | INFERRED | No IP range confirmation — inferred from headers and cert patterns |
| Infrastructure role (staging, dev, internal) | INFERRED | Hostname pattern + response content — assumption required |
| Exposure risk implication | INFERRED | Service fingerprint + port combination interpreted as risk |

**Prohibited usages:**
- OBSERVED for "dangerous port": the port being open is OBSERVED; "dangerous" is a DERIVED classification
- HEURISTIC for subdomain existence: CT logs either contain the record or do not — this is OBSERVED
- OBSERVED for CDN attribution: attribution always involves a lookup or inference step — minimum DERIVED

**Edge cases:**
- Subdomains without certificates: not discovered (CT log scope). This is a scope limitation, not a retrieval failure. Declare it.
- CDN vs origin: the response may come from a CDN edge node, not the origin. Service fingerprinting findings reflect the edge layer. Note this where detectable.
- Port open but service unresponsive: port state is OBSERVED (connection established). Service identity is a retrieval failure (`unreachable` or `unsupported`) — no confidence label for the service finding.

---

## /sandbox — URL Tab

**Applicable tiers:** OBSERVED, DERIVED, HEURISTIC, INFERRED  
**Not expected:** INDETERMINATE

Static URL structure analysis. Does not follow or dereference the URL.

| Finding | Confidence | Trigger |
|---|---|---|
| Homoglyph character in URL | OBSERVED | Lookalike Unicode character structurally present |
| URL length | OBSERVED | Character count |
| Query parameter count | DERIVED | Parsed from URL structure |
| Entropy of URL path or subdomain | DERIVED | Computed from character distribution |
| URL structure matches known phishing pattern | HEURISTIC | Pattern match against known deceptive structures |
| Subdomain depth consistent with evasion | HEURISTIC | Depth compared against typical legitimate structure |
| Redirect chain evasion | INFERRED | Multi-hop pattern interpreted as deliberate misdirection |

**Prohibited usages:**
- OBSERVED for pattern-match conclusions: "suspicious URL structure" is HEURISTIC, not OBSERVED
- No finding should conclude "this URL is malicious" at any confidence tier

---

## /sandbox — Script Tab

**Applicable tiers:** OBSERVED, DERIVED, HEURISTIC, INFERRED  
**Not expected:** INDETERMINATE

Static pattern analysis of submitted script content. Nothing is executed.

| Finding | Confidence | Trigger |
|---|---|---|
| `eval()` present | OBSERVED | Keyword present in script |
| `unescape()` / `atob()` present | OBSERVED | Keyword present in script |
| Execution policy bypass flag | OBSERVED | Flag string present in script |
| Base64-encoded string present | OBSERVED | Encoding pattern structurally present |
| Entropy score | DERIVED | Computed from character distribution |
| String concatenation depth | DERIVED | Computed from AST or pattern count |
| Obfuscation pattern detected | HEURISTIC | High entropy + evasion-consistent patterns — also present in minified legitimate code |
| Encoded payload likely contains executable content | INFERRED | Encoding + context, without decoding the payload |

**Prohibited usages:**
- OBSERVED for "obfuscated script": the indicators (eval, high entropy) are OBSERVED; the conclusion "obfuscated" is HEURISTIC
- INFERRED for keyword presence: `eval()` being present is OBSERVED, not inferred
- No finding should state the script "will" execute malicious behavior — static analysis cannot observe runtime

**Edge cases:**
- Minified legitimate code: frequently triggers HEURISTIC obfuscation findings. The false positive surface must be declared per finding.
- Multi-stage payloads: only the first stage is observable. Findings are scoped to what is structurally present. Do not infer second-stage behavior.

---

## /sandbox — Email Tab

**Applicable tiers:** OBSERVED, DERIVED, HEURISTIC, INFERRED  
**Not expected:** INDETERMINATE (email headers are typically deterministic)

Static analysis of submitted email headers and body structure.

| Finding | Confidence | Trigger |
|---|---|---|
| Header field present / absent | OBSERVED | Field present or absent in submitted headers |
| From: domain value | OBSERVED | Directly readable from header |
| Return-Path: domain value | OBSERVED | Directly readable from header |
| From: / Return-Path: mismatch | OBSERVED | Both values readable; mismatch is structural |
| Authentication-Results header values | OBSERVED | Values directly readable |
| Reply-To: diverges from From: | OBSERVED | Both values readable |
| Hop count | DERIVED | Count of Received: headers |
| Routing anomaly classification | DERIVED | Received: chain parsed and compared against expected patterns |
| Display name impersonation suspicion | HEURISTIC | Display name contains brand or person name not matching sending domain |
| Phishing-style anchor text mismatch | HEURISTIC | Anchor text suggests a domain different from the href |
| Spoofing conclusion | INFERRED | Header mismatches observed; delivery behavior depends on receiver |

**Prohibited usages:**
- HEURISTIC for From:/Return-Path: mismatch: the values are directly readable — this is OBSERVED
- INFERRED for header field values: field presence and value are OBSERVED
- OBSERVED for "this email is a phishing attempt": structural observations support the finding; the spoofing conclusion is INFERRED

**Edge cases:**
- DKIM-Signature header present: the header is OBSERVED. Cryptographic validity is not checked — note this explicitly. Do not publish a DKIM validity finding.
- Forwarded email: legitimate forwarding commonly produces From:/Return-Path: mismatches and modified Received: chains. HEURISTIC findings in forwarded email have elevated false positive rates — declare this.

---

## /sandbox — File Tab

**Applicable tiers:** OBSERVED, DERIVED, HEURISTIC, INFERRED  
**Not expected:** INDETERMINATE

Static structural inspection of uploaded files. Nothing is executed or rendered.

| Finding | Confidence | Trigger |
|---|---|---|
| File extension | OBSERVED | Extension present in filename |
| Homoglyph character in filename | OBSERVED | Lookalike Unicode character structurally present |
| Double extension | OBSERVED | Two extensions present in filename |
| MIME type | DERIVED | Determined from file magic bytes |
| Extension / MIME mismatch | DERIVED | Extension compared against MIME type from magic bytes |
| File entropy | DERIVED | Computed from byte distribution |
| PDF JavaScript stream present | OBSERVED | `/JavaScript` object present in PDF structure |
| PDF launch action present | OBSERVED | Launch action object present in PDF structure |
| PDF embedded file present | OBSERVED | Embedded file stream present in PDF structure |
| OLE2 VBA macro stream present | OBSERVED | VBA stream present in OLE2 container |
| ZIP contains executable | DERIVED | ZIP entries parsed; executable extension or MIME identified in entries |
| ZIP nested archive | OBSERVED | Archive entry present inside ZIP |
| Entropy consistent with packed/encrypted payload | HEURISTIC | High entropy score — also present in legitimately compressed content |
| File behavior implication | INFERRED | Structure indicates capability; actual behavior requires execution |

**Prohibited usages:**
- OBSERVED for MIME type: MIME is computed from magic bytes — it is DERIVED
- OBSERVED for "file is malware": structural indicators are OBSERVED or DERIVED; malware classification requires execution context
- HEURISTIC for OLE2 VBA stream presence: the stream is structurally present — this is OBSERVED

**Edge cases:**
- OOXML files (.docx, .xlsx, .pptx): macro detection in V1 is extension-based only. OLE2 inspection does not apply. Declare this scope limitation on any macro-related finding for OOXML files.
- ZIP entries beyond the 100-entry limit: entries beyond the limit are not inspected. Declare partial analysis where applicable.
- Files with no recognizable magic bytes: MIME is unknown — this is not a retrieval failure, it is a DERIVED result of `unknown`. Risk band is NEUTRAL, not CLEAN.
- Password-protected archives: content is inaccessible. Retrieval state `unsupported` for the content. The archive structure (size, entry names if readable) may still produce findings.
