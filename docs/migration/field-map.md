# docs/migration/field-map.md

Source-export structure lives here and nowhere else. No source field name appears in application code.

Read `Handbook.md` section 14 first for policy. This file is the mapping. The person identity and name policy in `Handbook.md` section 8 governs section 4 below.

---

## 1. Sources

**Export A, the CMS.** Six collections, 676 rows.

| Collection | Rows | Feeds |
|---|---|---|
| Organizations | 48 | `organizations`, `people`, `populations` |
| Needs Requests | 100 | `item_requests`, `people` |
| Items | 336 | `items` |
| Volunteer Requests | 23 | `volunteer_requests`, `people` |
| Volunteer Roles | 58 | `volunteer_roles` |
| Donors | 111 | `item_pledges`, `item_pledge_lines`, `volunteer_signups`, `volunteer_signup_roles`, `people` |

**Export B, contacts.** Not part of the CMS export and separately requested. Feeds `people`, `users`, `org_memberships`, and `digest_subscribers`. Carries the member records, the approval label, and the digest subscription flag. **Without it, no migrated member has an active membership and the subscriber list is lost.** See `SPRINT.md`.

**Export C, media.** `media_manifest.csv` from the pre-sprint harvest, keyed on `legacy_wix_id`. 56 images across 57 active request pages. Historical pending and archived records were never publicly reachable and have no captured image; that gap is accepted.

---

## 2. Order and dependency

```
populations
  → people          (consolidated from 4 sources, section 4)
    → users         (contacts export only)
      → organizations
        → org_memberships
        → organization_populations
          → item_requests → items
          → volunteer_requests → volunteer_roles
            → item_pledges → item_pledge_lines
            → volunteer_signups → volunteer_signup_roles
              → recompute counters
              → digest_subscribers
              → seed email_log (section 9)
```

Every step is idempotent on `legacy_wix_id`. Re-running the import against a partially loaded database updates rather than duplicates. This is not optional: the migration will be run more than once.

---

## 3. Conventions

- **`legacy_wix_id`** carries the source record id on every table that receives rows. Never a foreign key.
- **Dropped fields** are recorded in `dropped-fields.csv`, one row per source field with a reason. That file is a deliverable.
- **Assumptions** go in `migration-exceptions.csv`, one row per record where the import made a choice the source did not determine. Also a deliverable. This is how a defaulted quantity or a coerced status stays visible instead of disappearing into the data.
- **No guessing into the database.** Where the source is ambiguous about a person, the row lands with `needs_review = true`. Where it is ambiguous about anything else, it goes in the exceptions file.

---

## 4. `people`, the hard part

One human is one row. The source has the same human in up to four places with no key connecting them. The person identity and name policy in `Handbook.md` section 8 governs this section. The same rules apply to concatenated primary-contact names on Organizations, Item Requests, and Volunteer Requests, not only to Donor names.

**Sources, in this order:**

1. **Contacts export.** Every contact. Has first and last name as separate fields, plus email. The cleanest source; load it first so later passes match against it.
2. **Organizations.** `Primary Contact Name` (single combined string), `Primary Contact Email`, `Primary Contact Phone Number`.
3. **Needs Requests and Volunteer Requests.** Same three fields, per request. The same human appears once per request they contacted for.
4. **Donors.** `Name` (single combined string), `Email`, `Phone Number`.

**Matching:** `lower(email)`, exact. No fuzzy matching on names, ever. Two people named J. Martinez at the same organization are two rows until a human says otherwise at ADMIN-04.

**Name splitting.** Sources 2, 3, and 4 give one string. Migration never guesses a confident split.

