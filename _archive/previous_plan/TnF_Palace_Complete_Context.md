# T&F Palace App — Complete Context Document

> Full reference compiled from research, internal presentations, and employee notes.

---

## PART 1: THE HACKATHON BRIEF

### What You're Building

A mobile app that does for Taylor & Francis what the Palace Project does for public libraries — but for one publisher, across more content types (books, ebooks, journals, multimedia), working online and offline, plugging into T&F's content systems (mocked for now).

### The Three Stages

| Stage | When | What |
|---|---|---|
| Research & Plan | Week 1 | Research + produce a Team Plan (PRD + SDLC combined) |
| Learn & Refine | ~2 weeks | Learn tech stack, refine plan based on feedback. One plan gets chosen from four teams. |
| Build | ~8 weeks | Everyone builds the chosen plan as a working full-stack prototype |

### Tech Stack

| Layer | Technology |
|---|---|
| Mobile app | React Native (iOS + Android) |
| Backend | Java + Spring Boot |
| Database | MongoDB |
| T&F integration | Mocked by your own backend |

### The 7 Things Your Plan Must Cover

1. Problem & research summary
2. Proposed solution & design
3. Module breakdown
4. Feature list for 10 weeks (MVP)
5. Timeline — week by week
6. Tech approach
7. Assumptions & risks

---

## PART 2: THE PALACE PROJECT

### What It Is

Palace Project is a nonprofit reading app built by Lyrasis in partnership with DPLA. It is a fork of **Library Simplified** — the original open-source lending platform built by the New York Public Library (NYPL). The split happened in 2021: NYPL kept Library Simplified/SimplyE, Lyrasis launched Palace as a separate commercial turnkey service.

Both are open source. Palace's GitHub org is `ThePalaceProject`. The repos include:

- `circulation` — the backend (Python/Flask)
- `android-core` — Android app (Kotlin)
- `ios-core` — iOS app (Swift)
- `web-patron` — browser-based OPDS catalog client
- `library-registry` — tracks which libraries are in the network

### What Palace Actually Does

- Aggregates ebooks and audiobooks from many vendors (OverDrive, Boundless, BiblioBoard, ProQuest, etc.)
- Users browse, borrow, download, read offline
- Content types: **EPUB, PDF, audiobooks only** — no journals, no multimedia, no academic tools
- Uses OPDS feeds as the standard catalog format
- Uses Readium as the rendering engine
- Supports Adobe DRM and Readium LCP for content protection

### Palace's 5 Layers

```
USER (Student / Researcher)
         │
         ▼
PALACE MOBILE APP
Search · Borrow · Bookshelf · Download
React Native + Readium SDK
         │
    ┌────┴────┐
    ▼         ▼
PUBLISHER    READIUM LCP SERVER
BACKEND      Encrypt EPUB
OPDS Feed    Generate .lcp License
Borrow API   Store License Metadata
Concurrency  Create License ID
Loan DB
    │              │
    └──────┬────────┘
           ▼
    READIUM READER (SDK)
    Validate .lcp Signature
    Decrypt EPUB
    Render / Bookmarks / Search
           │
           ▼
    LICENSE STATUS (LSD)
    Active · Returned · Revoked · Expired
```

### The Complete Borrow Flow (Palace)

| Step | What happens |
|---|---|
| 1 | Publisher publishes OPDS catalog — metadata only, no loan, no license yet |
| 2 | Palace fetches `GET /opds` → displays books |
| 3 | User taps Borrow → `POST /borrow` with userId + bookId |
| 4 | Publisher validates: concurrency, subscription, checkout limit, permissions |
| 5 | Publisher calls `POST /generate-license` → LCP Server encrypts EPUB, creates `.lcp` file |
| 6 | Palace downloads two files: `AI.epub` + `AI.lcp` — both stored locally |
| 7 | Readium validates signature → decrypts EPUB → renders HTML |
| 8 | EPUB + license on device — no internet needed to read |
| 9 | License expiry: `GET /status` → LSD responds ACTIVE/RETURNED/REVOKED/EXPIRED |

