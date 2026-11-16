# app-legal-compliance

An agent skill that checks applications against laws that commonly result in lawsuits or regulatory action. Covers accessibility, privacy, children's data, biometrics, financial data, health data, housing, and payment cards.

Each section explains what the law requires, shows implementation patterns in code, documents real lawsuits and their outcomes, and links to the official source.

---

## What it covers

| Law | Trigger |
|---|---|
| ADA Title III + WCAG 2.1 AA | Any public-facing UI |
| GDPR | Any EU/EEA user |
| CCPA / CPRA | California users (100K+ annually, or selling data, or $25M+ revenue) |
| State privacy laws (VA, CO, CT, TX) | Users in those states |
| COPPA | Any user who may be under 13 |
| BIPA / CUBI / WBPA | Any biometric data capture (face, fingerprint, voice) |
| GLBA Safeguards Rule | Financial account data |
| HIPAA | PHI processed on behalf of a covered entity |
| Fair Housing Act | Rental listings, tenant screening, housing ads |
| PCI DSS | Payment card data |

---

## How to use

Load the skill when any of the triggers above apply. The skill walks through each relevant law, tells you what to implement, and gives you a verification checklist before shipping.

**Claude Code:**
```
Read SKILL.md and apply it to the current feature.
```

**In a task spec:**
```
Before shipping, load the app-legal-compliance skill and run the verification checklist for the data surfaces this feature touches.
```

---

## Sections in SKILL.md

- **Overview** — why legal exposure is a code problem
- **Risk surface map** — a table that maps feature types to the laws they trigger
- **Per-law sections** — requirements, code patterns, notable cases, official guidance links
- **References** — consolidated table of all official sources and testing tools
- **Common rationalizations** — with the reality behind each one
- **Red flags** — specific patterns in code that signal exposure
- **Verification checklists** — context-specific, with checkboxes, broken out by feature type

---

## Keeping it current

Laws change. When a new law passes or an existing one is amended, update SKILL.md and add a note to CHANGELOG.md. The version in `metadata.version` inside SKILL.md should match the latest CHANGELOG entry.

This skill covers US federal law, the EU GDPR, and select state laws as of its last update. It is not a substitute for legal counsel when real exposure exists.

---

## License

MIT. See [LICENSE](./LICENSE).