| Pattern | Action |
|---|---|
| Two tokens, no suffix or particle ambiguity | First token to `first_name`, second to `last_name` |
| One token | Best-effort split: both columns populated, `needs_review = true`, original in `source_note`, `review_note` states no last name was present |
| Three or more tokens | First token to `first_name`, remainder to `last_name`, `needs_review = true`, original in `source_note`, `review_note` states the name did not split confidently |
| Contains a comma | Do not attempt to reverse it. Best-effort split, `needs_review = true`, original in `source_note`, `review_note` states the name did not split confidently |
| Hyphenated surnames, suffixes (Jr, Sr, II, III), or particles (van, de, del) | Does not split confidently. Best-effort split, `needs_review = true`, original in `source_note`, `review_note` states why |
| Non-personal contact value (office name, department, etc.) | Both columns populated with best effort, `needs_review = true`, original in `source_note`, `review_note` states a person's name was expected |
| Empty or whitespace | `needs_review = true`, both names set to a stated placeholder, original in `source_note`, `review_note` states the source was empty |

The contacts export has a known corruption pattern: `first_name` exactly matching an organization's name, from a bulk contact update that overlaid an org-linked contact and wrote the business name into the person's first_name field. Roughly 1,200 live Wix records. **Count these (TE10). Report the count. Do not auto-correct.** The count sizes a manual scrub of the contacts CSV before import; it is diagnostic, not a gate on an automated cleanup step.

Every split that sets `needs_review` writes the original string into `source_note` verbatim. `review_note` gets a plain-language reason the operator can act on. The reviewer at ADMIN-04 needs to see what arrived and why it was flagged, not only what the import made of it.

**Records with no email.** `people.email` is `not null` and uniquely indexed on `lower(email)`. A source record with no email cannot be inserted as-is.

**Specified handling:** synthesize `missing+{source}-{legacyId}@invalid.local`, set `needs_review = true`, write the source record reference in `source_note`, and write a plain-language reason in `review_note`. The `.invalid` TLD is reserved and can never route, so a synthesized address cannot accidentally receive mail. The reviewer merges the record into a real person or clears it.

`[CAPTURE]` how many such records exist. Run the count before Monday; it determines whether the review queue is a ten-minute job or a two-hour one.

**Conflicting details on match.** Same email, different name or phone across sources. Keep the contacts-export values, since they come from structured fields. Where the contacts export has no row, keep the first-loaded value and write the conflict to `migration-exceptions.csv`. Do not overwrite silently and do not create a second row.

---

## 5. `users` and `org_memberships`

Both come from Export B only. The CMS knows nothing about logins.

**`users`:** one row per contact that was a site member. `person_id` from the match in section 4. `auth_subject` is **null on import**; it populates at first login under the new auth provider. `status` set to `invited`. `last_login_at` null.

**`org_memberships`:** the source associates a member to an organization through a free-typed `organization-name` custom field matched character for character against the organization name.

| Source condition | Action |
|---|---|
| Field matches an organization name exactly | Create membership, `org_id` resolved |
| Matches case-insensitively or after trimming whitespace | Create membership, log to exceptions with both strings |
| No match | **No membership created.** Log to exceptions with the unmatched string. A human resolves it |
| Field empty | No membership. The contact becomes a `people` row and nothing more |

That third row is the point of the whole rebuild appearing as a migration problem: every unmatched string is a member who was silently broken in the old system.

**Role:** `owner` where the person is the organization's `Primary Contact Email`, `member` otherwise (D35). The source has no role concept; this is an inference.

**Status:** `active` where the contact carries the approval label, `pending` otherwise. **This label is the only record of who is approved and it exists only in Export B.**

**Staff.** Alliance staff memberships against the `platform_owner` organization are not migrated. They are seeded directly, with roles assigned by the captain. Section 11.

---

## 6. `organizations`

| Source field | Target | Notes |
|---|---|---|
| ID | `legacy_wix_id` | |
| Organization Name | `name` | Unique. Collisions to exceptions |
| — | `slug` | Generated: lowercase, non-alphanumerics to hyphens, collapsed, trimmed. Collisions get a numeric suffix. **Permanent once assigned** |
| Website URL | `website_url` | |
| Mission Statement | `mission` | |
| Primary Population Served | `organization_populations` | Section 7 |
| Logo | `logo_url` | Section 10 |
| Org Address | `address_line1`, `address_line2`, `city`, `state`, `postal_code`, `address_formatted` | Section 8 |
| Org Phone Number | `phone` | |
| Primary Contact Name / Email / Phone Number | `primary_contact_person_id` | Resolved via section 4 |
| Approved | `status` | `APPROVED` → `approved`, `PENDING` → `pending`. Any other value → `pending`, logged |
| Approved Email Sent | dropped | Section 9 |
| Owner | dropped | Documented as unused |
| Page Link fields | dropped | Routes computed from `slug` and `id` |
| Created Date | `created_at` | Preserve. Do not let the default overwrite it |
| Updated Date | `updated_at` | Preserve |

