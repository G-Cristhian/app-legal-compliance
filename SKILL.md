---
name: app-legal-compliance
description: |
  Checks applications against laws that commonly result in lawsuits or regulatory action.
  Use when building any public-facing UI, adding a data collection point, handling payments,
  capturing biometric data, or serving users who may be minors. Use before any launch, when
  adding a new feature that processes personal data, or when auditing an existing app for
  legal exposure. Use when a client or stakeholder asks "are we compliant with...?"
license: MIT
metadata:
  version: "1.0.0"
---

# App Legal Compliance

## Overview

Applications get sued for three reasons: they collect data without proper consent, they exclude people with disabilities, or they handle regulated categories of data (health, financial, children's) without following the rules.

Legal exposure is a code problem. Consent mechanisms, data deletion paths, accessibility attributes, and age gates are all implementation details that live in the codebase. By the time a demand letter arrives, retrofitting them is expensive. Building them in is not.

This skill is a developer checklist, not legal advice. When real exposure exists, involve a lawyer.

## When to Use

- Building any UI accessible to the public
- Adding any data collection point: forms, analytics, biometrics, tracking pixels
- Shipping to users in California, the EU, Illinois, or any other regulated jurisdiction
- Building anything that might reach users under 13
- Working on payment flows or anything that stores health or financial data
- Auditing an existing app before a legal review or investor due diligence
- When a product decision touches resident screening, ad targeting, or occupancy (housing apps)

## Process: Map the Risk Surface First

Before checking individual laws, identify which surfaces your feature touches. A feature can trigger multiple laws simultaneously.

| Feature type | Laws triggered |
|---|---|
| Any public UI | ADA Title III, WCAG 2.1 AA |
| Data collection (any form, analytics, tracking) | GDPR, CCPA/CPRA, state privacy laws |
| Users who may be under 13 | COPPA |
| Facial recognition, fingerprints, voice capture | BIPA (Illinois), CUBI (Texas), WBPA (Washington) |
| Financial account data | GLBA Safeguards Rule |
| Health records or PHI on behalf of a provider | HIPAA |
| Rental listings, tenant screening, housing ads | Fair Housing Act, HUD digital guidance |
| Payment card numbers | PCI DSS |

If your feature touches more than one row, work through each section below. The laws are independent and can apply simultaneously to the same code path.

## Accessibility: ADA Title III and WCAG 2.1 AA

### Why This Gets Apps Sued

Courts have held that websites and apps are places of public accommodation under ADA Title III. Over 4,000 ADA web accessibility lawsuits were filed in 2023 in the US alone. Most start with a demand letter, not a formal complaint. Automated scanner tools generate these letters at scale, so an inaccessible form or missing alt text is a discoverable target.

WCAG 2.1 AA is the technical standard courts and regulators use as the benchmark for compliance.

### Notable Cases

| Company | What happened | Outcome |
|---|---|---|
| Domino's Pizza | Blind user couldn't order via website or app despite using a screen reader. 9th Circuit ruled ADA applies to websites. | Remanded for trial; $4.5M settlement (2021) |
| Target | Blind users couldn't navigate checkout on target.com. | $6M class action settlement (2008) |
| Netflix | DOJ investigated inaccessible streaming content. | $755K settlement + captioning requirements (2012) |
| Beyoncé / Parkwood Entertainment | Website had no alt text, inaccessible video players, and missing form labels. | Suit filed 2019; settled confidentially |

### What to Implement

**Images and media:**
```html
<!-- Wrong: decorative alt text -->
<img src="chart.png" alt="image">

<!-- Right: descriptive alt text -->
<img src="chart.png" alt="Bar chart showing Q3 revenue by region">

<!-- Right: decorative image marked explicitly -->
<img src="divider.png" alt="" role="presentation">
```

**Form labels (the most common failure):**
```html
<!-- Wrong: placeholder is not a label -->
<input type="email" placeholder="Email address">

<!-- Right: associated label -->
<label for="email">Email address</label>
<input type="email" id="email" name="email">
```

**Color contrast:** 4.5:1 for normal text, 3:1 for large text (18px+ or 14px+ bold). Use Lighthouse, Colour Contrast Analyser, or the browser DevTools contrast checker. Color alone must never be the only way to convey information: pair it with a label, pattern, or icon.

**Keyboard navigation:** Every interactive element (buttons, links, form fields, modals, dropdowns) must be reachable and operable via Tab/Shift+Tab, Enter, Space, and arrow keys. Test by unplugging the mouse.

**Error messages:**
```html
<!-- Wrong: generic error -->
<span class="error">Invalid input</span>

<!-- Right: identifies the field and how to fix it -->
<span id="email-error" role="alert">
  Email address is required. Enter your email in the format name@example.com.
</span>
<input type="email" id="email" aria-describedby="email-error" aria-invalid="true">
```

**Video and audio:** Captions for video, transcripts for audio-only content.

**Seizure safety:** No element flashes more than 3 times per second.

**Modals:** When a modal opens, focus must move into it. When it closes, focus must return to the trigger element.

**Page structure:**
```html
<!-- One H1 per page, logical hierarchy below it -->
<html lang="en">   <!-- language declaration required -->
<h1>Project overview</h1>
  <h2>Schedule</h2>
    <h3>Q3 milestones</h3>
```

**Testing:** Automated tools (axe DevTools, WAVE, Lighthouse) catch roughly 30% of WCAG failures. Test the remaining 70% manually with NVDA on Windows or VoiceOver on Mac.

**Official guidance:** [WCAG 2.1 full text](https://www.w3.org/TR/WCAG21/) · [DOJ ADA web guidance](https://www.ada.gov/resources/web-guidance/) · [WebAIM contrast checker](https://webaim.org/resources/contrastchecker/) · [axe DevTools](https://www.deque.com/axe/)

---

## Privacy: GDPR, CCPA/CPRA, and State Laws

### GDPR (EU/EEA Users)

GDPR applies to any app that processes personal data of people in the EU or EEA, regardless of where the company is incorporated. Fines reach 4% of global annual revenue or €20M, whichever is higher.

### Notable Cases

| Company | What happened | Outcome |
|---|---|---|
| Meta | Transferred EU user data to US servers without adequate safeguards. | €1.2B fine, Ireland DPC (2023) — largest GDPR fine to date |
| Amazon | Multiple GDPR violations including targeted advertising. | €746M fine, Luxembourg (2021) |
| Google | Cookie consent UI designed to steer users toward acceptance. | €150M fine, France CNIL (2022) |
| WhatsApp | Failed to explain how it shared data with Facebook; privacy notices inadequate. | €225M fine, Ireland DPC (2021) |

**Lawful basis:** Every data collection point requires a documented lawful basis (consent, contract, legitimate interest, legal obligation, vital interests, or public task). "Might be useful later" is not a lawful basis.

**Consent:** Consent must be freely given, specific, informed, and unambiguous. Pre-ticked boxes, bundled consents, and "by using this service you agree to tracking" do not qualify.

```javascript
// Wrong: analytics fires before consent
analytics.init({ trackingId: 'UA-XXXXX' });

// Right: analytics fires only after explicit consent
consentManager.onAccept('analytics', () => {
  analytics.init({ trackingId: 'UA-XXXXX' });
});
```

**Data subject rights:** Users must be able to request deletion, access, rectification, and portability. Design the schema so a user's data is findable and erasable, including backups, caches, search indexes, and analytics copies.

**Data processors:** Every third-party service that processes personal data on your behalf (analytics, error tracking, email, cloud hosting) requires a signed Data Processing Agreement (DPA). Most major vendors (Google, AWS, Stripe) provide these; request them before going live.

**International transfers:** Sending personal data to countries without an EU adequacy decision requires Standard Contractual Clauses (SCCs) or another approved mechanism.

### CCPA/CPRA (California Users)

Applies if you have 100,000+ California users annually, sell or share personal data, or have $25M+ in annual revenue.

**What to add to the footer:**
```html
<!-- Required link, must be visible on every page -->
<a href="/privacy/do-not-sell">Do Not Sell or Share My Personal Information</a>
```

The opt-out must work without penalizing the user (no paywalls, no degraded experience). The link cannot be hidden or styled to discourage clicks.

```typescript
// Opt-out handler: stops data sharing and updates preferences atomically
app.post('/privacy/do-not-sell', requireAuth, async (req, res) => {
  const { userId } = req.user;

  await db.privacyPreferences.upsert({
    userId,
    doNotSell: true,
    doNotShare: true,
    updatedAt: new Date(),
  });

  // Stop forwarding this user's data to ad networks and analytics vendors
  await Promise.all([
    adPlatform.setOptOut(userId, true),
    analyticsService.setOptOut(userId, true),
  ]);

  res.json({ success: true, message: 'Your opt-out has been recorded.' });
});
```

**Data subject requests:** 45-day response window. Build the request intake form before launch, not after the first request arrives.

**Sensitive personal information (SPI):** Social Security numbers, financial account data, health data, geolocation precise to 1,850 feet, racial or ethnic origin, religious beliefs, biometric data, and sexual orientation each require a separate opt-out mechanism under CPRA.

**Official guidance:** [GDPR full text](https://gdpr-info.eu/) · [EU guidance on lawful basis](https://edpb.europa.eu/our-work-tools/our-documents/guidelines/guidelines-052020-consent-under-regulation-2016679_en) · [CCPA/CPRA regulations](https://cppa.ca.gov/regulations/)

### Other State Laws

Virginia (VCDPA), Colorado (CPA), Connecticut (CTDPA), and Texas (TDPSA) follow a structure similar to CCPA. If you have users in those states and handle personal data, those laws apply. The consent and data subject rights requirements are substantially the same; check each state's specific thresholds.

---

## Children's Data: COPPA

The Children's Online Privacy Protection Act covers any service directed at children under 13, and any general-audience service with actual knowledge it is collecting data from someone under 13. The FTC enforces it.

**Age gate placement:** The age check must come before any data collection, not after account creation.

```javascript
// Wrong: collects email before age check
async function register(email, age) {
  await db.createUser({ email }); // PII already stored
  if (age < 13) return { error: 'Must be 13 or older' };
}

// Right: age check before any data persists
async function register(email, age) {
  if (age < 13) return { error: 'Must be 13 or older' };
  await db.createUser({ email });
}
```

**Verifiable parental consent:** If your app intentionally serves under-13 users, you need verifiable parental consent before collection. "The child said their parent said yes" does not qualify. Acceptable methods include signed consent forms, credit card verification, and phone or video calls.

**Third-party SDKs:** Analytics, advertising, and crash-reporting SDKs often collect device identifiers and usage data by default. If your app can reach under-13 users, check that every SDK offers a child-directed mode and enable it before launch.

**Data retention:** Delete children's data once it is no longer needed for the purpose it was collected. Document the retention period and enforce it in code, not in a privacy policy PDF.

**Official guidance:** [FTC COPPA rule](https://www.ftc.gov/legal-library/browse/rules/childrens-online-privacy-protection-rule-coppa) · [FTC COPPA FAQ for businesses](https://www.ftc.gov/tips-advice/business-center/guidance/complying-coppa-frequently-asked-questions)

### Notable Cases

| Company | What happened | Outcome |
|---|---|---|
| Epic Games (Fortnite) | Collected data from under-13 players without parental consent; dark patterns triggered purchases by minors. | $275M FTC settlement (2023) — largest COPPA fine ever |
| YouTube / Google | YouTube Kids served targeted ads to children; channels directed at children collected data without consent. | $170M FTC settlement (2019) |
| Amazon (Alexa) | Retained children's voice recordings indefinitely; failed to honor deletion requests. | $25M FTC settlement (2023) |
| TikTok (Musical.ly) | Collected personal data from under-13 users without parental consent. | $5.7M FTC settlement (2019) |

---

## Biometric Data: BIPA and Similar Laws

The Illinois Biometric Information Privacy Act is the most litigated biometric privacy law in the US. Its private right of action (individuals can sue without proving harm) is why it generates class actions. Statutory damages are $1,000 per negligent violation and $5,000 per intentional or reckless violation, multiplied by each affected user.

Biometric identifiers covered: fingerprints, voiceprints, facial geometry derived from a photo or video scan, retina scans, hand geometry.

### What to Implement Before Collection Begins

1. **Published retention and destruction policy:** Must be publicly available before any collection. Post it on your website.

2. **Written informed consent:** Obtained before collection, not bundled into a general terms of service.

```javascript
// Wrong: biometric capture starts on page load
startFacialRecognition();

// Right: explicit consent before any biometric processing
async function requestBiometricCapture() {
  const consent = await showBiometricConsentModal({
    purpose: 'Verify your identity during check-in',
    retentionPeriod: '3 years from last use, then destroyed',
    thirdPartySharing: 'None',
  });

  if (!consent.accepted) return;
  await recordConsent(userId, 'biometric', new Date());
  startFacialRecognition();
}
```

3. **No sale or profit:** Biometric data cannot be sold, leased, traded, or used to generate profit.

4. **Destruction schedule:** Biometric data must be destroyed when the initial purpose has been satisfied, or within 3 years of last interaction, whichever comes first. Build the deletion job before shipping the collection code.

**Other states:** Texas (CUBI) and Washington (WBPA) have similar requirements. New York City has biometric identifier ordinances for commercial establishments.

**Official guidance:** [Illinois BIPA full text](https://www.ilga.gov/legislation/ilcs/ilcs3.asp?ActID=3004) · [Texas CUBI](https://statutes.capitol.texas.gov/Docs/BC/htm/BC.503.htm) · [Washington WBPA](https://app.leg.wa.gov/rcw/default.aspx?cite=19.375)

### Notable Cases

| Company | What happened | Outcome |
|---|---|---|
| Meta (Facebook) | Facial recognition feature "Tag Suggestions" collected face geometry from photos of Illinois users without consent. | $650M class action settlement (2021) |
| Google Photos | Automatically scanned and tagged Illinois users' faces without informed written consent. | $100M class action settlement (2022) |
| TikTok | Collected biometric identifiers including faceprints and voiceprints from Illinois users. | $92M class action settlement (2021) |
| Snap (Snapchat) | Lenses and filters collected face geometry from Illinois users without a published retention policy. | $35M class action settlement (2022) |
| Six Flags | Collected fingerprints from season pass holders without a written consent policy. Illinois Supreme Court ruled no actual harm required to sue. | Established BIPA private right of action (2019) — opened the door to current class action volume |

---

## Financial and Health Data: GLBA and HIPAA

### GLBA Safeguards Rule (Financial Data)

The FTC's Gramm-Leach-Bliley Act Safeguards Rule applies to financial institutions, which the FTC defines broadly. If your app collects, transmits, or processes financial account numbers, routing numbers, credit data, or income information and falls under the FTC's definition, the Safeguards Rule applies.

**What to implement:**
- Encryption in transit (TLS 1.2+) and at rest (AES-256) for all financial data
- Access controls: least privilege, role-based, with audit logging
- Annual risk assessment: documented, reviewed, acted on
- Incident response plan: who does what, in what order, within what timeframe

```typescript
// Every read of financial data must be logged: who, what, when, why
async function getAccountData(
  targetUserId: string,
  requestingUserId: string,
  purpose: 'statement' | 'payment' | 'audit'
) {
  await db.auditLog.create({
    action: 'financial_data_access',
    targetUserId,
    requestedBy: requestingUserId,
    purpose,
    timestamp: new Date(),
    ipAddress: req.ip,
  });

  // Return only the fields the stated purpose requires (minimum necessary)
  return db.accounts.findOne({
    where: { userId: targetUserId },
    select: fieldsForPurpose(purpose),
  });
}

// Wrong: blanket access to the full record
const account = await db.accounts.findOne({ userId });

// Right: scoped by purpose
const account = await getAccountData(userId, currentUser.id, 'statement');
```

**Official guidance:** [FTC GLBA Safeguards Rule](https://www.ftc.gov/legal-library/browse/rules/safeguards-rule)

### HIPAA (Health Data)

If your app processes Protected Health Information (PHI) on behalf of a covered entity (hospital, insurer, provider), you are a Business Associate and HIPAA applies. PHI includes any health information that can be linked to a specific individual.

Before handling PHI, you must have a signed Business Associate Agreement (BAA) with the covered entity. Without a BAA, you should not be in possession of PHI.

**Technical safeguards:**
- PHI encrypted in transit (TLS 1.2+) and at rest (AES-256)
- Access to PHI logged and auditable: who accessed what record, when
- Automatic session timeout for interfaces that display PHI
- Minimum necessary principle: code should request and display only the PHI fields the current operation actually requires

```typescript
// Auto session timeout on any screen that displays PHI
const PHI_TIMEOUT_MS = 15 * 60 * 1000; // 15 minutes, per common HIPAA guidance

function usePhiSessionTimeout(onTimeout: () => void) {
  const timer = useRef<ReturnType<typeof setTimeout>>();

  const resetTimer = useCallback(() => {
    clearTimeout(timer.current);
    timer.current = setTimeout(onTimeout, PHI_TIMEOUT_MS);
  }, [onTimeout]);

  useEffect(() => {
    ['mousemove', 'keypress', 'click'].forEach(e =>
      document.addEventListener(e, resetTimer)
    );
    resetTimer();
    return () => {
      ['mousemove', 'keypress', 'click'].forEach(e =>
        document.removeEventListener(e, resetTimer)
      );
      clearTimeout(timer.current);
    };
  }, [resetTimer]);
}

// Minimum necessary: select only the fields the operation requires
// Wrong: fetch the full patient record
const patient = await db.patients.findOne({ id: patientId });

// Right: scope the query to what this view needs
const patient = await db.patients.findOne({
  where: { id: patientId },
  select: ['name', 'dateOfBirth', 'nextAppointment'],
});
```

---

## Fair Housing (Housing Applications)

The Fair Housing Act (FHA) prohibits discrimination based on race, color, national origin, religion, sex, familial status, and disability. HUD has issued guidance making clear that digital tools are covered: algorithmic screening, online applications, and ad targeting all fall under the Act.

### Algorithmic Screening

Income-to-rent ratio requirements, credit score minimums, and criminal history filters can produce disparate impact on protected classes even when they appear neutral. HUD and courts evaluate disparate impact using statistical analysis of outcomes across protected groups.

Before shipping any scoring or filtering feature for housing:
- Document the business necessity for each screening criterion
- Review outcome distributions across race, national origin, familial status, and disability
- Confirm the tool does not exclude categories that FHA protects (familial status filters are common violations)

```typescript
// Document the business necessity and compliance status of every screening criterion
// This object is the paper trail a compliance review or court will ask for
const SCREENING_CRITERIA = {
  incomeToRentRatio: {
    threshold: 2.5,
    businessNecessity: 'Industry standard threshold for rent affordability risk',
    disparateImpactReviewed: true,
    lastReviewDate: '2026-01-15',
  },

  // Wrong: blanket criminal history exclusion
  // criminalHistory: { excludeAny: true },

  // Right: individualized assessment per HUD 2016 guidance
  criminalHistory: {
    policy: 'individualized-assessment',
    factors: ['nature of offense', 'time elapsed', 'evidence of rehabilitation'],
    hudGuidanceCompliant: true,
  },

  // Wrong: familial status as a filter
  // occupantsPerBedroom: { max: 1 },  // excludes families with children; FHA violation

  // Right: health and safety standard with documented basis
  occupantsPerBedroom: {
    max: 2,
    basis: 'IRC Section R304 minimum square footage standard',
    hudGuidanceCompliant: true,
  },
} satisfies Record<string, ScreeningCriterion>;
```

**Official guidance:** [HHS HIPAA for professionals](https://www.hhs.gov/hipaa/for-professionals/index.html) · [HHS HIPAA security rule](https://www.hhs.gov/hipaa/for-professionals/security/index.html)

**Official guidance:** [HUD Fair Housing Act overview](https://www.hud.gov/program_offices/fair_housing_equal_opp/fair_housing_act_overview) · [HUD 2016 guidance on criminal history screening](https://www.hud.gov/sites/documents/HUD_OGCGUIDAPPFHASTANDCR.PDF) · [HUD guidance on algorithmic tools](https://www.hud.gov/sites/dfiles/PA/documents/HUDAIReport2023.pdf)

### Notable Cases

| Company | What happened | Outcome |
|---|---|---|
| Meta (Facebook) | Housing ads on the platform allowed advertisers to exclude users by race, national origin, religion, sex, and familial status through targeting tools. | $115,054 HUD settlement (2022); prior $5M settlement with NFHA and other civil rights orgs (2019) |
| Realtor.com (News Corp) | Alleged steering of users toward or away from neighborhoods based on demographic composition. | HUD complaint filed 2023, ongoing |
| Twitter | HUD filed complaint alleging housing ads on the platform allowed discriminatory targeting. | HUD complaint filed 2023, ongoing |

### Ad Targeting

Meta settled with HUD in 2022 for $115,054 over housing ad targeting that excluded users by zip code as a proxy for race. If your app serves housing ads or integrates with ad platforms, review the targeting parameters before launch. Protected characteristics and their proxies (zip code, neighborhood name, language) cannot be exclusion criteria for housing ads.

### Online Applications

Online rental applications must not ask for information about protected classes before or during the screening process. Reasonable accommodation request workflows must exist and must be accessible to users with disabilities.

---

## Payment Cards: PCI DSS

PCI DSS is a contractual requirement from card networks (Visa, Mastercard, Amex), not a statute. Violations result in fines from card brands, assessments, and potential loss of payment processing. A breach while out of compliance shifts liability to the merchant.

**Official guidance:** [PCI DSS standards library](https://www.pcisecuritystandards.org/document_library/) · [Stripe PCI compliance guide](https://stripe.com/guides/pci-compliance) · [PCI DSS quick reference](https://www.pcisecuritystandards.org/documents/PCI_DSS-QRG-v3_2_1.pdf)

### Notable Cases

| Company | What happened | Outcome |
|---|---|---|
| Target | Attackers compromised POS systems and exfiltrated 40M card numbers. PCI DSS controls were in place but not properly monitored. | $18.5M multistate AG settlement (2017); $67M Visa settlement |
| Home Depot | Malware on payment terminals exposed 56M card numbers. PCI compliance had lapsed. | $19.5M consumer settlement (2016); $25M bank settlement |
| Heartland Payment Systems | SQL injection attack exposed 130M card numbers. Largest card breach at the time. | $110M+ in settlements (2009-2010) |

The core rule: never let card data touch your servers.

```javascript
// Wrong: card form in your DOM; card numbers transit your server
<form action="/payment">
  <input name="card_number" type="text">
  <input name="cvv" type="text">
</form>

// Right: processor-hosted iframe; card data never enters your DOM
<div id="card-element">
  <!-- Stripe, Braintree, or similar injects a hosted iframe here -->
</div>
<script>
  const cardElement = stripe.elements().create('card');
  cardElement.mount('#card-element');
</script>
```

**What to verify:**
- Raw card numbers, CVVs, and magnetic stripe data are never logged, stored, or transmitted through your systems
- Payment forms use the processor's hosted fields or redirect — not an inline HTML form
- If you store payment tokens, confirm PCI scope with your processor
- Confirm your processor has signed a PCI-compliant contract with you

---

## References

### Laws and official guidance

| Area | Resource |
|---|---|
| ADA web accessibility | [DOJ ADA web guidance](https://www.ada.gov/resources/web-guidance/) |
| WCAG 2.1 AA | [W3C WCAG 2.1 full text](https://www.w3.org/TR/WCAG21/) |
| GDPR | [GDPR full text (gdpr-info.eu)](https://gdpr-info.eu/) |
| GDPR consent rules | [EDPB consent guidelines](https://edpb.europa.eu/our-work-tools/our-documents/guidelines/guidelines-052020-consent-under-regulation-2016679_en) |
| CCPA / CPRA | [California Privacy Protection Agency](https://cppa.ca.gov/regulations/) |
| COPPA | [FTC COPPA rule](https://www.ftc.gov/legal-library/browse/rules/childrens-online-privacy-protection-rule-coppa) |
| COPPA FAQ | [FTC COPPA business FAQ](https://www.ftc.gov/tips-advice/business-center/guidance/complying-coppa-frequently-asked-questions) |
| BIPA (Illinois) | [Illinois BIPA statute](https://www.ilga.gov/legislation/ilcs/ilcs3.asp?ActID=3004) |
| CUBI (Texas) | [Texas CUBI statute](https://statutes.capitol.texas.gov/Docs/BC/htm/BC.503.htm) |
| WBPA (Washington) | [Washington WBPA statute](https://app.leg.wa.gov/rcw/default.aspx?cite=19.375) |
| GLBA Safeguards Rule | [FTC Safeguards Rule](https://www.ftc.gov/legal-library/browse/rules/safeguards-rule) |
| HIPAA | [HHS HIPAA for professionals](https://www.hhs.gov/hipaa/for-professionals/index.html) |
| HIPAA security rule | [HHS HIPAA security rule](https://www.hhs.gov/hipaa/for-professionals/security/index.html) |
| Fair Housing Act | [HUD Fair Housing overview](https://www.hud.gov/program_offices/fair_housing_equal_opp/fair_housing_act_overview) |
| Fair Housing + criminal history | [HUD 2016 criminal history guidance (PDF)](https://www.hud.gov/sites/documents/HUD_OGCGUIDAPPFHASTANDCR.PDF) |
| Fair Housing + AI/algorithms | [HUD AI report 2023 (PDF)](https://www.hud.gov/sites/dfiles/PA/documents/HUDAIReport2023.pdf) |
| PCI DSS | [PCI Security Standards Council](https://www.pcisecuritystandards.org/document_library/) |

### Testing tools

| Tool | Purpose |
|---|---|
| [axe DevTools](https://www.deque.com/axe/) | Automated accessibility scanning (browser extension + CI) |
| [WAVE](https://wave.webaim.org/) | Visual accessibility evaluation |
| [Lighthouse](https://developer.chrome.com/docs/lighthouse/) | Accessibility, performance, and best practices (built into Chrome DevTools) |
| [WebAIM Contrast Checker](https://webaim.org/resources/contrastchecker/) | Color contrast ratio calculator |
| [NVDA](https://www.nvaccess.org/download/) | Free screen reader for Windows (manual accessibility testing) |

## Common Rationalizations

| Rationalization | Reality |
|---|---|
| "We're too small for this to apply" | GDPR applies at the first EU user. COPPA applies at the first under-13 user. BIPA plaintiffs often target small companies because they are easier to pursue. |
| "We'll add accessibility later" | Demand letters arrive before you're ready. Courts have held that inaccessible websites are ADA violations in progress, not future ones. |
| "Our privacy policy covers it" | A privacy policy that does not match actual data practices becomes evidence in litigation, not a defense against it. |
| "We use a third-party for payments, so PCI doesn't apply" | If your DOM ever touches card data — even briefly through an inline form — you are in PCI scope. |
| "We don't have users in Illinois" | BIPA applies where the biometric data is collected from, not where your company is located. If an Illinois resident uses your app, BIPA applies. |
| "The user agreed to our terms of service" | Blanket ToS acceptance does not satisfy GDPR consent requirements. Consent must be specific, informed, and freely given. |
| "No one has complained" | Demand letters are generated by automated accessibility scanners. You may be in the queue. |
| "We'll handle deletion requests manually" | Manual erasure misses backups, caches, and analytics copies. If the schema cannot find a user's data, the deletion request cannot be honored. Design for it. |
| "HIPAA is the hospital's problem, not ours" | If you hold PHI on behalf of a covered entity without a BAA, you are in violation. The hospital's compliance does not cover you. |

## Red Flags

- Cookie banner fires after analytics loads, not before
- Form labels implemented as placeholder text that disappears when the user types
- Biometric feature with no published retention or destruction policy
- "Do Not Sell" link that leads to a form with no confirmation or response window
- Application collects income, credit score, or employment before disclosing how that data is used
- Age verification happens after account creation or after any personal data is collected
- Payment form is not an iframe or redirect served by the processor
- Ad targeting for housing includes zip code, neighborhood, or language as exclusion filters
- Algorithmic screening tool ships without a disparate impact review
- Third-party SDKs present in an app that may reach under-13 users, without child-directed mode enabled
- Schema has no deletion path for personal data (no user_id index on data tables, no soft-delete with hard-delete job)
- PHI-handling code with no BAA in place

## Verification

### Before shipping any feature that collects data or renders a public UI:

- [ ] Automated accessibility scan (axe or Lighthouse) run; all critical and serious findings resolved
- [ ] Keyboard-only navigation tested through every interactive flow
- [ ] All form inputs have associated labels (not placeholder-only)
- [ ] Color contrast meets 4.5:1 for normal text, 3:1 for large text
- [ ] All consent mechanisms fire before data collection, not after
- [ ] Privacy policy updated to cover any new data points collected
- [ ] Third-party data processors have signed DPAs before data is sent to them

### Before shipping a biometric feature:

- [ ] Written retention and destruction policy published on the website before any collection begins
- [ ] Informed written consent modal in place before any biometric identifier is captured
- [ ] Deletion job implemented and scheduled
- [ ] No sale or profit path exists for the biometric data

### Before shipping a payment flow:

- [ ] Card data never enters your DOM (hosted fields or redirect confirmed)
- [ ] No card numbers, CVVs, or magnetic stripe data stored or logged anywhere in your systems
- [ ] PCI scope confirmed with your payment processor

### Before shipping a housing feature with screening or ads:

- [ ] Screening criteria reviewed for disparate impact on FHA-protected classes
- [ ] Ad targeting parameters reviewed for protected-class proxies (zip code, neighborhood, language)
- [ ] Reasonable accommodation request workflow exists and is accessible

### Before shipping anything that may reach users under 13:

- [ ] Age gate confirmed to run before any personal data collection
- [ ] All third-party SDKs configured with child-directed mode enabled
- [ ] Data retention schedule documented and enforced in code
