# OPEN-ITEMS.md

Every open question in the corpus, in one place. Sourced from all twenty-six specs, `Handbook.md`, `DECISIONS.md`, `docs/email/TEMPLATES.md`, and `docs/migration/field-map.md`.

**Settled rulings** live in `DECISIONS.md`. This file tracks what is still open.

Organized by **who can answer it**, because that is what determines whether it can be closed before Monday.

**Where a default is stated, that default is what gets built if nobody answers.** Defaults are chosen to be safe and reversible, not to be right. Silence is an answer, just not a good one.

---

## What is genuinely blocking

In priority order. Nothing else stops a lane.

1. **Test export** — sample from all six CMS collections. Unblocks migration field-type questions (section TE) and population distinct values (O9). Critical path.
2. **Contacts export** — with labels and marketing subscriptions. Without it, member approval state and the subscriber list cannot be migrated.
3. **Twelve email body texts** — from the sending platform's editor. Subjects are known; bodies have never been read by anyone on this team.
4. **Staff roster** — who gets `staff_admin` or `staff_approver` on the platform-owner organization. Nobody can log into `/admin` until these memberships exist.
5. **Capture walkthrough** — roughly 150 observation items across eighteen bound surfaces (section C). Not decisions; just work.

---

## A. Captain decisions

All captain decisions through D49 are recorded in `DECISIONS.md`. See that file for the full list and reasoning. Do not reopen settled items here. D40 confirmed on the Aug 14 call (closes B5). D49 closes B9. O1 closed as D45. O3 answered: 48-hour target is being hit; literal confirmation-screen copy remains a capture item.

---

## B. Asks of other people

| # | Ask | From | Blocks | Notes |
|---|---|---|---|---|
| B1 | **Test export**, sample from all six CMS collections | Site owner | The whole migration lane | Critical path. See section TE |
| B2 | **Contacts export**, with labels and marketing subscriptions | Site owner | Member approval state, subscriber list | Not in the CMS. Not reconstructable any other way |
| B3 | **Twelve email body texts** from the sending platform's editor | Site owner, owner-level access | Lane B copy | Subjects are known. Bodies have never been read by anyone on this team. Tiffany flagged on the Aug 14 call that one of the 12 triggered emails (new-user-login approval) may be obsolete now that magic link is decided (D40, confirmed). Check against the delivered body text once it arrives — if obsolete, the count drops to 11 and `docs/email/TEMPLATES.md` needs the removal noted, not silently dropped. Wix's triggered-email editor has no mobile preview for any of the 12 templates; formal mobile capture is waived for these 12 specifically. This does not answer B6/O7, which covers the 18 bound UI surfaces and remains open |
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
| C4 | **Does MP-05 edit organization details, or only members?** | MP-05 | Changes the shape of the whole surface |
| C5 | **The two-week agreement checkbox:** exists, verbatim text, gates submission | PB-02 | Organizations plan around the expectation it sets |
| C6 | **Which location does the volunteer card show:** event location, organization city, or both | PB-03 | Not interchangeable. A volunteer choosing a Saturday cares about the event |
| C7 | **Search debounce interval** | PB-01, PB-03 | Time it if no number is readable |

### Everything else, by surface

| Surface | Capture items | Notes |
|---|---|---|
| MP-01 | 3 | Open on copy capture only; magic link confirmed (D40) |
| MP-02 | 5 | Two strings written fresh per D9; captain reviews before ship |
| MP-03 | 7 | Twelve field labels. Mobile address and upload controls |
| MP-04 | 8 | Includes C2. Which statuses appear in each selector, sort order |
| MP-05 | 7 | Includes C4, which gates the rest |
| MP-06 | 4 | Success message wording matters; it must convey pending, not granted |
| MP-07 | 8 | C3 closed. Deadline option labels, dropoff field presence |
| MP-08 | 8 | Whether description, condition, product link appear here |
| MP-09 | 7 | Status option labels, which request fields are editable |
| MP-10 | 7 | What distinguishes description from details on screen |
| MP-11 | 7 | Do not harmonize copy against MP-08 |
| MP-12 | 7 | Current labels for interested and confirmed |
| MP-13 | 8 | Includes C1. Whether volunteer notes appear |
| PB-01 | 8 | Includes C7. Two empty states, captured independently |
| PB-02 | 8 | Includes C5. Region order, whether requested and remaining both show |
| PB-03 | 8 | Includes C6, C7 |
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

## TE. Questions the test export answers

One sample export, roughly an hour, and it closes more open items than anything else on this page.

### The four at-risk field types

| # | Question |
|---|---|
| TE1 | **Per-item quantity arrays.** Structure preserved, or flattened to text? |
| TE2 | **Multi-reference fields.** What delimiter, and do they hold record ids or display names? |
| TE3 | **Structured addresses.** Sub-parts, or a flattened display string? `city` is required on every approved organization |
| TE4 | **Image references.** Internal references or resolvable URLs, and do they match the harvest manifest? |

### Four distinct-value scans, ten minutes

Deadline Type. Item Condition. Need Status. Primary Population Served.

Each maps to a constrained column. Without the actual values, the import defaults and logs, and 300 exception rows is a different week than 5.

### Four counts from the Donors collection

| # | Count | Why it matters |
|---|---|---|
| TE5 | Rows with no quantity array | **Highest data-quality risk.** Lines default to quantity 1, so recomputed counters understate claimed quantities and public remaining counts are visibly wrong on day one |
| TE6 | Rows with both an item and a volunteer request reference | The prose rule the old database could not enforce. Each needs human resolution |
| TE7 | Rows with neither reference | Same |
| TE8 | Rows with no email | Sizes the review queue. Ten-minute job or two-hour job |

TE10. Rows whose first_name exactly matches an organization's name (join against the Organizations collection) rather than a plausible personal name. This is the corruption pattern from a bulk contact update Tiffany described — roughly 1,200 records affected in the live Wix system when an org-linked contact was overlaid, and the org's business name landed in the person's first_name field. This count sizes a manual scrub the team is doing on the contacts CSV before import — it is diagnostic, not a gate on an automated cleanup step. Report the count; do not auto-correct.

### One count from Organizations

TE9. How many records arrive with a name that does not split into exactly two tokens. Same purpose as TE8.

---

## E. Tracking

Suggested workflow, since this file is the one place all of it lives:

Mark each item as it closes, in this file, with the answer inline. Do not close an item by editing the spec alone; the spec is where the answer goes, and this file is where the team sees that it went somewhere.

When a `[CAPTURE]` is filled, update that spec's section 15 table and delete the marker in the body. A spec whose section 15 is empty is marked **Spec status: Complete** in its header block, and that is the signal a builder can pick it up without asking anything.