`approved_at` and `approved_by`: the source records neither. Set `approved_at` to `Updated Date` for approved organizations, `approved_by` null. Log the inference once in the exceptions file rather than per row. D43/D48 (leave `approved_at` null) apply to item and volunteer requests in the historical batch, not to organizations. Revisit organization `approved_at` only if it, or a derived "member since," is ever displayed publicly or used to sort or filter a directory.

**No `approval_events` rows are backfilled (D36).** The audit trail starts at cutover. Fabricating history with invented actors and timestamps is worse than an empty trail with a known start date.

---

## 7. `populations`

Seeded from `select distinct` on the source tags field across all 48 organizations. **Not from an invented taxonomy.**

Process: extract distinct values, trim, group case-insensitively, sort alphabetically, assign `sort_order` in that order, all `is_active = true`. Add an `Other` row if one is not already present.

Values that appear once, or that read as free text rather than a category, go to `organizations.populations_other` instead of becoming a population row, with the organization also assigned to `Other`. `[CAPTURE]` the distinct-value list before Monday and have the captain draw that line. It is a ten-minute review of maybe twenty values and it determines whether ADMIN-05's Other region opens with three entries or thirty.

---

## 8. Addresses

The source stores a structured address object with sub-parts and a formatted display string. Whether the export preserves the structure or flattens it to the display string is unknown until a test export is inspected.

**If structured:** map sub-parts to their columns directly, keep the display string in `address_formatted`.

**If flattened:** parse the formatted string. Do not attempt a general address parse. Extract `city`, `state`, and `postal_code` from the end of the string, which is reliable for US addresses in a standard format, and put the remainder in `address_line1`. Anything that does not fit the pattern goes to exceptions with `city` left null.

**`city` is required for every approved organization.** It renders as the location on both public browse surfaces. Any approved organization with a null `city` after import is a blocking validation failure, not a warning. There are 48 organizations; if the parse leaves gaps, fill them by hand.

This is one of the four at-risk field types. Test it on a sample export before any full run.

---

## 9. `email_log` seeding

The source carries `Approved Email Sent` as a boolean on organizations, which existed only to prevent double-sends.

**Seed an `email_log` row** at `status = 'sent'` for every organization where the flag is true: `template_key = 'org_approved'`, `entity_type = 'organization'`, `entity_id` the new id, `to_email` the primary contact, `payload` empty, `sent_at` the organization's `Updated Date`, and a note recording that it was migrated rather than dispatched.

Without this, the dedup index has no record of the send and a staff member touching an approved organization can re-welcome an organization that joined two years ago (D37).

---

## 10. Media

Source image fields hold an internal reference, not a URL. Public URLs follow `static.wixstatic.com/media/{mediaId}/v1/fill/...`, where everything from `/v1/` onward is a server-side transform; stripping it returns the original upload. Genuine uploads follow a `b770af_...~mv2.{ext}` pattern. Template graphics do not and are excluded.

`media_manifest.csv` holds the harvested originals keyed on `legacy_wix_id`.

**Process:** match each request and organization to its manifest entry, upload the file to the app's own storage (D38), write the resulting URL to `image_url` or `logo_url`.

**Do not write source-hosted URLs into the database (D38).** They are outside our control and they break whenever the source site changes. Rehost everything.

**Storage:** Replit object storage for the sprint (D39). Every read and write goes through a single storage adapter module so a later move to S3-compatible storage is one file plus a copy job over roughly 60 images. If a bucket already exists on an account The Alliance controls, use that instead and skip the migration later.

