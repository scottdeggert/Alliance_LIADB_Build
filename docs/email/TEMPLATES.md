# docs/email/TEMPLATES.md

Twelve templates. Subject lines below are captured verbatim from the current system and are bound. Body copy is not yet captured; see section 3.

Read `Handbook.md` section 13 first. This file is the implementation detail, not the policy.

---

## 1. A defect in the dedup index, which blocks this lane

`email_log_once_idx` is unique on `(template_key, entity_type, entity_id)` where `entity_id is not null and status <> 'failed'`.

Five of the twelve templates send to more than one recipient for the same entity:

- The four staff notifications each go to two addresses, a primary staff contact and a general information address.
- `org_request_approved` goes to the organization's primary contact and to the request's creator.

Under the current index, the first row inserts and **the second is rejected by the database**. One recipient gets the email. The other silently gets nothing, and the failure looks like a constraint violation rather than a missing notification.

This is not an edge case. It affects every organization submission, every request submission, every member invitation, and every request approval, which is most of the traffic in the system.

**Recommended fix: add `to_email` to the index.**

```sql
drop index email_log_once_idx;

create unique index email_log_once_idx
  on email_log (template_key, entity_type, entity_id, lower(to_email))
  where entity_id is not null and status <> 'failed';
```

This preserves what the index is actually for, which is once per recipient per entity, and it keeps the idempotent-approval behavior in `Handbook.md` section 12 intact: approving twice still sends each person one email.

**Ratified as D32** and folded into `migrations/0001_initial_schema.sql`.

---

## 2. Shared behavior

**Dispatch.** Provider is Resend. Every send writes an `email_log` row at `queued` before dispatch and updates it to `sent` with the provider message id, or to `failed` with the error. Sends never block a user-facing response: queue inside the transaction, respond, dispatch after.

**Sender.** From address and display name in environment variables. Reply-to is set per template; see each entry. The sending domain needs SPF, DKIM, and DMARC verified before any real send, per `Handbook.md` section 18.

**Variable resolution.** Every variable resolves before send. A template that would render an empty name, an empty organization, or a literal placeholder **does not send**: log it `failed` with the reason and surface it at ADMIN-06. This rule is the direct fix for the blank-name failure in `Handbook.md` section 16 and it applies without exception. A missing optional value means the surrounding line is omitted, not that the line renders with a blank.

**Links.** Every link is an absolute URL on the app's own domain, built from an environment variable. No relative paths, no hardcoded hosts. Staff notification links go to the specific admin queue, not to a generic admin landing.

**Plain text.** Every template ships an HTML part and a plain-text part. The text part is written, not auto-stripped from the HTML.

**Styling.** Minimal. Navy and white from `Design.md`, system font stack rather than the web fonts, single column, no background images. Email clients are not browsers and the design tokens are a reference here rather than a specification.

**Recipients.** Staff addresses come from environment variables, never hardcoded. Everyone else resolves from the database.

**No unsubscribe footer on these twelve.** All are transactional. The digest, which needs one, is phase two.

---

## 3. Body copy is not captured, and this is a gap

Subject lines below are verbatim from the current system's documentation. **The bodies are not.** They live in the current platform's triggered-email editor, not in any code file or in the 22-page report, which means nobody on this team has seen them.

They need capturing the same way surfaces do: open each template in the sending platform's dashboard and copy the body text exactly. That is a different place from the surface walkthrough, likely requires owner-level access, and it is not on the pre-Monday ask list yet. **Add it.**

Until captured, every body below is specified by what it must contain rather than by its wording. Build against the requirements, drop in the captured copy when it arrives. Do not write finished marketing prose and let it ship as though it were the original.

Tiffany flagged on the Aug 14 call that one of the 12 triggered emails (new-user-login approval) may be obsolete now that magic link is decided (D40, confirmed). Check against the delivered body text once it arrives (B3). If obsolete, the count drops to 11 and the removal is noted here, not silently dropped.

**No formal mobile capture for these 12 (B3).** Wix's triggered-email editor has no mobile preview for any of the 12 templates; formal mobile capture is waived for these 12 specifically. This does not answer B6/O7, which covers the 18 bound UI surfaces and remains open.

---

## 4. Staff notifications

Reply-to: the sending address. Recipients: both staff addresses, from environment variables.

### `staff_new_org`

| | |
|---|---|
| **Trigger** | Organization submitted at MP-03 |
| **Entity** | `organization` / the new organization's id |
| **Subject** | `Organization Pending Approval: {organizationName}` |
| **Variables** | organizationName, primaryContactName, primaryContactEmail, city, adminUrl |

Body must contain: the organization name, the primary contact's name and email, the city, and a link to ADMIN-01. `[CAPTURE]` body.

### `staff_new_item_request`

