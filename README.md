# ShieldScope — Public Repository

This repository is the public transparency layer for [ShieldScope](https://shieldscope.app), a passive security posture analysis platform.

**What is in this repository:**
- Roadmap and upcoming work
- Changelog (mirrors [shieldscope.app/updates](https://shieldscope.app/updates))
- Known limitations per tool
- Methodology and scoring documentation

**What is not in this repository:**
- Source code (the platform backend is not open source)
- Infrastructure configuration
- Private keys or credentials

---

## Platform

ShieldScope is a set of passive security analysis tools for IT professionals and security researchers. All tools perform static or DNS-level analysis only — nothing executes, nothing is stored, no active scanning occurs.

Tools currently live at [shieldscope.app](https://shieldscope.app):

| Tool | Description |
|---|---|
| [/headers](https://shieldscope.app/headers) | HTTP security header analysis |
| [/mail](https://shieldscope.app/mail) | SPF, DKIM, and DMARC authentication analysis |
| [/url](https://shieldscope.app/url) | URL transparency and redirect chain analysis |
| [/password](https://shieldscope.app/password) | Password strength and breach exposure check |
| [/exposure](https://shieldscope.app/exposure) | Subdomain discovery and external attack surface mapping |
| [/sandbox](https://shieldscope.app/sandbox) | Static artifact analysis — URLs, scripts, emails, and files |

---

## Navigation

- [ROADMAP.md](ROADMAP.md) — upcoming work and priorities
- [CHANGELOG.md](CHANGELOG.md) — what has changed and when
- [KNOWN-LIMITATIONS.md](KNOWN-LIMITATIONS.md) — scope boundaries and known gaps
- [docs/methodology.md](docs/methodology.md) — how analysis works
- [docs/confidence-taxonomy.md](docs/confidence-taxonomy.md) — confidence classification model
- [docs/scoring-philosophy.md](docs/scoring-philosophy.md) — scoring and risk band definitions

---

## Contact

Security issues: [shieldscope.app/security](https://shieldscope.app/security)  
General: [shieldscope.app/contact](https://shieldscope.app/contact)