**Known gap:** roughly half the historical pending and archived records were never publicly reachable and have no harvested image. Those rows get a null `image_url`. Accepted; do not attempt to backfill.

Filename construction: media ids already carry an extension. Check before appending, or files land as `~mv2.jpg.jpg`.

---

## 11. Platform owner and staff

Not in any export. Created by the migration as a fixed step:

- One `organizations` row, `kind = 'platform_owner'`, `name` and `slug` for The Alliance, `status = 'approved'`.
- `people` rows for each staff member, matched against the contacts export where present.
- `users` rows for each.
- `org_memberships` against the platform owner at `status = 'active'`, roles per the captain: `staff_admin` or `staff_approver`.

`[CAPTURE]` the staff list and each person's role. Needed before anyone can log into `/admin`, which means it blocks all of Lane C's testing, not just the migration.

---

## 12. `item_requests` and `items`

### `item_requests`, from Needs Requests

| Source field | Target | Notes |
|---|---|---|
| ID | `legacy_wix_id` | |
| Organization (reference) | `org_id` | Resolved through `legacy_wix_id`. An unresolvable reference is a blocking failure |
| Request Title / Title | `title` | **Two fields, one target.** `[CAPTURE]` which is populated and whether they ever differ. Rule until known: prefer `Request Title`, fall back to `Title`, log any row where both are populated and differ |
| Description | `description` | |
| Image | `image_url` | Section 10 |
| Need Status | `status` | `Active` → `active`, `Pending` → `pending`, `Archived` → `archived`. Nothing imports as `draft`; that status is new. **All active requests migrate as-is (D42).** No pre-migration outreach to confirm they are still live, and no cleanup pass that drops older unmet requests |
| Deadline Type | `deadline_type` | Section 14 |
| Deadline Date | `deadline_date` | |
| Archive On | `expires_on` | Renamed |
| Quantity Helped | `people_helped` | |
| Primary Contact Name / Email / Phone Number | `contact_person_id` | Section 4 |
| Items (multi-reference) | not migrated | The child rows carry the parent reference; the multi-reference is redundant |
| Owner | dropped | |
| Page Link fields | dropped | |
| Created Date / Updated Date | `created_at` / `updated_at` | Preserve |

`submitted_at`, `approved_at`, `approved_by`, `archived_at`, `archived_reason`, `created_by`: not recorded in the source. **`approved_at` and `submitted_at` are left null on the one-time historical Wix migration batch only (D43, D48).** They are not carried over from source and are not derived from `Updated Date` or any other Wix field. This is not a general import default. Any later load — a second scripted import, a manual batch, cutover-week new posts — must set `approved_at` explicitly to a real, current timestamp. A fabricated historical date is dishonest, and using the historical-batch import timestamp would make every migrated request look newly approved when the send job runs. Set `archived_at` to `Updated Date` for archived requests; leave the rest null. `archived_reason` null for migrated rows, since the source does not distinguish manual from expired from fulfilled. Log the `archived_at` inference once.

### `items`

| Source field | Target | Notes |
|---|---|---|
| ID | `legacy_wix_id` | |
| Item Request (reference) | `item_request_id` | Unresolvable is a blocking failure |
| Item Name | `name` | |
| Item Description | `description` | |
| Item Condition | `condition` | Section 14 |
| Product Link | `product_url` | |
| Quantity | `quantity_requested` | Must be greater than zero. A zero or null value goes to exceptions with a defaulted 1 |
| Claimed Quantity | **not imported** | Recomputed. Section 16 |
| Remaining Quantity | dropped | Generated column |
| Received Quantity | `quantity_received` | Imported as-is. Not a counter |
| Donors (multi-reference) | not migrated | Reconstructed from the Donors collection |
| — | `sort_order` | Assigned by source order within the parent |

---

## 13. `volunteer_requests` and `volunteer_roles`

### `volunteer_requests`