| | |
|---|---|
| **Trigger** | Item request submitted at MP-08, or moved to pending at MP-09 |
| **Entity** | `item_request` / the request id |
| **Subject** | `Item Request Pending Approval: {itemRequestName}` |
| **Variables** | itemRequestName, organizationName, itemCount, adminUrl |

Body must contain: the request title, the submitting organization, how many items it asks for, and a link to ADMIN-02. `[CAPTURE]` body.

### `staff_new_volunteer_request`

| | |
|---|---|
| **Trigger** | Volunteer request submitted at MP-11, or moved to pending at MP-12 |
| **Entity** | `volunteer_request` / the request id |
| **Subject** | `Volunteer Request Pending Approval: {volunteerRequestName}` |
| **Variables** | volunteerRequestName, organizationName, roleCount, adminUrl |

Body must contain: the request title, the organization, how many roles, and a link to ADMIN-02. `[CAPTURE]` body.

### `staff_new_user`

| | |
|---|---|
| **Trigger** | Member invited at MP-06 |
| **Entity** | `org_membership` / the new membership id |
| **Subject** | `New Member Pending Approval: {memberName}` |
| **Variables** | memberName, memberEmail, organizationName, invitedByName, adminUrl |

Body must contain: who was invited, their email, which organization, who invited them, and a link to ADMIN-03. `[CAPTURE]` body.

The current version links to the person's record in the old contacts system. The replacement links to ADMIN-03, which is where the approval now happens.

---

## 5. Organization notifications

Reply-to: the staff primary contact address, so a member replying reaches a person.

### `org_approved`

| | |
|---|---|
| **Trigger** | Organization approved at ADMIN-01 |
| **Recipient** | Primary contact |
| **Entity** | `organization` / the organization id |
| **Subject** | `Welcome to the Love in Action Database {organizationName}` |
| **Variables** | organizationName, contactFirstName, loginUrl |

Body must contain: a welcome, the organization name, how to log in, and a link. `[CAPTURE]` body.

**This is the highest-consequence email in the system.** It is how an approved organization learns it can start posting. A silent failure leaves them waiting indefinitely with no signal anywhere they can see. ADMIN-01's result message states the failure explicitly for this reason.

**Login method: magic link (D40, confirmed Aug 14).** Body copy describes requesting a link at `/login`, not entering a password. Tiffany reviewed the reasoning and did not object.

### `org_request_received`

| | |
|---|---|
| **Trigger** | Request submitted at MP-08 or MP-11 |
| **Recipient** | The submitting member |
| **Entity** | `item_request` or `volunteer_request` / the request id |
| **Subject** | `{itemOrVolunteer} Request Pending Approval: {requestName}` |
| **Variables** | itemOrVolunteer, requestName, organizationName, submitterFirstName |

`itemOrVolunteer` renders as the word the current subject line uses for each type. `[CAPTURE]` both exact words; they appear in the subject and getting them wrong is visible in every member's inbox.

Body must contain: confirmation the request was received, that staff review it before it goes public, and what happens next. `[CAPTURE]` body.

### `org_request_approved`

| | |
|---|---|
| **Trigger** | Request approved at ADMIN-02 |
| **Recipients** | Organization's primary contact **and** the request's creator. One email each. If they are the same person, one email total |
| **Entity** | `item_request` or `volunteer_request` / the request id |
| **Subject** | `Your Love in Action Request was Approved!` |
| **Variables** | requestName, organizationName, recipientFirstName, publicUrl |

Body must contain: the request title, that it is now public, and a link to the public page so the organization can share it. `[CAPTURE]` body.

The public link is worth confirming exists in the captured body. Organizations share these to their own donor networks, and a link in the approval email is the moment they are most likely to do it.

**Dedup:** depends on the index fix in section 1. Without it the second recipient gets nothing.

### `org_member_approved`

| | |
|---|---|
| **Trigger** | Membership approved at ADMIN-03 |
| **Recipient** | The new member |
| **Entity** | `org_membership` / the membership id |
| **Subject** | `Love in Action Database Login Info for {memberName}` |
| **Variables** | memberName, organizationName, loginUrl, dashboardUrl |

Body must contain: that they have been approved, which organization, how to log in, and a link. `[CAPTURE]` body.

Second-highest consequence after `org_approved`, and it fails the same way: the person can log in but has no way to find out. ADMIN-03's result message states a failure explicitly.

Login wording describes requesting a link at `/login`, not entering a password (D40, confirmed Aug 14).

### `org_new_item_donation`

| | |
|---|---|
| **Trigger** | Item pledge recorded at PB-02 |
| **Recipient** | The request's contact person |
| **Entity** | `item_pledge` / the pledge id |
| **Subject** | `Item(s) have been donated for {requestName}` |
| **Variables** | requestName, donorName, donorEmail, donorPhone, itemSummary, supportersUrl |

`itemSummary` uses the same computed format as MP-13: quantity, `x`, space, item name, comma-separated. Compute it once, in one place, shared with MP-13, so the email and the screen never disagree.

