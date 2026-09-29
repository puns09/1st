---
name: partner-vetting
description: Run the Everest Fleet background check on a prospective employee or vendor (an individual broker who sources, onboards and manages drivers) when the user shares a CV, resume or profile and asks to vet, verify, screen or check the person. Produces a Word report for leadership, delivered in chat only.
---

# Everest Fleet background check

Audience: Everest Fleet leadership. Output: one Word (.docx) report sent in
chat. The report surfaces evidence and open questions. It never says hire or
reject.

Consent is handled by Legal. Do not ask about it.

## Data handling

- Work only in the session scratchpad directory. Never write the CV, notes or
  report inside the repository, Google Drive, Gmail or any artifact.
- Deliver the .docx with SendUserFile (display: attach), then delete the CV
  copy, notes and report from the scratchpad and say so.

## Step 1. Intake

Confirm with the user: subject type (`employee` or `vendor`), role, city.
Vendors are individuals who source, onboard and manage drivers for Everest
Fleet, not companies.

## Step 2. Extract claims from the CV

Name, phone, email, city, social handles, every employer with title and dates,
education, certifications. Flag gaps over 3 months and overlapping roles.

## Step 3. Anchor identity

A web result counts as the subject only if it matches 2 or more anchors:
name + city, employer, phone, email, or a confirmed handle. Weaker matches go
to "Possible matches, not attributed". Common names produce many false hits.

## Step 4. Work history (everyone)

- Each employer existed in the claimed period (company site, MCA data, news).
- Titles and dates vs. public profiles and press.
- Institution recognised (UGC / AICTE lists). Degree authenticity is a manual
  check.
- Undisclosed directorships or own businesses (MCA director search).

## Step 5. Vendor-specific checks

- Past or current work with other fleet operators or aggregators (Uber fleet
  partners, Ola, Rapido, other fleet companies).
- Driver complaints: Facebook groups, YouTube, Google reviews, complaint
  forums. Look for charging drivers fees, withholding earnings or deposits,
  vehicle financing schemes, poaching drivers to competitors.
- Driver recruitment posts under their name or number, especially ones asking
  for fees or deposits.
- GST registration or business in their name, if any.

## Step 6. Adverse media and legal

News of FIRs, fraud, cheating, cheque bounce (Sec 138 NI Act), recovery
suits, harassment, violence, regulatory orders. eCourts needs a captcha: add
it to the browser handoff list with the exact search to run.

## Step 7. Social media (every public account, none skipped)

Platforms: LinkedIn, Facebook, Instagram, X, YouTube, plus any handle in the CV.

1. Find accounts via web search on name + city + employer + phone.
2. Try to open each public account yourself (WebFetch, or the pre-installed
   Chromium via Playwright).
3. For every account or search that hits a login wall or captcha, add it to a
   numbered **browser handoff list**: exact URL or search string, and what to
   screenshot (profile header, About, recent 20 posts, any posts matching the
   Step 5 and 6 themes). Send the list to the user and wait for screenshots
   before finalising the report. Never mark a platform "nothing found" if it
   was not actually viewed; mark it "not viewed" instead.

Record only what bears on the role:
- Profile consistency with the CV (employer, title, city, dates).
- Conduct: threats, abuse, harassment, violence, bragging about fraud.
- Work for competitors, driver recruitment activity.

Never record religion, caste, political views, health, family, relationships
or personal life, even if clearly visible. Leave it out of notes and report.

## Step 8. Everest Reporting DB check (read-only)

Search the subject's full name (and phone, if on the CV) with case-insensitive
ILIKE. Report a section **only if the name is found**. If not found, omit the
section entirely.

Tables to check:

| Table | What a hit means | Columns to report |
|---|---|---|
| `public.fleet_driver` | Was an Everest driver | status, date_of_joining, date_of_exit, city_id |
| `public.fleet_driver_blacklist` (join `driver_id` to `fleet_driver.id`) | Blacklisted as a driver | start_date, end_date, blacklist_reason, whitelist_reason |
| `public.driver_exited` (join `driver_id`) | Driver exit record | disposition, deactivation_date |
| `crm.lead_lead_view` + `crm.lead_details` (on `lead_id`) | Applied as a driver | created_at, blacklisted, exit_reason, total_os |
| `public.everest_partners`, `vendor.franchise_vendors`, `public.everest_vendor_leads` | Past or current vendor / partner | vendor_type or partner_type, activation_date, deactivation_date |
| `public.everest_subvendor_vendor_mapping` | Was a sub-vendor | subvendor_name, city |
| `public.everest_employee_master` | Past or current employee | employment_status, designation, doj, date_of_relieving |
| `public.vendors` | Service or supply vendor | vendor_type, is_active |

Rules:
- Check the column names with `get_table_schema` before querying; the schema
  may have changed.
- Never copy PAN, bank details, DOB, address or father's name into the report.
- A name match alone is not identity. Tag it [Likely] only if city or phone
  also matches, otherwise [Guessing] and say "name match only".
- Outstanding balances (`total_os`, ledger balances) are relevant and should be
  reported for returning drivers or vendors.

## Step 9. Build the Word report

Load the `anthropic-skills:docx` skill, then build the .docx in the scratchpad.

Tags on every finding: [Certain] primary source and anchored; [Likely]
secondary source or weaker anchor; [Guessing] inference, state what it rests
on. Every finding has a source (URL, screenshot number, or DB table) and
access date.

Structure:

1. **Page 1, leadership summary**
   - Name, role, subject type, city, report date.
   - Status line: Clear / Minor discrepancies / Needs review.
   - Up to 5 bullets, [Certain] and [Likely] findings only.
   - Boxed "Unconfirmed, do not act on" list for [Guessing] items and
     weakly anchored matches.
2. **Claims vs. evidence** table: claim, evidence, source, tag.
3. **Discrepancies with CV**: item, CV says, evidence says, source, tag,
   question for the subject.
4. **Adverse findings**: finding, source, date, anchors matched, tag,
   question for the subject.
5. **Vendor checks** (vendors only).
6. **Social media**: one row per platform: handle, viewed by (Claude / user
   screenshot / not viewed), consistency with CV, relevant findings.
7. **Everest internal records** (only if the name was found).
8. **Searched, nothing found**: what was searched.
9. **Possible matches, not attributed.**
10. **Limits of this report**: what could not be accessed.
11. **Manual verification checklist**: degree verification, reference calls,
    police verification, credit check via licensed agency (if the role handles
    cash), address verification, subject's responses to open questions.

## Step 10. Deliver

Send the .docx with a 3-line chat summary (status line, top finding, number of
open questions). Then delete the working files and confirm.
