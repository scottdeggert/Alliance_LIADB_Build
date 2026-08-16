# OPEN-ITEMS.md

Every open question in the corpus, in one place. Sourced from all twenty-six specs, `Handbook.md`, `DECISIONS.md`, `docs/email/TEMPLATES.md`, and `docs/migration/field-map.md`.

**Scope boundary.** This file tracks open items for **the build**: decisions, copy, credentials, and captures that a builder or an agent needs in order to write code this week.

Open items about **the legacy data import** live in `docs/migration/field-map.md` section 20 and are deliberately kept separate. They are post-build concerns with a different audience and a different deadline, and mixing them here would put migration chores in front of agents who should be building surfaces. If you are working the migration lane, read both.

**Settled rulings** live in `DECISIONS.md`. This file tracks what is still open.

Organized by **who can answer it**, because that is what determines whether it can be closed before Monday.

**Where a default is stated, that default is what gets built if nobody answers.** Defaults are chosen to be safe and reversible, not to be right. Silence is an answer, just not a good one.

---

## What is genuinely blocking

1. **Contacts export** — from Customers & Leads, not CMS. Requires owner-level permissions the captain does not have; Tiffany must pull it. Without it there are no logins, no memberships, and no subscriber list. Filtered to the labels `Member Organization Representatives` and `Approved Area Needs Member`, roughly 150 rows, is enough. The full 2,321-contact export is only needed for `digest_subscribers`, which is not on the critical path.
2. ~~**Twelve email body texts**~~ — **closed 2026-08-16, eleven of twelve.** Staff, Member Organization, and Community email exports captured and merged into `docs/email/TEMPLATES.md` sections 4-6, with three content questions resolved as D51-D57 in `DECISIONS.md`. One body remains: `org_member_approved`, the membership-activation email referenced inside `org_approved`'s own copy (see `TEMPLATES.md` section 5 and section 8's open-captures table).
3. **Staff roster** — who gets `staff_admin` or `staff_approver`. Nobody can log into `/admin` until these memberships exist.
4. **Capture walkthrough** — roughly 145 observation items across eighteen bound surfaces (section C). Not decisions; just work.

~~Verify Replit actual DB connection details once live project is created (Neon vs. Helium)~~ — **closed as D58.** Staying on Replit-managed Postgres (Helium) is the deliberate answer, not a placeholder pending verification. Standard `pg` driver only; no Neon-specific client.

~~Test export~~ — **closed 2026-08-14.** All six collections exported and audited. Findings in `docs/migration/data-audit.md`.

**Monday morning rule.** Anything still open at kickoff gets resolved, proceeded past with the stated default, or logged as a known bug and built around. Nothing waits. A default that turns out wrong is a Wednesday fix; a lane that never starts is not.

---

## A. Captain decisions

All captain decisions through D49 are recorded in `DECISIONS.md`. See that file for the full list and reasoning. Do not reopen settled items here. D40 confirmed on the Aug 14 call (closes B5). D49 closes B9. O1 closed as D45. O3 answered: 48-hour target is being hit; literal confirmation-screen copy remains a capture item.

---

## B. Asks of other people