### Two Independent Status Systems (Palace)

**Loan Status** — owned by Publisher Database:

```
ACTIVE → RETURNED → HOLD → EXPIRED → RENEWED
```

**DRM License Status** — owned by LCP/LSD Server:

```
Valid → Revoked → Returned → Updated → Cancelled
```

These are two completely separate systems. When a librarian cancels a loan: Publisher DB sets `status = Cancelled`, then separately calls LCP Server to revoke the license. Two events, two systems, coordinated at application level.

### How Readium LCP Works — The Hotel Room Analogy

| Physical | Digital |
|---|---|
| Hotel Room | EPUB file (stays on device always) |
| Key Card | DRM License (.lcp file) |
| Checkout time | Loan expiry date |
| Card deactivated | License revoked |

> **The EPUB stays on your phone — but without a valid license it is random, unreadable bytes.**

License validation runs 5 steps every time you open a book:

1. Read the `.lcp` file
2. Verify digital signature (RSA-SHA256 — one changed byte = rejected)
3. Check rights and expiry (copy, print, tts, devices, expires)
4. Extract encryption key from the license
5. Decrypt and render EPUB — opens only if all checks pass

### OPDS License Fields

| Field | Meaning |
|---|---|
| `checkouts` | Lifetime borrow limit |
| `concurrency` | Max simultaneous users |
| `length` | Loan duration |
| `expires` | Exact expiry date |
| `format` | EPUB / PDF / LCP |
| `devices` | Device limit per user |
| `copy` | Can user copy text? |
| `print` | Can user print? |
| `tts` | Text-to-speech allowed? |

### What Readium Is — Two Separate Parts

```
Readium
   │
   ├── Readium Reader (SDK)    → renders EPUB/PDF
   │   EPUB 2 + 3 parser
   │   PDF support
   │   Bookmarks + highlights
   │   Reading position
   │   In-book search
   │   Themes + accessibility
   │   Text-to-speech
   │
   └── Readium LCP             → the DRM layer (completely optional)
       Encrypts the EPUB
       Generates .lcp file
       RSA signature
       Key management
```

> These two are completely independent. You can use the Reader without LCP.

### Palace's Android Module Architecture

The Android app (`android-core`) is extremely modular. Key module groups:

| Stage | Modules | Job |
|---|---|---|
| Catalog | `opds-client`, `opds-core`, `opds2` | Parse OPDS feeds |
| Borrowing | `books-borrowing`, `books-controller` | Checkout orchestration |
| Local state | `books-database`, `books-registry-api` | Persisted + live in-memory state |
| Format handling | `books-formats` | Identifies file/license type |
| DRM (optional) | `lcp`, `adobe-extensions` | Gated behind build flags |
| Reading | `viewer-spi`, `viewer-epub-readium2`, `viewer-pdf-pdfjs`, `viewer-audiobook` | One interface, four implementations |
| Sync | `bookmarks`, `analytics-circulation` | Position/bookmarks sync |

**Key pattern:** viewer layer is a plugin system. `viewer-spi` defines "what a reader must do." EPUB/PDF/audiobook are separate, swappable implementations. Nothing in borrowing or database layer knows which one is used.

---

## PART 3: TAYLOR & FRANCIS

### Content Hierarchy

```
Journal
  └── Volume
        └── Issue
              └── Article
```

Books are flat. Journals are nested four levels deep.

### Content Types

- **Journals** — 2,700+ cross-disciplinary journals on Taylor & Francis Online (Atypon platform)
- **eBooks** — one of the world's largest STEM/HSS ebook collections, DRM-free, unlimited simultaneous users under most institutional models, on taylorfrancis.com (separate platform from journals)
- **Multimedia** — audio abstracts, podcasts, video articles attached to journal articles. T&F's own data shows multimedia more than doubles an article's readership
- **Reference works** — Routledge Handbooks Online, encyclopedic reference products, historical archive collections

