---
name: partner-vetting
description: Run the Everest Fleet pre-onboarding check on a prospective employee or vendor when the user shares a CV, resume, vendor profile or company document and asks to vet, verify, screen or check the person or company. Produces a sourced, confidence-tagged report for human review.
---

# Everest Fleet partner vetting

The output is a set of leads for a human decision maker, not a decision.
Never recommend reject or hire. Recommend what to verify next.

## Step 0: Gates (stop if any fails)

1. **Consent.** Ask the user to confirm the subject has signed Everest Fleet's
   background verification consent covering these checks (India DPDP Act 2023
   requires notice and consent for this processing). If not confirmed, stop and
   say so. Do not run "just the public part" as a workaround.
2. **Subject type.** Decide: `individual` (employee, consultant) or `entity`
   (vendor company, with its named promoters/directors as individuals).
3. **Storage.** Write inputs to `vetting/inputs/` and reports to
   `vetting/reports/`. Both are gitignored. Never commit, push, email or upload
   a report unless the user asks for that specific destination.

## Step 1: Extract claims

Read the CV/profile and list every checkable claim in a table:
name, email, phone, city, each employer with title and dates, each degree with
institution and year, certifications, company CIN/GSTIN/PAN (entities),
public links (LinkedIn, GitHub, website). Note date gaps over 3 months and
overlapping roles.

## Step 2: Anchor identity

Pick 2 or more anchors (name + employer + city, email, profile URL) before
searching. A search result counts as the subject only if it matches at least
two anchors. Otherwise record it under "Possible matches, not attributed".
Common Indian names produce frequent false matches: be strict.

## Step 3: Checks

Run what applies. For each, use WebSearch/WebFetch and record the URL.

**Individuals**
- Employers exist and existed in the claimed period (company site, MCA
  company master data, news). Flag shell or unverifiable employers.
- Titles and dates vs. public profiles and press. LinkedIn usually blocks
  fetching: use search snippets and say so.
- Institutions are recognised (UGC, AICTE lists). Flag known diploma mills.
  Degree authenticity itself needs the institution or DigiLocker: list it as a
  manual step.
- Professional footprint: publications, talks, GitHub, awards, news quotes.
- Adverse media: fraud, cheating, embezzlement, harassment, regulatory orders
  (SEBI, RBI), NCLT, director disqualification. Only where anchored.
- Directorships: MCA director search on the name (DIN), for undisclosed
  conflicts with Everest Fleet vendors or competitors.

**Entities (vendors)**
- MCA master data: CIN, status (active/strike off), incorporation date,
  directors, paid-up capital, open charges, last filing date.
- GSTIN status and registered name match (public GST search).
- Insolvency/NCLT, SEBI orders, blacklists, EPFO establishment presence.
- Adverse media on the entity and each director (anchored).
- Financial health from filed or published data only (revenue, charges,
  auditor remarks if public). Say plainly when financials are behind MCA's
  paywall and were not seen.

**Internal (optional, ask first)**
- Everest Reporting DB: read-only lookup for prior dealings with this
  person/vendor. Inspect schema before querying; do not guess table names.
- Gmail/Drive: prior correspondence, only if the user asks.

## Step 4: Out of scope (do not collect, even if found)

Religion, caste, health, disability, pregnancy, sexual orientation, political
views, union membership, family members, relationships, home location
tracking, personal (non-professional) social media content. If such content
surfaces incidentally, leave it out of the report.

Also not possible from here, list as manual/vendor steps: individual credit
score (CIBIL via licensed agency with consent), criminal/police verification,
Aadhaar/PAN verification, reference calls, physical address verification.

## Step 5: Report

Write `vetting/reports/<YYYY-MM-DD>_<subject-slug>.md` using
`vetting/REPORT_TEMPLATE.md`. Rules:
- Every finding has a source URL, access date and a tag: [Certain] (primary
  source, anchored), [Likely] (secondary source or one weak anchor),
  [Guessing] (inference; say what it rests on).
- "Nothing found" is a finding. Say what was searched.
- Separate "Discrepancies with CV" from "Adverse findings".
- Adverse findings get a "question to put to the subject" so they can respond
  before any decision.
- End with the manual verification checklist.

Reply in chat with a 5-line summary and the report path. Do not paste the
full report into chat unless asked.