Same mapping as item requests, including D42 (active requests migrate as-is) and D43/D48 (`approved_at` and `submitted_at` left null on the historical batch only), with these differences:

| Source field | Target | Notes |
|---|---|---|
| Title | `title` | One title field on this side |
| Details | `details` | Own column. Not merged into `description` |
| Event Location | `event_location` | |
| — | `deadline_date` | **Null for every migrated row.** The source has no such field; it is new per deviation one |
| Roles (multi-reference) | not migrated | Child rows carry the parent |

### `volunteer_roles`

| Source field | Target | Notes |
|---|---|---|
| ID | `legacy_wix_id` | |
| Volunteer Request (reference) | `volunteer_request_id` | |
| Role Name | `name` | |
| Role Description | `description` | |
| Quantity | `quantity_needed` | Greater than zero, same rule as items |
| Interested Quantity | **not imported** | Recomputed. Section 16 |
| Received Quantity | `quantity_confirmed` | Imported as-is |
| Claimed Quantity | dropped | Vestigial |
| Remaining Quantity | dropped | Generated |
| Manual Sort | `sort_order` | Text to integer. Unparseable values fall back to source order, logged |

---

## 14. Coded values

Three source fields hold text values the new schema constrains. **Run `select distinct` on each before writing the mapping.** The values below are expected, not confirmed.

**Deadline Type** → `date_specific`, `until_fulfilled`, `ongoing`. Expected source values include a date-specific variant and an until-fulfilled variant. Any unmapped value defaults to `until_fulfilled` on the item side and `ongoing` on the volunteer side, logged to exceptions.

**Item Condition** → `new`, `gently_used`, `any`. Expected source values include New and Gently Used. Null or unmapped defaults to `any`, logged.

**Need Status** → per section 12. Any unmapped value defaults to `archived` rather than `active`, so nothing unexpected becomes public. Logged.

The distinct-value scan is a ten-minute job against a test export and it is the difference between a clean import and 300 exception rows.

---

## 15. `item_pledges`, `volunteer_signups`, and their lines

The Donors collection holds both branches in one table, with a rule written in prose that a record has either an item request reference or a volunteer request reference but not both.

**Split rule, per row:**

| Condition | Action |
|---|---|
| Item Request reference present, Volunteer Request absent | `item_pledges` |
| Volunteer Request present, Item Request absent | `volunteer_signups` |
| Both present | **Neither.** Log to exceptions for human resolution. The database can now enforce what prose could not, and this is where the unenforced rule shows up |
| Neither present | Log to exceptions. The person is still created |

`[CAPTURE]` counts for rows three and four before Monday.

### Common fields

| Source field | Target |
|---|---|
| ID | `legacy_wix_id` |
| Name / Email / Phone Number | `person_id`, via section 4 |
| Notes | `notes` |
| Organization (reference) | not stored | The organization resolves through the request. Storing it again would let the two disagree |
| Title | dropped | System-generated |
| Created Date / Updated Date | `created_at` / `updated_at` |

### `item_pledge_lines`

From `Item to Quantity Array`, an array of maps carrying item id, name, and quantity. One line per entry: `item_id` resolved through `legacy_wix_id`, `quantity` as given.

**Where the array is empty or absent** but the Items multi-reference has entries, the source records which items without how many. Create one line per referenced item at **quantity 1** and write every one to `migration-exceptions.csv`. Do not skip the pledge; a donor who claimed something is a donor.

`[CAPTURE]` how many donor rows are in this state. If it is more than a handful, the recomputed counters will understate claimed quantities and the numbers on public request pages will be visibly wrong on day one. This is the single most likely source of a bad number in the migrated data.

`Donated Item Quantities` and `Multi Reference as String` are dropped. Both are denormalized display text and both are now computed.

### `volunteer_signup_roles`

From the Volunteer Roles multi-reference. One row per referenced role, `volunteer_role_id` resolved through `legacy_wix_id`. No quantity; interest is one spot per role per signup.

A referenced item or role that does not resolve is logged and skipped, and the parent pledge or signup still imports.

---

## 16. Counter recomputation