| # | Ask | From | Blocks | Notes |
|---|---|---|---|---|
| B1 | ~~**Test export**, sample from all six CMS collections~~ | Site owner | The whole migration lane | **Closed 2026-08-14.** All six collections exported and audited. See `docs/migration/data-audit.md`. |
| B2 | **Contacts export**, with labels and marketing subscriptions | Site owner | Member approval state, subscriber list | **Now the top blocker.** Lives in Customers & Leads, not CMS. The captain lacks the permission; Tiffany must pull it. A slice filtered to the labels `Member Organization Representatives` and `Approved Area Needs Member` (~150 rows) unblocks logins and memberships. The full export is only needed for the subscriber table, which is not critical path. Not reconstructable any other way. |
| B3 | ~~**Twelve email body texts**~~ **One remaining: `org_member_approved`'s body**, from the sending platform's editor | Site owner, owner-level access | Lane B copy for that one template | Eleven of twelve closed 2026-08-16, see `docs/email/TEMPLATES.md` sections 4-6. `org_member_approved`'s own spec entry already assumes it's kept and reworded for magic link rather than dropped — confirm that reading against the delivered body, don't assume it. Wix's triggered-email editor has no mobile preview; formal mobile capture stays waived for these 12 (does not answer B6/O7, which covers the 18 bound UI surfaces) |
| B4 | **Staff list with roles**, `staff_admin` or `staff_approver` | Executive director | All admin testing | Nobody can log into `/admin` until these memberships exist |
| B5 | **Login method notification** | Executive director | Nothing — decided | **Confirmed.** Magic link per D40. Tiffany reviewed the reasoning on the Aug 14 call and did not object |
| B6 | **Screenshot folder**, confirm mobile coverage | Executive director | Every bound surface | If desktop-only, mobile capture joins the walkthrough |
| B7 | **From address, display name, and the two staff notification addresses** | Executive director | Email dispatch | Environment variables, never hardcoded |
| B8 | **DNS access** for the `lia` subdomain | The Alliance | Deployment | |
| B9 | **Brand assets**, logo files | Executive director | Nothing — decided | **Decided.** Logos and related graphics in `assets/` per D49. `Design.md` covers the tokens |
| B10 | **Feature list** | Executive director | Nothing | Triaged to phase two by default |
| B11 | **Builder skill check:** Replit experience, prior AI-assisted building | Team | Lane assignment | |

**If B2 cannot be produced,** the fallback is staff re-approving the current member list by hand at ADMIN-03. That costs most of a day and needs scheduling Monday, not discovering Thursday.

---

## C. Capture walkthrough items

Each spec's section 15 is its own checklist. This is the map, plus the items that break testing if missed.

### The seven that matter most

| # | Item | Surface | Why |
|---|---|---|---|
| C1 | **Exact item summary format**, separator and spacing in `3x Blankets, 2x Pillows` | MP-13 | Now computed rather than stored, appears in an email too, and any difference is instantly visible in side-by-side |
| C2 | **Dropdown option format**, separator and date format | MP-04 | Members scan those lists by date |
| ~~C3~~ | ~~**Is the request contact name one input or two?**~~ | ~~MP-07~~ | **Closed.** Two inputs per D41 and `Handbook.md` section 8. Label capture stays open in MP-07 section 15 |
| ~~C4~~ | ~~Does MP-05 edit organization details, or only members?~~ | ~~MP-05~~ | **Closed, Aug 16.** Both, per D59. Inferred from MP-04 dashboard tile layout; reverses if direct MP-05 capture shows otherwise. |
| ~~C5~~ | ~~The two-week agreement checkbox: exists, verbatim text, gates submission~~ | ~~PB-02~~ | **Mostly closed, Aug 16.** Exists, text confirmed: "I agree to fulfill this request within the next 2 weeks." Whether it gates submission is the one piece still open. |
| ~~C6~~ | ~~Which location does the volunteer card show~~ | ~~PB-03~~ | **Closed, Aug 16.** Event location, confirmed against `event_location` and all eleven captured cards. |
| C7 | **Search debounce interval** | PB-01, PB-03 | Time it if no number is readable |

### Everything else, by surface