### The Four Business Models

| Model | What it means |
|---|---|
| **Subscription** | Institution pays annually, access for that period. End date = subscription end. |
| **Trial** | Short-term access, same mechanism as subscription with near-term end date |
| **EBA** (Evidence Based Acquisition) | Broad access upfront, at period end T&F looks at usage, institution buys used titles permanently |
| **Perpetual** | Institution buys permanent access to specific title/collection, no expiry ever |

### Who the Users Are

- **Institutions** (university/hospital libraries) — primary commercial relationship
- **Researchers/faculty** — read, cite, publish multimedia alongside articles
- **Students** — access via institutional entitlement
- **Librarians/admins** — manage acquisitions, licenses, usage reporting

### Platform Fragmentation Problem

T&F journals and ebooks are on **two completely separate platforms** with different DOI-based routing logic. A user needs to know which platform to go to. This is the core problem your unified app solves.

### Existing Mobile Efforts

The 2014 T&F mobile site had a **device pairing mechanism** — pair a device to institutional login for up to 180 days of full access before needing to re-pair. Relevant pattern for your offline access model.

T&F's ebooks already support: offline access, PDF and EPUB formats, adjustable fonts, bookmarking, annotations, text-to-speech. But there is **no single, modern, unified mobile app** spanning journals + books + multimedia.

---

## PART 4: WHAT YOUR APP IS NOT (vs Palace)

| Palace model | Your model |
|---|---|
| Lending — borrow a book, it expires, you return it | Entitlement — access because your subscription covers it |
| Loan tracking per item per user | No loans, no returns, no per-item checkout |
| Concurrency = limited seats per book | Access = subscription active + content in bundle |
| LCP DRM encryption required | Unencrypted EPUB, access controlled at app layer |
| LSD server for real-time loan status | Local license.json expiry check |
| Multi-publisher, multi-library aggregator | Single publisher (T&F), multi-tenant capable |

> **T&F confirmed: no loans, no concurrency rules (as of now), no downloads — only read online.** Your app changes this by adding offline access.

---

## PART 5: OPDS

### What OPDS Is

OPDS (Open Publication Distribution System) is a standard format for listing books so any app that "speaks OPDS" can read any catalog that "speaks OPDS" — without custom code per source.

It's RSS for books. RSS = Really Simple Syndication, a standard format where a website publishes a list of its latest content that any RSS reader app can consume.

| | RSS | OPDS |
|---|---|---|
| Lists | Blog posts / podcast episodes | Books |
| Each entry has | Title, link, date, summary | Title, cover, author, borrow link |
| Built on | XML (Atom) | Atom XML (v1) or JSON (v2) |
| Lets any app | Read any blog | Read any catalog |

### Two Versions

- **OPDS 1.x** — XML/Atom (old)
- **OPDS 2.0** — JSON (modern, use this)

### OPDS in Your Architecture — Different from Palace

You don't serve OPDS live from a database. Instead:

1. A **scheduled job** (Lambda or cron) periodically calls T&F's internal catalog API
2. Builds the OPDS feed JSON
3. Stores it as a **static file in S3**
4. App fetches from S3 — fast, cheap, no database hit per browse request

This is correct because catalog content changes rarely (new books added weekly, not per second).

### OPDS for Multitenancy

- App ↔ your backend: use plain REST JSON — no OPDS needed, you own both ends
- New publisher ↔ your backend: **OPDS 2.0 is the ingestion contract**

Any publisher who exposes an OPDS 2.0 feed gets pulled in automatically by your importer — no custom backend code per publisher. Demo moment: "watch us onboard a new publisher live, with zero new code."

---

## PART 6: YOUR FEATURES — DETAILED DECISIONS