**Never import counter values from the source.** After all pledges, signups, and lines are loaded:

```sql
update items i set quantity_claimed = coalesce(l.total, 0)
from (select item_id, sum(quantity) total from item_pledge_lines group by item_id) l
where l.item_id = i.id;

update items set quantity_claimed = 0
where id not in (select item_id from item_pledge_lines);

update volunteer_roles r set quantity_interested = coalesce(s.total, 0)
from (select volunteer_role_id, count(*) total from volunteer_signup_roles group by volunteer_role_id) s
where s.volunteer_role_id = r.id;

update volunteer_roles set quantity_interested = 0
where id not in (select volunteer_role_id from volunteer_signup_roles);
```

`quantity_remaining` is generated and recomputes automatically.

**Compare the recomputed values against the source's stored counters** and write every difference to `migration-exceptions.csv`. Do not correct toward the source; the recomputed value is authoritative. The comparison exists because a large gap means pledge lines were lost, which is a migration defect rather than a data-quality finding.

---

## 17. `digest_subscribers`

From Export B only.

| Source | Target |
|---|---|
| Contact email | `email` |
| Subscription flag or label | `status`: `subscribed` or `unsubscribed` |
| — | `unsubscribe_token`, generated |
| — | `person_id`, left null on import (D27) |
| — | `legacy_source`, set to a value rendering as `Imported` at ADMIN-08 |
| Subscription date, if present | `subscribed_at`, else the import timestamp |

---

## 18. Validation

Every check runs and passes before the migration is accepted. Output is a written report, committed.

**Row counts, per table, exact:**

| Table | Expected |
|---|---|
| `organizations` | 48, plus 1 platform owner |
| `item_requests` | 100 |
| `items` | 336 |
| `volunteer_requests` | 23 |
| `volunteer_roles` | 58 |
| `item_pledges` + `volunteer_signups` | 111 minus any logged in section 15 rows three and four |
| `people` | Unknown until run. Report it; it is the headline number for the duplicate-collapse story |

**Referential:**
- Every request resolves to an existing organization
- Every item resolves to an existing item request
- Every role resolves to an existing volunteer request
- Every pledge line resolves to an item on the same request as its pledge
- Every signup role resolves to a role on the same request as its signup
- Every membership resolves to an existing user and organization

**Integrity:**
- Every counter equals the sum of its lines. The drift query in `schema.sql` returns zero rows
- Every approved organization has a non-empty `city`
- No duplicate emails in `people`, case-insensitive
- Every organization has a unique, non-empty `slug`
- Every source record maps to exactly one destination row, verified by `legacy_wix_id`
- No `people` row is referenced by zero rows and also carries a synthesized email; that combination is an import artifact with no purpose

**Deliverables:**
- `redirects.csv`, old public path to new full URL
- `dropped-fields.csv`
- `migration-exceptions.csv`
- The validation report itself
- The count of `people` rows flagged `needs_review`

---

## 19. Test before the full run

Four field types are where fidelity breaks. Test each on a sample export before any full run:

1. **Per-item quantity arrays.** Does the export preserve the array structure or flatten it to text?
2. **Multi-reference fields.** Delimiter, and whether they hold ids or display names.
3. **Structured addresses.** Structured or flattened. Section 8.
4. **Image references.** Internal references or resolvable URLs, and whether they match the harvest manifest.

A test export that answers these four questions is on the critical path and is worth more than any other pre-Monday task.

---

## 20. Open captures

| What is needed | Source |
|---|---|
| **Staff list and roles for the platform owner org** | Captain. Blocks all admin testing |
| **Test export answering the four questions in section 19** | Site owner, pre-Monday, critical path |
| Distinct values for Deadline Type, Item Condition, Need Status | Test export |
| Distinct population values, and where the free-text line falls | Test export, then captain |
| Count of donor rows with no quantity array | Test export. Highest data-quality risk |
| Count of donor rows with both or neither request reference | Test export |
| Count of records arriving with no email | Test export |
| Whether Request Title and Title ever differ | Test export |