| Surface | Capture items | Notes |
|---|---|---|
| MP-01 | 3 | Open on copy capture only; magic link confirmed (D40) |
| MP-02 | 5 | Two strings written fresh per D9; captain reviews before ship |
| MP-03 | 7 | Twelve field labels. Mobile address and upload controls |
| MP-04 | 6 | Includes C2. Which statuses appear in each selector, sort order. partially closed Aug 16 |
| MP-05 | 6 | Scope closed per D59. Field labels and layout still `[CAPTURE]` |
| MP-06 | 4 | Success message wording matters; it must convey pending, not granted |
| MP-07 | 8 | C3 closed. Deadline option labels, dropoff field presence |
| MP-08 | 8 | Whether description, condition, product link appear here |
| MP-09 | 7 | Status option labels, which request fields are editable |
| MP-10 | 7 | What distinguishes description from details on screen |
| MP-11 | 7 | Do not harmonize copy against MP-08 |
| MP-12 | 7 | Current labels for interested and confirmed |
| MP-13 | 8 | Includes C1. Whether volunteer notes appear |
| PB-01 | 8 | Includes C7. Two empty states, captured independently |
| PB-02 | 6 | Includes C5. Region order, whether requested and remaining both show. partially closed Aug 16 |
| PB-03 | 6 | Includes C6, C7. partially closed Aug 16 |
| PB-04 | 10 | Notes field label and helper text; the prompt wording drives whether people use it |
| PB-05 | 4 | Whether the form collects anything beyond an email |
| ADMIN-01 … 08 | 0–1 | No screenshots. O2 still open in ADMIN-01, question sharpened after the Aug 14 call. D12's default stands |

**Total: roughly 145 items across eighteen surfaces.** All eighteen need a desktop and a mobile screenshot, which is 36 image files.

### Rules for whoever does the walkthrough

Copy and paste microcopy. Do not paraphrase. "Your request has been submitted for approval" and "Request submitted!" are not the same product.

Do not harmonize. If two surfaces say different things today, record both.

**Do not invent a value to fill a gap**, including as a human. Leave the `[CAPTURE]` marker. An empty marker is correctable; invented copy is not, because afterward nobody can tell which values were real.

Capture empty states independently where a surface has more than one.

---

## TE. Test export — CLOSED

All questions answered against the full export on 2026-08-14. Findings in `docs/migration/data-audit.md`; mappings updated in `docs/migration/field-map.md`.

| # | Question | Answer |
|---|---|---|
| TE1 | Per-item quantity arrays | Structure preserved. Parseable JSON |
| TE2 | Multi-reference fields | JSON arrays of record ids, not display names |
| TE3 | Structured addresses | Structured, five key shapes, every key optional |
| TE4 | Image references | Internal `wix:image://` refs, matching the manifest |
| — | Deadline Type | 3 values. Source misspells it `Until Fufilled` on 78 rows |
| — | Item Condition | 12 source variants collapsing to 3 |
| — | Need Status | Active / Pending / Archived |
| — | Primary Population Served | 24 distinct values. Three near-duplicate pairs open as O9 |
| TE5 | Donor rows with no quantity array | **10**, affecting 19 items with no recoverable quantity |
| TE6 | Rows with both references | 0 |
| TE7 | Rows with neither | 6 |
| TE8 | Rows with no email | 0 |
| TE9 | Names not splitting into two tokens | 43 raw, collapsing to 16 flagged after dedup |
| TE10 | Org name in contact first_name | **Still open.** Only answerable from the contacts export |

O9 is unblocked: the 24 values are listed in `data-audit.md` section 4. Default if nobody rules is to seed all 24 unmerged.

---

## V. Build verification items

Not questions. Things a builder must confirm the code actually does, extracted from decisions where they were buried in prose. Lane C owns all three.

| # | Verify | From | If missed |
|---|---|---|---|
| V1 | ADMIN-02's approve action writes `approved_at = now()` and `approved_by` in the same transaction as the status change | D48 | Every request approved during cutover week is invisible to the first real digest |
| V2 | The organization disable action does not clear `approved_at` or `approved_by` | D44 | Historical approval record lost; annual reporting undercounts |
| V3 | Disable is exposed on whichever admin surface manages already-approved organizations, not only the pending queue | D44 | Christina cannot disable a non-renewing org without a database write |

---

## E. Tracking

Suggested workflow, since this file is the one place all of it lives:

Mark each item as it closes, in this file, with the answer inline. Do not close an item by editing the spec alone; the spec is where the answer goes, and this file is where the team sees that it went somewhere.

When a `[CAPTURE]` is filled, update that spec's section 15 table and delete the marker in the body. A spec whose section 15 is empty is marked **Spec status: Complete** in its header block, and that is the signal a builder can pick it up without asking anything.