### Phase 1 Scope: Books + Journals Only

---

### Feature 1: Multitenancy

**Decision: Actually build it now — publishers as first-class entities.**

Every MongoDB collection carries `tenantId`. Auth resolves user to tenant at login. Every query is tenant-scoped.

```json
Publisher  { "tenantId", "name", "brandingConfig", "ingestionType": "opds2 | native" }
CatalogItem { "itemId", "tenantId", "type": "book | journal", "metadata", "sourceRef" }
```

T&F = tenant #1. Build one small demo publisher (handful of items) to prove the model. Skip per-tenant theming and billing for now.

---

### Feature 2: OPDS Architecture

Static JSON in S3, built by a scheduled ingestion job. OPDS 2.0 format. Used for catalog browsing (metadata only) and as the publisher onboarding contract for new tenants.

---

### Feature 3: Authentication

**Two account types:**
- Institutional (SSO/IP-based)
- Individual B2C (email-based, per Informa's model)

**Token model:**
- Short-lived access token (~1hr, silent refresh while online)
- Longer refresh token (30 days) = forced logout frequency

**Critical UX rule from Palace:** credentials entered once and stored. Re-auth only required for new downloads, never for opening already-downloaded content — even past refresh token expiry.

**Auth flow:**
```
University login → Identity Provider → OAuth/JWT → Content API → Token validated → content returned
```

---

### Feature 4: Reader (Readium Without DRM)

Use `react-native-readium` as a pure rendering engine. No LCP. No encryption.

**Content types:**
- Books/ebooks: PDF + EPUB via Readium Reader
- Journal articles: PDF via Readium, HTML as bundled asset (HTML + CSS + images zipped)
- Multimedia: separate video/audio player component

**Readium gives you for free:** EPUB 2+3 parsing, PDF support, bookmarks + highlights, reading position, in-book search, themes, accessibility, text-to-speech.

**Readium's only job:** receives a local file path, renders it. Knows nothing about licenses or subscriptions.

```javascript
// Readium is called AFTER your app has already
// verified the license. It just renders.
<ReadiumView
  file={localEpubPath}
  location={lastReadPosition}
  onLocationChange={(loc) => saveReadingPosition(loc)}
  settings={{ fontSize: 18, theme: 'dark' }}
/>
```

---

### Feature 5: Premium / DRM Tier

**Decision: Real cryptographic enforcement — JWT-signed license tokens.**

Flow:
1. Checkout request → backend checks active seat count vs `maxConcurrentSeats`
2. Seat free → server generates signed JWT: `{ licenseId, userId, contentId, deviceId, iat, exp }`
3. Client verifies token signature, gets access
4. `exp` checked client-side — content locks on expiry even offline
5. Online renewal/revocation via heartbeat call

---

### Feature 6: Denial Events + Usage Tracking

```json
DenialEvent {
  "tenantId": "",
  "userId": "",
  "contentId": "",
  "timestamp": "",
  "reason": "seat_limit_reached | not_entitled | expired_license"
}
```

Two consumers:
- **User-facing:** "all seats in use" message + holds/waitlist option
- **Admin-facing:** denial count per title → librarian insight ("buy more seats")

---

### Feature 7: Search

- Rich filters: author, subject, DOI, ISBN
- Related works via stored `relatedItemIds` on each CatalogItem
- Collaborative filtering: stretch goal only, not MVP (needs real usage volume)

---

### Feature 8: Individual B2C Tier

`accountType: "individual" | "institutional"` on user record. Individual signup is a mocked flow (toggle that flips the flag). Same content, different entitlement-resolution path.

---

### Access Badges

| Badge | Meaning |
|---|---|
| Open Access | Free, permanent, no auth needed |
| Full Access | Entitled via institution or individual subscription |
| Preview Only | Not entitled, abstract/sample only |
| Requires Institutional Login | Visible in search, gated behind SSO |
| Limited Seats (Premium) | DRM tier, concurrent-seat content |
| Downloaded | Available offline right now |

---

### Later Features (Post-MVP)

- **Audiobooks:** not a core T&F product — build generic audio/video player for multimedia extenders first, extensible to full audiobooks later
- **Citation/export tools:** `GET /items/{id}/citation?format=bibtex|ris|endnote`
- **Cross-referencing:** chapters ↔ articles via stored `citedByItemIds`
- **COUNTER-style usage reporting:** biggest differentiator for librarians — raw data already collected from denial events + usage logs
- **Reading lists / course-reserve collections:** faculty curating sets for courses/research groups
- **Recommendation engine:** needs usage data first
- **Open Access filter:** first-class, prominent filter

---

## PART 7: THE FOUR CONCEPTS

### Authentication, Authorization, License, Entitlement

Think of a university library with a restricted rare books room:

| Concept | Question it answers | Analogy |
|---|---|---|
| **Authentication** | Who are you? | Door guard checking your ID |
| **Authorization** | Are you allowed to do this action? | Librarian checking if you can enter the rare books room |
| **Entitlement** | Is this specific book in your package? | Catalogue checker — is this title in your institution's collection? |
| **License** | Is your subscription still active right now? | Has your membership expired? |

**Every time a user opens a book, all four run in sequence:**

```
1. Auth        → valid token?              → No → login screen
2. Authz       → allowed to read content?  → No → access denied
3. License     → subscription active?      → No → "access expired"
4. Entitlement → book in your package?     → No → "preview only"
5.             → all pass → open the book
```

**Important:** License Service and Entitlement Service are siblings — neither calls the other. The Content API calls both and combines the answers.

```
Content API          ← the orchestrator
    │        │
    ▼        ▼
License    Entitlement
Service    Service
    │        │
    ▼        ▼
Is today   Is this book
between    in their
start/end? package?
    │        │
    └───┬────┘
        ▼
   Both pass?
   → Return content (S3 URL)
```

---

## PART 8: THE MICROSERVICES

### Five Services, Strict Boundaries

```
Mobile App
    │
    ▼
API Gateway ──── Auth Service ──── Users DB (MongoDB)
    │
    ├──→ Content API ──────────────────────── S3 (EPUBs + OPDS feed)
    │         │
    │         ├──→ License Service ─────────── Licenses DB (MongoDB)
    │         │
    │         └──→ Entitlement Service ──────── Bundles DB (MongoDB / Neo4j)
    │
    └──→ Notification Service
```

---

### Service 1 — Auth Service

Only job: manage identity. Issue tokens. Refresh. Logout.

```
POST /auth/login     → validate credentials, return JWT
POST /auth/refresh   → take refresh token, return new JWT
POST /auth/logout    → invalidate session
```

**Owns:** Users collection. Knows nothing about books or subscriptions.

```json
{
  "userId": "u123",
  "email": "john@university.ac.uk",
  "accountType": "institutional",
  "institutionId": "inst_oxford",
  "passwordHash": "...",
  "refreshToken": "..."
}
```

---

### Service 2 — API Gateway

Every request hits here first.
- Validates JWT → Authentication
- Checks user role → Authorization
- Forwards to correct downstream service

Nothing reaches any other service without passing through here.

---

### Service 3 — License Service

Only job: time-bound subscription validity.

```
GET  /license/{userId}/check      → is any license currently active?
POST /license                     → create license (admin)
PUT  /license/{licenseId}/revoke  → early revocation
```

```json
{
  "licenseId": "lic_001",
  "institutionId": "inst_oxford",
  "publisherId": "tandf",
  "bundleId": "bundle_stem_2026",
  "startDate": "2026-01-01",
  "endDate": "2026-12-31",
  "status": "active"
}
```

Does **not** know which books are in the bundle. That is entitlement's job.

---

### Service 4 — Entitlement Service

Only job: is this specific content item inside this user's bundle?

```
GET /entitlement/{userId}/{contentId}  → entitled or not?
GET /entitlement/{userId}              → list all accessible content
```

**Check sequence:**
1. Get user's `institutionId`
2. Get all active `licenseIds` for that institution (calls License Service)
3. Get all `bundleIds` covered by those licenses
4. Check if requested `contentId` is in any bundle
5. Return: entitled / not entitled

**Why Neo4j here:** the relationship between users, bundles, licenses, publishers, and content items is a graph problem. Neo4j traverses `user → bundle → publisher → 3,000 titles` much faster than SQL joins.

**The hard problem:** if a user has a bundle covering T&F + another publisher, and the T&F part expires, you need to instantly know which content is blocked without re-checking everything. Graph DB solves this.

---

### Service 5 — Content API

Only job: serve content after all checks pass.

```
GET /content/{contentId}/metadata   → title, author, cover, format
GET /content/{contentId}/download   → pre-signed S3 URL (expires in 15 mins)
GET /content/catalog                → OPDS feed (served from S3)
```

Calls License Service + Entitlement Service. Never does its own auth — gateway already handled that.

---

### The Full Request Flow — Opening a Book

```
1. App sends: GET /content/isbn_001/download
   + Authorization: Bearer <JWT>

2. API Gateway:
   → verifies JWT signature          ✓ Authentication
   → checks user role = "reader"     ✓ Authorization
   → forwards to Content API

3. Content API calls License Service:
   → GET /license/u123/check
   → startDate=Jan1, endDate=Dec31, today=Jul12 ✓
   → Returns: valid

4. Content API calls Entitlement Service:
   → GET /entitlement/u123/isbn_001
   → isbn_001 is in bundle_stem_2026
     covered by lic_001, belonging to inst_oxford ✓
   → Returns: entitled

5. Content API returns:
   → Pre-signed S3 URL (expires in 15 mins)
   → License metadata: { startDate, endDate, copy, print, tts }

6. App downloads EPUB to local storage
   Stores license metadata alongside it

7. User opens book (even offline):
   → App reads local license metadata
   → Checks: is today before endDate? ✓
   → Opens Readium — no network call needed
```

---

## PART 9: THE OFFLINE / ONLINE MODEL

### Download Time vs Open Time — Two Completely Separate Gates

```
DOWNLOAD TIME (happens once)       OPEN TIME (every single open)

Needs network ✓                    Works offline ✓
Hits your server ✓                 Reads local file only ✓
License check ✓                    No network call ✓
Entitlement check ✓                Just: today < endDate? ✓
Runs once ✓                        Runs every open ✓
```

---

### What Gets Stored Locally — Two Files Only

```
/local-storage/content/isbn_001/
    ├── book.epub          ← plain, unencrypted file
    └── license.json       ← checked on every open
```

```json
{
  "contentId": "isbn_001",
  "userId": "u123",
  "institutionId": "inst_oxford",
  "startDate": "2026-01-01",
  "endDate": "2026-12-31",
  "issuedAt": "2026-07-12T10:00:00Z",
  "copy": true,
  "print": false,
  "tts": true
}
```

---

### Every Scenario Handled

| Scenario | What happens |
|---|---|
| Active subscription, online or offline | Read license.json → today before endDate → Readium opens |
| Expired subscription | Read license.json → today after endDate → block, show expiry message |
| Fully offline | Same local check, works identically, no network needed |
| Institution renews | App syncs → License Service returns new endDate → update license.json → reopens |
| User manually copies EPUB | Plain file, usable outside app — accepted risk, T&F already distributes DRM-free ebooks |

---

### The Open Book Logic

```javascript
const openBook = async (contentId) => {

  // No network call — reads local file
  const license = await readLocalLicense(contentId);

  // Pure date comparison
  const today = new Date();
  const endDate = new Date(license.endDate);

  if (today > endDate) {
    showScreen("SubscriptionExpired");
    return;
  }

  // All good — hand to Readium
  // Readium knows nothing about any of the above
  const epubPath = getLocalEpubPath(contentId);
  openReadiumReader(epubPath);

};
```

---

### Protection Layers (Without Encryption)

```
Layer 1: JWT auth            → only valid users get in
Layer 2: License check       → only active subscribers get URLs
Layer 3: Entitlement check   → only entitled users get this specific book
Layer 4: Pre-signed URL      → URL expires in 15 mins, cannot be shared
Layer 5: Local expiry check  → app blocks content after subscription ends
Layer 6: Single publisher    → no third party whose rights you're risking
```

---

## PART 10: KEY FACTS FROM THE EMPLOYEE

- T&F does **not** currently do loans, concurrency rules, or downloads — read online only. Your app adds offline.
- A license at T&F is simple: `licenseId + startDate + endDate`. That's it.
- **Build as microservices**, not a monolith.
- Use **Lambda** for scheduled jobs (OPDS ingestion).
- OPDS catalog stored in **S3**, not served live from DB.
- **Two separate APIs**: one for content, one for LCP/license.
- License is called at **open time**, not download time.
- Use **ISBN and DOI** to differentiate licenses per content item.
- **Borrow API → Content API → Entitlement API** is the call chain.
- **Neo4j** for the publisher-bundle-content relationship graph.
- **Multiple token types for journals** — journals live on a different platform with different auth than books.
- **Open Access is a differentiator** — surface prominently via OPDS, no auth friction.
- **getFTR** (Get Full Text Research) — existing T&F mechanism resolving DOI to content URL, potentially hookable.
- Auth for Informa = **email-based**.
- **Blue-green deployment** for zero downtime releases.
- **VPC, subnets, security groups** for AWS networking isolation.
- **BDD format test cases** — Given / When / Then.
- **First 3 weeks without DRM** — start simple, add LCP later.
- Start by: creating a docx → converting to EPUB → opening in Readium → license check.

---

## PART 11: RECOMMENDED BUILD ORDER

| Step | What to build |
|---|---|
| 1 | Understand T&F's four business models + content hierarchy |
| 2 | Build users list + email auth |
| 3 | Create a docx → convert to EPUB → open in Readium (prove the pipe works) |
| 4 | Build License API: `licenseId + startDate + endDate + check on open` |
| 5 | Build Content API: serve EPUB via pre-signed S3 URL behind license check |
| 6 | Build OPDS: scheduled job → S3 → catalog feed |
| 7 | Build Entitlement layer: per-book checks on top of bundle license |
| 8 | Add multitenancy: `tenantId` on all collections, demo second publisher via OPDS ingestion |
| 9 | Add concurrency (premium tier) |
| 10 | Add DRM/LCP — only after all above is solid |
| 11 | Write BDD tests throughout |

---

## PART 12: YOUR DIFFERENTIATORS vs Palace

| Pain point in Palace | What your app does better |
|---|---|
| Ebooks + audiobooks only | Books + journals + multimedia unified |
| Multi-publisher aggregator, shallow per-publisher features | Single publisher, deep T&F-specific features (DOI, citations, related works) |
| Weak search across federated catalogs | Tunable search over one known, bounded catalog |
| No journal-specific tooling | Citation export, article ↔ chapter cross-referencing |
| No librarian analytics | COUNTER-style usage reporting, denial event tracking |
| No open access first-class treatment | OA content surfaced prominently, no auth friction |
| No course reserve / reading lists | Faculty-curated collections for courses/research groups |
| Multi-library, multi-vendor complexity | Clean single-publisher entitlement model |

---

*Document compiled from: hackathon brief, Palace Project GitHub research, T&F platform research, internal Palace presentation (palace-presentation.html), and employee session notes.*