Body must contain: which request, who pledged, what they pledged with quantities, their contact details, and a link to MP-13. `[CAPTURE]` body.

`entity_type` is `item_pledge`, which is not in the `approval_events` check constraint and does not need to be; `email_log.entity_type` is an unconstrained text column. Each pledge is distinct, so dedup here prevents a double-send on a retry rather than blocking legitimate repeats.

### `org_new_volunteer`

| | |
|---|---|
| **Trigger** | Volunteer signup recorded at PB-04 |
| **Recipient** | The request's contact person |
| **Entity** | `volunteer_signup` / the signup id |
| **Subject** | `A Volunteer has Expressed Interest in Serving` |
| **Variables** | requestName, volunteerName, volunteerEmail, volunteerPhone, roleList, notes, supportersUrl |

Body must contain: which request, who expressed interest, which roles, their contact details, **their notes**, and a link to MP-13. `[CAPTURE]` body.

The notes field carries availability, experience, and any accommodation the volunteer needs. It must appear in this email in full, not truncated. If the captured body omits it, add it and note the addition, because this is the email the organization acts on and the accommodation is the part they most need to read before making contact.

Subject line stays as captured (D30). Adding the request title would improve inbox distinguishability but is a bound-artifact deviation deferred to phase two.

---

## 6. Supporter confirmations

Reply-to: the requesting organization's contact email, so a donor replying reaches the organization directly rather than staff.

### `donor_item_confirmation`

| | |
|---|---|
| **Trigger** | Item pledge recorded at PB-02 |
| **Recipient** | The person who pledged |
| **Entity** | `item_pledge` / the pledge id |
| **Subject** | `Thank you for donating item(s) to {organizationName}` |
| **Variables** | donorFirstName, organizationName, itemSummary, orgContactName, orgContactEmail, orgContactPhone, dropoffLocation, requestUrl |

Body must contain: thanks, what they pledged with quantities, **the organization's contact information**, and the dropoff location when the request has one. `[CAPTURE]` body.

The organization's contact details are required, per `Handbook.md` section 13. This email is the only channel a donor has to ask a question, change a quantity, or cancel, because there is no path in the app to modify a pledge. If the contact details are missing, the donor has nowhere to go.

If the captured body states a fulfillment window, keep it exactly. It corresponds to the agreement checkbox at PB-02 and organizations plan around it.

### `donor_volunteer_confirmation`

| | |
|---|---|
| **Trigger** | Volunteer signup recorded at PB-04 |
| **Recipient** | The person who signed up |
| **Entity** | `volunteer_signup` / the signup id |
| **Subject** | `Thank you for expressing interest in volunteering!` |
| **Variables** | volunteerFirstName, organizationName, requestName, roleList, orgContactName, orgContactEmail, orgContactPhone, followUpWindow, requestUrl |

Body must contain: thanks, which roles they expressed interest in, that the organization will follow up, **the stated follow-up window**, and the organization's contact details. `[CAPTURE]` body.

The follow-up window is a promise the organization keeps. Christina confirmed on the Aug 14 call that the 48-hour target is being hit in practice (O3); no change to the stated window. Capture the exact period and wording from both this template and PB-04's on-screen confirmation, and make sure they agree. If they disagree today, that is a defect worth fixing rather than a bound inconsistency worth preserving; take it to the captain. Literal copy is still `[CAPTURE]`.

This email records interest, not a booking. The wording should not read as a confirmed shift.

---

## 7. Testing

Verified against a live inbox before the build is done, per `Handbook.md` section 18. For each of the twelve:

- Renders in HTML and in plain text
- Every variable resolves, with no placeholder text and no blank name
- Every link resolves to the right entity on the right host
- Arrives in a real inbox, not spam, from the verified domain
- Writes exactly one `email_log` row per recipient, at `sent`

Then the negative tests:

- A send with an unresolvable variable does not go out and logs `failed` with a readable reason at ADMIN-06. This is one of the five tests in `Handbook.md` section 16.
- Approving the same entity twice produces one email per recipient, not two.
- A multi-recipient template produces one row per recipient. **This is the test that catches the index defect in section 1 if the fix has not been applied.**

---

## 8. Open captures

| What is needed | Source |
|---|---|
| **All twelve body texts, verbatim** | The sending platform's triggered-email editor. Owner-level access. **Add to the pre-Monday ask list.** On arrival, check whether the new-user-login approval template is obsolete under D40 (B3); if it is, note the removal here rather than dropping it silently |
| Formal mobile capture of the 12 templates | **Waived** per B3. Does not answer B6/O7 |
| The two words `itemOrVolunteer` renders as | Same source |
| Whether the follow-up window in `donor_volunteer_confirmation` matches PB-04's on-screen text | Capture both, then captain if they disagree. O3 confirmed the 48-hour operational target; literal wording is still capture |
| From address, display name, and staff recipient addresses | Executive director |
