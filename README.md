# Love in Action / Area Needs Database

A marketplace connecting The Alliance's member organizations (Roseville, CA) to the public. Organizations post material and volunteer needs. The public claims items or expresses interest in roles. Alliance staff approve organizations, members, and requests before anything is public.

This repo is the **build contract and source assets** for the August 17–21, 2026 rebuild. Application code is not in this tree yet.

**Start here:** [`Handbook.md`](Handbook.md) is the build contract. [`replit.md`](replit.md) is the agent rules file — read it first on every task.

---

## Where to look

| File | Role |
|---|---|
| [`Handbook.md`](Handbook.md) | Authoritative build contract: model, invariants, surfaces |
| [`replit.md`](replit.md) | Rules that are never broken; where to look for everything else |
| [`DECISIONS.md`](DECISIONS.md) | Numbered decisions (D1–…) that specs and schema must honor |
| [`OPEN-ITEMS.md`](OPEN-ITEMS.md) | Captures and questions still unresolved |
| [`SPRINT.md`](SPRINT.md) | Week schedule and coordination — for people, not for building |
| [`docs/Design.md`](docs/Design.md) | Color, type, and card tokens from the live site |
| [`docs/specs/{ID}.md`](docs/specs/) | One surface per file (MP, PB, ADMIN) |
| [`docs/screenshots/`](docs/screenshots/) | Desktop and mobile captures — **folder exists, files not yet added** |
| [`docs/email/TEMPLATES.md`](docs/email/TEMPLATES.md) | Twelve email templates |
| [`docs/migration/field-map.md`](docs/migration/field-map.md) | Source export → target schema |
| [`migrations/`](migrations/) | Numbered SQL. `0001` is the complete initial schema |
| [`assets/`](assets/) | Logos, page headers, member-dashboard graphics |

`schema.sql` (current schema as a single `pg_dump` file) is referenced by the handbook and is not in the repo yet.

---

## Surfaces

Twenty-six surfaces. Specs are in `docs/specs/`. Bound public and member-portal surfaces also need screenshots in `docs/screenshots/` before they are built.

**Member portal (MP-01–MP-13)** — login, signup, dashboard, requests, supporters.

**Public browse (PB-01–PB-05)** — item and volunteer browse/detail, digest subscribe.

**Staff admin (ADMIN-01–ADMIN-08)** — approval queues, people review, populations, email log, audit, subscribers. Specs are the design; no screenshots.

---

## File tree

```
Alliance_LIADB_Build/
├── README.md
├── Handbook.md
├── replit.md
├── DECISIONS.md
├── OPEN-ITEMS.md
├── SPRINT.md
├── assets/
│   ├── alliance-logo-blue.png
│   ├── alliance-logo-gradient.png
│   ├── headers/
│   │   ├── LIA Email Header.png
│   │   ├── LIA Main Page Header-no words.jpg
│   │   ├── LIA Main Page Header.png
│   │   ├── Provide an Item Header.png
│   │   └── Volunteer your Time Header.png
│   └── member_dashboard_graphics/
│       ├── Member Dashboard AMC Login.png
│       ├── Member Dashboard Donors-Volunteers.png
│       ├── Member Dashboard Header.png
│       ├── Member Dashboard Item Request.png
│       ├── Member Dashboard Volunteer Request.png
│       ├── Member Dashboard-Add Users.png
│       └── My Org.png
├── docs/
│   ├── Design.md
│   ├── email/
│   │   └── TEMPLATES.md
│   ├── migration/
│   │   └── field-map.md
│   ├── screenshots/                          (empty)
│   └── specs/
│       ├── _TEMPLATE.md
│       ├── ADMIN-01.md                       Organization approval queue
│       ├── ADMIN-02.md                       Request approval queue
│       ├── ADMIN-03.md                       Member approval queue
│       ├── ADMIN-04.md                       People review queue
│       ├── ADMIN-05.md                       Populations management
│       ├── ADMIN-06.md                       Email log
│       ├── ADMIN-07.md                       Audit trail
│       ├── ADMIN-08.md                       Digest subscribers
│       ├── MP-01.md                          Login
│       ├── MP-02.md                          Global navigation and post-login routing
│       ├── MP-03.md                          Organization sign-up
│       ├── MP-04.md                          Organization dashboard
│       ├── MP-05.md                          Organization settings and member management
│       ├── MP-06.md                          Add a database user
│       ├── MP-07.md                          Item request, create
│       ├── MP-08.md                          Add items
│       ├── MP-09.md                          Item request, edit
│       ├── MP-10.md                          Volunteer request, create
│       ├── MP-11.md                          Add roles
│       ├── MP-12.md                          Volunteer request, edit
│       ├── MP-13.md                          View donors and volunteers
│       ├── PB-01.md                          Browse item requests
│       ├── PB-02.md                          Item request detail and claim
│       ├── PB-03.md                          Browse volunteer requests
│       ├── PB-04.md                          Volunteer request detail and interest
│       └── PB-05.md                          Digest subscribe and unsubscribe
└── migrations/
    └── 0001_initial_schema.sql
```

Captured August 14, 2026.
