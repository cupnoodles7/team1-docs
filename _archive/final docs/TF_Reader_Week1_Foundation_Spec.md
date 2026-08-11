# T&F Reader — Week 1 Foundation Specification

**Team:** team1 — Discovery & Selection
**Week:** 1 — 10 to 14 August 2026
**Scope:** Phase 0 foundation sprint (Days 1–2, all five people) and Phase 1 component + feature work (Days 3–5)
**Status:** Supplements the Team1 Delivery Plan. Where this document and Section 05 of the delivery plan disagree, this document is correct — see §1.2.

---

## 1. Purpose and premise

### 1.1 What this document is for

The delivery plan says *what* Week 1 produces. This says *who does what, in what order, and what "done" means* — at the resolution needed to start on Monday morning without another planning conversation.

It covers two things:

1. **The Phase 0 foundation sprint** — Days 1–2, all five people on one branch, no feature work. This is the blocking work, and it is collectively owned precisely so that no single person is a bottleneck.
2. **Days 3–5 per person**, with Khushi's block re-cut as genuine leaf work.

### 1.2 A correction to the delivery plan

The delivery plan contains an internal contradiction that has to be resolved before Monday.

| Source | What it says about the component library |
|---|---|
| Section 01, Phase 1 (`_build_plan.py:253`) | "built by all five people during the foundation phase, not by one owner" |
| Section 01, inventory table (`_build_plan.py:322`) | 22 components, split five ways, author named per component |
| Section 07, load by week (`_build_plan.py:1035`) | "Khushi: Tokens Day 1; **her seven components**; F4 shell and navigation; state gallery" |
| **Section 05, Week 1 Days 3–5 (`_build_plan.py:788`)** | **"Khushi: Design tokens, the top ten components, the state gallery"** |

Three sources say five-way split. One says Khushi owns ten. **The Section 05 row is the outlier and is treated here as stale.** The five-way split stands.

This matters because the two readings produce completely different weeks. Under the Section 05 reading, one person is a hard blocker on the other four for three days. Under the Section 01 reading, the blocking work is done by everyone in Phase 0 and nobody waits.

### 1.3 The two genuine fan-out risks

With the split restored, only two Week 1 artefacts block more than one person:

- **Design tokens** — single-authored by Khushi on Day 1, before any component exists, so no component author ever picks a colour or a spacing value. Roughly a two-hour transcription job from the design specification. Single authorship here is deliberate, not a bottleneck: it is the thing that makes splitting the other twenty-one components safe.
- **App shell and navigation (F4)** — everything with a screen needs it. Moved into Phase 0 as an all-hands item.

Everything else in Week 1 is either self-contained or consumed from a fixture.

### 1.4 Standing assumptions

Three assumptions are baked into this document. All are flagged rather than hidden:

- **Greenfield.** No T&F Reader codebase exists. Day 1 begins with `npx create-expo-app`, not with `git pull`. §3.1 covers the bootstrap.
- **Individual subscribers (B2C) are NOT cut — reversed 11 August.** wokay recommended cutting them at their own gate (decision 7) and that recommendation is **not** being taken; they will supply the B2C details later. **`subscribe`, `Session.type` and the Subscribe flow all stay.** Every statement in this document that says Subscribe or B2C is deleted, cut or removed is superseded by this line — they were written when the cut looked settled. Two live consequences: `ActionButton` keeps six variants rather than five, and screen 03's second option is **not** settled as browse-free-content.
- **TypeScript — reversed 11 August.** This document previously specified JavaScript, and §1.5 below specified three replacements for the missing compiler. The team has switched to TypeScript with `strict: true`, so the compiler is back and §1.5 is largely moot. What remains of it is in §1.5a. **Anything elsewhere in this document that assumes JavaScript, JSDoc typedefs or `propTypes` is superseded.**
- **Khushi is newer to React Native.** Her block is re-cut to props-only presentational components — no navigation, no async, no effects, no adapter access. §7 sets out the rule and why it is also good architecture rather than a special case.

### 1.5 Contracts as types — restored

The delivery plan's central mechanism is *contracts as types*: one file defines `ContentItem`, `Institution`, `Session`, `DataAdapter` and `resolveAccess`, four people code against it, and the same file is attached to the Section 09 question sets so that wokay's and flambeau's silence safely means *they accepted our shape*.

> **One correction to that premise, and it matters this week.** "Silence means acceptance" holds where **we** send a shape and they do not object. It does **not** hold on the four items wokay have asked *us* for by end of Week 1 — there, their document publishes their own silence-defaults, and on one of them (subject filters) their default deletes a designed feature. §P0-8 sets out what must be answered rather than merely sent.

**As of 11 August that mechanism is intact rather than simulated.** The contract is `src/model/types.ts`, TypeScript with `strict: true`. Three things this section used to work around are now simply true:

| What the compiler does | Consequence |
|---|---|
| Guarantees the whole team builds against one shape | A change to `types.ts` breaks the build for whoever hasn't followed it, in seconds, by name. Previously a red squiggle they had to open the file to see |
| Guarantees `MockAdapter` and `ApiAdapter` stay interchangeable | `implements DataAdapter` on both. The Week 4 claim that "integration is a configuration change" is now checked, not hoped for |
| Carries the prop contracts | One props interface per component. **`propTypes` is deleted from the conventions** — see §1.5a |

**The vocabularies are still runtime arrays, and that is now a feature rather than a workaround.** `ACCESS_TIERS` and the rest are declared as `as const` arrays with their union types derived via `typeof X[number]`. One declaration produces both the value that `validate.ts` and the gallery iterate, *and* the type. Adding a member and forgetting the other half is no longer possible.

### 1.5a What still carries its own weight

The compiler cannot check data that arrives at runtime, so two controls survive unchanged:

| Risk | Control | Where |
|---|---|---|
| A malformed fixture, or a real API response that doesn't match the contract | **Validation on load** — asserts against the contract in dev and throws loudly, naming the item and the field | P0-4 |
| Mock and real adapters drifting in **behaviour** — a zero-result feed throwing instead of returning `browseInstead`, an unknown id returning undefined instead of rejecting | **The adapter conformance suite.** Shape is the compiler's job now; behaviour is still the suite's | P0-3 |

**And one control is deleted, not replaced.** `propTypes` is gone. It was never going to work: React 19 — which ships with current Expo — ignores `propTypes` silently, so 21 components would have carried blocks that caught nothing. One typed interface replaces the JSDoc typedef *and* the propTypes block, so this is less work than the JavaScript path, not more.

**`tsconfig.json` is committed with `strict: true`** and `npm run typecheck` runs in CI — a real gate rather than an editor nicety. `any` and `as` are allowed without review pushback in Week 1: the value is in the contract file and the props, not in purity. A team that treats the compiler as something to satisfy loses time; a team that treats it as a contract checker with escape hatches doesn't.

---

## 2. Week 1 at a glance

```
DAY 1 (Mon 10)     DAY 2 (Tue 11)      DAY 3 (Wed 12)      DAYS 4-5 (Thu 13 - Fri 14)
──────────────     ──────────────      ──────────────      ──────────────────────────
ALL FIVE, ONE BRANCH, NO FEATURE WORK  LIBRARY COMPLETION  FEATURE WORK

Hour 0 scaffold    shell + tab nav     Prayas   4 comps    Prayas   F1, F2, A4
Tokens  (Khushi)   core five comps     Moktik   2 comps    Moktik   search shell, B1
Types   (Akriti)   fixtures complete   Keshav   3 comps    Keshav   B9, C1
Fixtures (Prayas)  ────────────────    Akriti   2 comps    Akriti   D1 skeleton, F6
Conventions (all)  GATE: main runs     Khushi   2 comps    Khushi   2 comps, gallery, C2
Section 09 sent    6 of 21 merged      ──────────────      ──────────────────────────
                                       19 of 21 merged     Fri 16:00 consistency review
                                                           Fri 17:00 board handover
```

**The Phase 0 gate is absolute.** No feature branch is cut until every item in §3 is merged to `main` and the app runs on a device or simulator. Two days of apparent slowness buys back a week; skipping the gate is what turns Week 3 integration into a rewrite.

---

## 3. Phase 0 — Days 1–2, all five people, one branch

Eight deliverables. All merged to `main` by end of Day 2.

### P0-1 · Repository scaffold

**Owner:** Prayas, with all five pairing for the first hour
**Duration:** ~3 hours, Day 1 morning
**Blocks:** literally everything

Because there is no repo, hour zero is a set of decisions. Make them once, in the room, and write them into the README. The recommendations below are defaults chosen for a five-person, three-week prototype — take them unless someone has a concrete reason otherwise, and do not spend the morning debating them.

| Decision | Recommendation | Reason |
|---|---|---|
| Toolchain | **Expo (managed)** | No native build setup across five machines. The prototype needs no custom native modules. |
| Language | **TypeScript**, `strict: true` — reversed 11 Aug | See §1.5. Cheapest on Day 2 while only six components exist; the cost of switching roughly triples once Day 3's thirteen land. |
| Contract checking | **`tsconfig.json` with `strict: true`**, and `npm run typecheck` in CI | A build gate, not an editor hint. `any` and `as` permitted freely in Week 1. |
| Prop contracts | **One props interface per component.** `prop-types` is **not used** | React 19 ignores `propTypes` entirely, so the blocks would have caught nothing. One interface replaces two declarations — less work for everyone, including anyone newer to React Native, who now gets the error before the app builds rather than not at all. |
| Font | **`expo-font` + `@expo-google-fonts/inter`**, loaded at app start | Design Spec §2.2 names Inter. Retrofitting it after the library exists shifts every line height at once. |
| Navigation | **React Navigation** — `bottom-tabs` + `native-stack` | Four tabs and pushed detail screens is exactly its shape. Better documented than the alternatives, which matters with a mixed-experience team. |
| State | **Zustand** for session, selection and pending intent | Three small stores. Redux is more ceremony than this needs. |
| Persistence | **AsyncStorage** behind a `Storage` interface we own | Institution selection and pending intent must survive an app restart (see FL-5). Keep it behind an interface so swapping to MMKV is one file. |
| Testing | **Jest + React Native Testing Library** | Needed for the Week 4 BDD suite; cheaper to add on Day 1 than Week 4. |
| Quality | **ESLint + Prettier, pre-commit hook** | Five authors, one branch, daily merges. Non-negotiable. |

**Folder structure** — matches the file-ownership table in the delivery plan, so ownership is visible in the tree:

```
src/
  theme/          tokens.ts                        [Khushi — single author]
  model/          types.ts, fixtures/, validate.ts [Prayas — Akriti 2nd reviewer]
  adapters/       MockAdapter.ts, conformance.test.ts  [Prayas]
  access/         resolveAccess.ts, session.ts     [Akriti — only place access logic may live]
  components/     one folder per component         [split five ways]
  screens/        catalogue/ search/ institution/ detail/ library/ profile/
  search/         shell/ catalogue/ institution/   [Moktik — Keshav 2nd reviewer]
  storage/        pendingIntent.js, selection.js
  gallery/        StateGallery.tsx                 [Khushi]
  navigation/     RootNavigator.tsx, tabs.ts
```

**Done when:** `main` contains a running Expo app; all five people have cloned it, installed, and launched it on their own machine; lint and a trivial test pass in CI or via a documented command. **Every one of the five confirms this before anyone goes further** — a scaffold that only runs on the author's laptop is not a scaffold.

---

### P0-2 · Design tokens

**Owner:** Khushi — **single author, never split**
**Duration:** ~2 hours, Day 1, immediately after the scaffold runs
**Blocks:** every component

Transcribe §2.1, §2.2 and the measurable parts of §2.3 of `TF_Reader_Design_Package/docs/TF_Reader_Design_Specification.md` **verbatim** into typed constants. Transcription, not interpretation — do not add a colour, do not adjust a size, do not invent an intermediate grey.

```ts
// src/theme/tokens.ts   ← .ts, so the `as const` below is valid rather than a syntax error
export const color = {
  primary:      '#00A19D',  // T&F Teal — primary buttons, active tabs, links
  navy:         '#1A3A5C',  // Deep Navy — top bars, dark overlays
  textPrimary:  '#1A1A2E',  // Charcoal
  textSecondary:'#6B7280',  // Medium Gray — metadata, disabled
  surface:      '#F8F9FA',  // Off-White — cards, section backgrounds
  border:       '#E5E7EB',  // Light Gray
  success:      '#10B981',  // Open Access badges, successful download
  error:        '#EF4444',  // Access restricted, destructive
  wait:         '#F59E0B',  // No seats, waitlist
  subscription: '#2563EB',  // Subscription badge
  elite:        '#7C3AED',  // Elite badge
} as const

// Design Spec §2.2 — Inter, "or closest system sans-serif equivalent"
export const font = {
  family: 'Inter',
  fallback: 'System',
} as const

export const type = {
  pageTitle:     { weight: '700', size: 24, lineHeight: 32 },
  sectionHeader: { weight: '600', size: 18, lineHeight: 24 },
  body:          { weight: '400', size: 15, lineHeight: 22 },
  meta:          { weight: '400', size: 13, lineHeight: 18 },
  button:        { weight: '600', size: 15, lineHeight: 20 },
  smallLabel:    { weight: '500', size: 12, lineHeight: 16 },
} as const

// Design Spec §2.3 — cards carry a "subtle box shadow".
// React Native splits this by platform, so it must be a token or five
// authors will invent five shadows.
export const elevation = {
  card: {
    ios:     { shadowColor: '#1A1A2E', shadowOpacity: 0.06,
               shadowRadius: 8, shadowOffset: { width: 0, height: 2 } },
    android: { elevation: 2 },
  },
} as const

export const space  = { xs: 4, sm: 8, md: 16, lg: 24, xl: 32 } as const
export const radius = { card: 8, sheet: 16, pill: 999 } as const
```

**Inter must be loaded on Day 1, not later.** §2.2 names it specifically. On Expo that is `@expo-google-fonts/inter` plus `expo-font`, wired into the scaffold in P0-1. If it is deferred, all twenty-one components get built and eyeballed in the system font, and adding Inter in Week 3 shifts every line height and every truncation point in the library at once.

**A decision this file now has to make: no per-institution theming.** wokay's institution record carries `branding.primaryColor` (Imperial's is `#003E74`). It is tempting, and it is a trap in Week 1 — a runtime-variable primary colour means no token is a constant, every component that uses `color.primary` becomes theme-aware, and the Day 5 consistency review has no fixed baseline to review against. **The prototype ships one fixed palette: `color.primary` stays `#00A19D` (T&F Teal, §2.1).** `branding.primaryColor` is carried in the `Institution` type and deliberately unused, so adopting it later is additive rather than a refactor.

This is a decision, not an oversight — say it in standup, and put it on the dependency board as *decided by team1* rather than leaving it to look like something nobody noticed. If leadership want institution-branded headers at the demo, it is a Week 3 conversation with a real cost.

**Values inferred rather than specified** — flag all three in standup so nobody later treats them as ratified design:

| Token | Basis |
|---|---|
| `radius.card: 8`, `radius.sheet: 16` | **Specified** — §2.3, stated explicitly |
| `radius.pill: 999` | Inferred — §2.3 says badges are "pill-shaped" but gives no value |
| `space.*` | Inferred — no spacing scale exists anywhere in the design specification |
| `elevation.card` | Inferred — §2.3 says "subtle box shadow" with no values |

**Done when:** merged; Inter renders on both platforms; and a lint rule or documented review check rejects any raw hex, raw spacing value or inline shadow inside `src/components/`.

---

### P0-3 · Contracts as types

**Owner:** Akriti
**Duration:** Day 1 afternoon
**Blocks:** all four other people, and Section 09

This file is the seam the other four code against, and it is simultaneously the artefact attached to the Section 09 question sets. Writing it is not internal prep — it is the contract dispatch.

> **This is no longer transcription from scratch. It is reconciliation against a known backend.** wokay have published their source of truth: seven MongoDB collections, the `catalogueItems` record in full, the two app-facing REST shapes, the OPDS acquisition-link block and the error envelope. Most of what this file used to *ask for* is answered. Write it against their field names, and mark only the genuinely-missing fields as asks. §P0-8 lists what still goes out.

```js
// src/model/types.ts
// The single contract. Four people code against this file, and this file is
// what goes out with the Section 09 question sets. TypeScript, strict.
// Field names track wokay's published schema — do not rename them to taste.

/** @typedef {'OPEN_ACCESS'|'SUBSCRIPTION'|'ELITE'} AccessTier
 *           wokay's catalogueItems.accessTier. UPPERCASE is theirs, not ours. */
/** @typedef {'available'|'requires_loan'|'requires_signin'|'not_entitled'|'no_seats'} AccessState */
/** @typedef {'read'|'download'|'borrow'|'signin'|'waitlist'} ActionId
 *           'subscribe' RETAINED — B2C is not cut. Details to come from wokay. */
/** @typedef {'PDF'|'EPUB'|'AUDIO'} FileFormat   wokay's contentType. A FORMAT. */
/** @typedef {'book'|'journal'|'article'|'audiobook'} WorkType
 *           ⚠ UNBACKED. wokay model no work type at all; contentType is the
 *           file format above. Screens 04 and 05 differ by exactly this axis.
 *           THE DAY 1 ASK — see §P0-8. Until answered, fixtures carry it and
 *           the adapter derives it heuristically, isolated in ONE function. */
/** @typedef {string} ShelfId
 *           A group id from the root feed's navigation rows. Discovered per
 *           institution and configured per institution (wokay's admin console
 *           has a per-institution catalogue config: feed title, shelf order,
 *           page size). NOT three fixed endpoints. */

/**
 * The acquisition link, normalised. THIS is what decides the buttons.
 * wokay: "the acquisition link decides the buttons, not your own logic.
 * If we do not send a link, the button does not exist." Absent → actions [].
 * @typedef {Object} Acquisition
 * @property {'open-access'|'acquisition'|'borrow'} rel   rel, tail segment only
 * @property {string}  href          flambeau's loan endpoint. FOLLOW IT, never build it
 * @property {string}  type          MIME
 * @property {boolean} canPersist    false ⇒ no Download button, and the server refuses it
 * @property {boolean} encrypted     encryption block present ⇒ Subscription or Elite
 * @property {boolean} hasSearchIndex
 * @property {number}  [originalLength]
 * @property {'UNLIMITED'|'CONCURRENT'} [licenceModel]
 * @property {{total: number}} [copies]   total ONLY. available is NOT in the feed
 */

/**
 * ⚠ THIS BLOCK IS AN EXTRACT, NOT THE CONTRACT. The live file is
 * TF_Reader_Mobile_team1/src/model/types.ts, and it is ahead of this document —
 * it carries the full reconciliation against wokay's frozen samples and their
 * answers of 11 August. Read it there before writing code.
 *
 * @typedef {Object} ContentItem
 * @property {string}      id             ⚠ NOT a field. Parse it off the tail of
 *                                        the publication's `self` href.
 *                                        `metadata.identifier` is not the id
 * @property {ShelfId}    [shelfId]       which shelf this arrived on
 * @property {WorkType}   [type]          ✅ from metadata.@type — schema.org/Book
 *                                        and /Audiobook confirmed; journal and
 *                                        article values still to come
 * @property {FileFormat}  contentType    ⚠ DERIVED from the acquisition link's
 *                                        MIME. There is no contentType in the feed
 * @property {string}      title
 * @property {string}     [subtitle]      detail feed only
 * @property {string[]}    authors        ⚠ may be EMPTY — one sample book has
 *                                        editors and no author. Fall back to them
 * @property {string[]}   [editors]
 * @property {string[]}   [narrators]     audiobooks
 * @property {string}     [publisher]     from metadata.publisher.name — an object
 * @property {string}     [language]      ISO code — wokay have it, we did not
 * @property {string}     [publishedDate] ✅ metadata.published, an ISO date
 *                                        ('2020-09-30') — a real publication
 *                                        date, not the ingest createdAt
 * @property {string[]}    subjects       from metadata.subject[].name
 * @property {string}     [coverUrl]      largest entry in images[]
 * @property {string}     [thumbUrl]      smallest entry in images[]
 * @property {string}     [abstract]      from description — DETAIL FEED ONLY
 * @property {number}     [pageCount]     metadata.numberOfPages
 * @property {number}     [durationSec]   metadata.duration, audiobooks
 * @property {{isbn?: string, raw: string}} identifiers
 *           ⚠ URN PARSING IS BACK. The feed emits metadata.identifier as
 *           'urn:isbn:9780367211745', or 'urn:tf:catalogue:item_env' where there
 *           is no ISBN. The flat isbn in wokay's document is not in the feed.
 *           doi does NOT EXIST and never will — drop it from screen 04.
 * @property {AccessTier}  accessTier     ⚠ DERIVED — there is no tier field.
 *                                        licenceModel UNLIMITED → SUBSCRIPTION,
 *                                        CONCURRENT → ELITE, absent → OPEN_ACCESS.
 *                                        wokay's mapping. Computed in the adapter
 *                                        so components still get a plain field
 * @property {Acquisition}[acquisition]   absent ⇒ no actions at all. Exactly ONE
 *                                        per work — confirmed, so there is no
 *                                        multi-format case
 */

/**
 * wokay's GET /api/v1/institutions/{id}. Unauthenticated. Their shape, not ours.
 * @typedef {Object} Institution
 * @property {string}  id                  "inst_7f3"
 * @property {string}  code                "imperial" — unique
 * @property {string}  name
 * @property {string} [type]               "UNIVERSITY"
 * @property {string}  country
 * @property {string} [city]
 * @property {string} [logoUrl]            ✅ ANSWERED — was crestUrl (W-17)
 * @property {{logoUrl?: string, primaryColor?: string}} [branding]
 *           primaryColor is per-institution. We do NOT theme from it — see §P0-2
 * @property {{method: 'SAML', idpHint: string}} [signIn]
 *           method is ALWAYS 'SAML'. There is no authMethod field on their
 *           record; it stays in the payload so we need no special case.
 *           idpHint is what we hand to flambeau — see §5, Keshav.
 * @property {string} [catalogueUrl]       THEY hand us this. We never build it.
 */

/**
 * @typedef {Object} Session
 * @property {string}   userId
 * @property {string}  [institutionId]   optional — a B2C session may have none.
 * @property {string[]} roles
 * @property {number}   exp
 */

/**
 * Per-user, per-item and mutable — flambeau CAP-4. Never a property of the feed.
 * @typedef {Object} Loan
 * @property {string} itemId
 * @property {'none'|'active'|'expired'} state
 * @property {number} [expiresAt]
 */

/**
 * GET /api/v1/availability?itemId= — flambeau, Wk 4, ELITE ONLY, DETAIL ONLY.
 * null everywhere else, because copies.available is not in the feed.
 * @typedef {Object} Availability
 * @property {number}  available
 * @property {number}  total
 * @property {number} [queuePosition]
 */

/**
 * wokay's error envelope. One shape for every failure.
 * @typedef {Object} ApiError
 * @property {string} timestamp
 * @property {number} status
 * @property {ErrorCode} code       ErrorState copy is keyed on THIS, not on status
 * @property {string} message
 * @property {string} path
 * @property {string} traceId
 */
/** @typedef {'NO_ENTITLEMENT'|'ENTITLEMENT_EXPIRED'|'ENTITLEMENT_SUSPENDED'
 *           |'CONTENT_NOT_READY'|'DOWNLOAD_NOT_PERMITTED'
 *           |'FORBIDDEN_INSTITUTION_MISMATCH'|'INVALID_DEVICE_PUBLIC_KEY'
 *           |'UNAUTHENTICATED'|'TOKEN_EXPIRED'|'NOT_FOUND'} ErrorCode */

/**
 * @typedef {Object} AccessResult
 * @property {AccessTier}  tier      A BADGE LABEL. Not an input to `actions`.
 * @property {AccessState} state
 * @property {ActionId[]}  actions   derived from item.acquisition — §3.1
 */

/**
 * `availability` is nullable and is null on EVERY list surface, because
 * copies.available does not exist in the feed. Supply it only on item detail,
 * after the flambeau call. no_seats is returned only when it is non-null and
 * available === 0 — which is why a ContentCard can never render no_seats.
 * @callback ResolveAccess
 * @param {ContentItem}       item
 * @param {Session|null}      session       null = logged out. Design Spec §5.2.
 * @param {Loan|null}         loan
 * @param {Availability|null} [availability] detail screen only, Elite only
 * @returns {AccessResult}
 */

/**
 * OPDS paginates by `next` link only. Never count pages, never build a URL.
 * `total` is populated only by the institutions REST endpoint, which is the
 * one place page/size/total exists.
 * @template T
 * @typedef {Object} Page
 * @property {T[]}     items
 * @property {string} [nextHref]   follow it until it is absent
 * @property {number} [total]      institutions endpoint only
 * @property {boolean}[browseInstead] zero-result feed carried a navigation
 *                                    entry instead of publications — §P0-4
 */

/**
 * Shaped around "follow hrefs, do not build URLs". Every method that walks the
 * catalogue takes an href we were given, not an id we templated.
 * @typedef {Object} DataAdapter
 * @property {(catalogueUrl: string) => Promise<Array<{id: ShelfId, label: string, href: string}>>} getShelves
 *           parses the root feed's navigation rows. There is no fixed feed list.
 * @property {(href: string) => Promise<Page<ContentItem>>} getShelf
 *           also used for the next link — same method, different href
 * @property {(searchTemplate: string, q: string) => Promise<Page<ContentItem>>} searchCatalogue
 *           the ONE permitted expansion: ".../search{?query}"
 * @property {(href: string) => Promise<ContentItem>} getItem
 * @property {(ids: string[]) => Promise<{items: ContentItem[], notFound: string[], denied: string[]}>} getItemsBatch
 *           POST /api/v1/catalogue/items:batch, capped at 100 ids
 * @property {(params: {q?: string, country?: string, page?: number, size?: number}) => Promise<Page<Institution>>} getInstitutions
 *           the one paged endpoint: page/size/total, not next links
 * @property {(id: string) => Promise<Institution>} getInstitution
 */

// Runtime unions. In TypeScript these were types only; in JavaScript they must
// exist as values, because validation, the filter chips and the state gallery
// all enumerate them. One definition, three consumers.
export const ACCESS_TIERS  = ['OPEN_ACCESS', 'SUBSCRIPTION', 'ELITE']
export const ACCESS_STATES = ['available', 'requires_loan', 'requires_signin',
                              'not_entitled', 'no_seats']
export const ACTION_IDS    = ['read', 'download', 'borrow', 'signin', 'waitlist']
export const FILE_FORMATS  = ['PDF', 'EPUB', 'AUDIO']
export const WORK_TYPES    = ['book', 'journal', 'article', 'audiobook']  // unbacked
export const ACQ_RELS      = ['open-access', 'acquisition', 'borrow']
export const ERROR_CODES   = ['NO_ENTITLEMENT', 'ENTITLEMENT_EXPIRED',
                              'ENTITLEMENT_SUSPENDED', 'CONTENT_NOT_READY',
                              'DOWNLOAD_NOT_PERMITTED',
                              'FORBIDDEN_INSTITUTION_MISMATCH',
                              'INVALID_DEVICE_PUBLIC_KEY', 'UNAUTHENTICATED',
                              'TOKEN_EXPIRED', 'NOT_FOUND']
// AUTH_TYPES is DELETED. Institutional sign-in is always SAML; there is no
// field to branch on. Anything importing it must be updated — see §5, Keshav.
```

#### What changed, and why each change matters

| Was | Now | Because |
|---|---|---|
| `accessTier` — *"⚠ THE Q-D ASK"* | ✅ answered, uppercase | It already exists on `catalogueItems`, is exposed on `items:batch`, and is offered as a **free filter**. The hardest ask in the plan was answered before it was asked. |
| `AUTH_TYPES`, `Institution.authType` | **deleted** | *"Institutional sign-in is ALWAYS SAML, so there is no `authMethod` field."* Replaced by `signIn: {method:'SAML', idpHint}`. |
| `crestUrl` | `logoUrl`, plus `branding` | W-17 answered. |
| `copies: {total, available}` | `copies: {total}` on the acquisition link | The feed cannot know `available` — wokay never call flambeau while building a feed. `available` is a separate per-item call. |
| tier → actions | `Acquisition` → actions | §1.2 below. The largest correction in this document. |
| `getFeeds()` / `getCatalogue(feedId)` | `getShelves(catalogueUrl)` / `getShelf(href)` | We are handed `catalogueUrl` and told *"follow hrefs, do not build URLs."* Shelves are navigation rows on the root feed. |
| — | `getItemsBatch(ids)` | New endpoint. Turns a list of ids into titles and covers in one call. |
| — | `ApiError`, `ERROR_CODES` | There is a published error model and we had none. |
| — | `Availability`, 4th resolver arg | Makes the detail-only nature of `no_seats` structural rather than remembered. |
| — | `language`, `code`, `type`, `city`, `catalogueUrl` | Fields they have that we did not model. |
| `'subscribe'` in `ACTION_IDS` | **retained** | B2C is not cut — reversed 11 Aug. wokay will supply details. |
| `ContentType` (one union) | `FileFormat` + `WorkType` (two) | These were conflated. One is theirs and real; one is ours and unbacked. |

#### The single most important line is no longer `accessTier`

It is the rule that **`resolveAccess` does not read `accessTier` to decide an action.** wokay ask for written confirmation of this by end of Week 1, and their fallback if we stay silent is to write it as a contract test on their side. The tier survives only as the badge label.

```
rel=open-access                       → ['read','download']
rel=acquisition + canPersist true     → ['borrow'] → ['read','download']
rel=borrow      + canPersist false    → ['borrow'] → ['read']
no acquisition link                   → []
```

Session decides signed-in versus `requires_signin`. Loan decides `requires_loan` versus `available`. Availability gates `no_seats`. Tier decides nothing.

**Exporting the unions as arrays is still right, for a better reason than before.** They are declared `as const` and the union types are derived from them with `typeof X[number]` — so `ACCESS_TIERS` is simultaneously the value that `validate.ts` checks against and that the state gallery iterates, *and* the type that props import. **One declaration, not two that must agree.** Under the JavaScript plan this was a workaround for a missing union type; in TypeScript it is the better idiom, because adding a member and forgetting the other half stops being possible.

> **One ordering note that now matters more.** `AUTH_TYPES` is imported by `InstitutionRow` (§6.6) and `InstitutionDetailView` (§7.2, K6). Deleting it is a three-file edit, and it must land **with** this file rather than after it, or two components ship against a union that no longer exists.

### The adapter conformance test

**Owner:** Prayas, same day

Without an `interface DataAdapter`, nothing stops `MockAdapter` and the later `ApiAdapter` from drifting apart — and "integration is a configuration change" in Week 4 depends entirely on them not drifting.

Write one test suite, exported, that takes any adapter and asserts it: implements all **seven** methods; returns a `Page` shape with an `items` array from every list method; returns objects passing `validate.ts` from `getItem` and `getInstitution`; returns `{items, notFound, denied}` from `getItemsBatch` with all three keys present even when two are empty; surfaces a zero-result feed as `browseInstead: true` rather than as an error or a throw; rejects an unknown id with an `ApiError`-shaped rejection carrying `code: 'NOT_FOUND'` rather than returning `undefined`; and **never accepts an id where the interface takes an href** — the signature is what stops URL construction creeping back in.

`MockAdapter` passes it in Week 1. `ApiAdapter` must pass the identical suite in Week 3 before it is wired up.

> **Its job narrowed on 11 August.** With `implements DataAdapter` on both adapters, the compiler now guarantees the *shape*, so the suite no longer has to check that eight methods exist with the right signatures. What it still has to check is **behaviour the types cannot express**: that a zero-result feed surfaces as `browseInstead` rather than throwing, that an unknown id rejects with a `NOT_FOUND`-shaped error rather than resolving to undefined, and that latency is simulated. Keep it, and drop the shape assertions — they are now dead weight that the build already covers.

**Done when:** merged; `npm run typecheck` clean in CI; `MockAdapter` declared `implements DataAdapter`; the behavioural conformance suite passes against it; and the interface block is pasted into the four Section 09 dispatches.

> **On what to send wokay and flambeau.** Paste the interfaces as-is. TypeScript reads cleanly to a Spring Boot team — `interface ContentItem { … }` is closer to a Java record than JSDoc ever was, which is a small side benefit of the switch. What must not happen is sending prose descriptions of the fields; the whole point of contract-first is that they correct a concrete artefact.

---

### P0-4 · Mock fixtures

**Owner:** Prayas
**Duration:** Day 1 afternoon into Day 2
**Blocks:** everyone on Days 3–5

Roughly **40 content items** and **8 institutions**, spanning all three file formats and all three access tiers. Coverage matters more than volume, and the awkward cases matter more than the clean ones — a fixture set of forty tidy books will pass every test and fail the demo.

> **The samples have landed — derive from them. ✅** Three frozen OPDS fixtures are in `wokay_docs/frozen/` (dated 10 Aug): `01-home-catalogue.json` (root feed), `02-shelf-group.json` (a paginated shelf), `03-publication-detail.json` (book detail). Four items across Book and Audiobook, and all three tiers' link shapes. **R2 is closed** — Prayas is no longer designing blind, and nothing should be hand-written where a real shape exists. Copy their nesting exactly, then add the awkward cases below on top: their samples are tidy, because sample data always is.
>
> **Seven things in the samples contradict the previous version of §P0-3, and all seven are now corrected in `src/model/types.ts`. wokay confirmed on 11 August that the samples are the current contract and their source-of-truth document is outdated — so where the two disagree, the sample wins.**
>
> | Correction | Consequence |
> |---|---|
> | **There is no `accessTier` field.** The tier is derived from the acquisition link's `licenceModel` — `UNLIMITED` → Subscription, `CONCURRENT` → Elite, absent → Open Access | wokay's own mapping, so it is a rule not a guess. Derived in the adapter so components still get a plain field, which is what keeps `AccessTierBadge` and `ContentCard` unchanged |
> | The item id is **not a field** — it is only in the tail of the `self` href | Parse it off `self`. `metadata.identifier` is not the id, and `getItemsBatch` needs the real one |
> | **URN parsing is back** — the feed emits `identifier: 'urn:isbn:…'`, not a flat `isbn` | Reverses the "no URN extraction" line in §5, Prayas. Non-ISBN items carry `urn:tf:catalogue:…` |
> | `contentType` **is not in the feed** — format comes from the acquisition link's MIME | Real field on `items:batch` only. Derivation is the primary path |
> | `@type` **is** in the samples — `schema.org/Book`, `schema.org/Audiobook` | Work type is largely answered; the ask narrows to confirming the vocabulary |
> | `metadata.published` is a **real publication date** (`2020-09-30`) | Closes the `publishedYear` question — it is not the ingest timestamp |
> | `navigation` and `groups` are **two different lists** | Tabs come from navigation; screen 01's sections come from groups. See §5, Prayas |

Mandatory awkward cases:

| Case | Exercises |
|---|---|
| **A book with `editor` and no `author`** — this is real, it is sample `item_ab6` | ContentCard's byline. Our card renders `authors`; with none present it shows a **blank line under the title** on an item wokay supplied. The card must fall back to `editors` |
| **One item per `licenceModel` case, including the absent case** | The tier is **derived**, not sent — `UNLIMITED` → Subscription, `CONCURRENT` → Elite, **key absent entirely** → Open Access. The absent case is the trap: an undefined map lookup and a genuinely missing key are different facts, and treating them the same makes every malformed Subscription item render as free |
| **An item whose acquisition `rel` is unrecognised** | The rel lookup must be exact-match. `http://opds-spec.org/acquisition` is a *string prefix* of the borrow rel, so a `startsWith` implementation reads an Elite item as Subscription and draws a Download button on a `canPersist: false` file |
| Missing `coverUrl` | ContentCard placeholder path |
| No ISBN, no DOI | Detail rendering with an empty identifier block. Note `isbn` is a top-level field and `doi` does not exist backend-side at all |
| 180-character title | Truncation across card, detail and search result |
| **Elite item, `canPersist: false`, `copies: {total: 2}`** | Borrow-then-Read with **no Download button**. `available` is deliberately absent — it is not an item field |
| **A separate `Availability` fixture, `{available: 0, total: 2, queuePosition: 3}`** | `no_seats` → `['waitlist']` on the **detail screen only**. Not an item property |
| **Subscription item, `rel=acquisition`, `canPersist: true`** | Borrow-then-Read-and-Download — the tier that *is* downloadable |
| **Open Access item with no `encrypted` block and no loan** | `['read','download']` with no borrow step, in every session state |
| **An item with no `acquisition` link at all** | `actions: []` — the button must not exist |
| Audiobook, `contentType: 'AUDIO'`, `audio/*` only | FormatSelector with a single non-document format. Audio is never encrypted, on any tier |
| Article with no abstract | Detail screen with an empty body region |
| Institution with no `logoUrl` | InstitutionRow initials fallback — W-17 |
| **A zero-result search feed** — navigation entry, no `publications` key | `browseInstead` → EmptyState's browse-instead action. A missing `publications` key is **not** an error |
| **An `ApiError` envelope per code**, at minimum `NO_ENTITLEMENT`, `ENTITLEMENT_EXPIRED`, `DOWNLOAD_NOT_PERMITTED`, `CONTENT_NOT_READY` | ErrorState copy keyed on `code` — *"your library's subscription has expired"*, not *"not available"* |
| Subscription item, not in any fixture entitlement | `not_entitled` / screen 14 via pending intent |

**Two cases from the previous version are now invalid and must not be written.** `Elite item with copies.available = 0` — `available` is not an item field, so this fixture would encode a shape the backend never emits; it is re-expressed above as a separate `Availability` fixture. `Institution with authType: 'unknown'` — the field does not exist; sign-in is always SAML. The nearest real case is an institution with `signIn` absent from the payload, which is a sign-in-blocked institution rather than a routing fallback.

**One case is downgraded, not deleted.** *Same work in two feeds, same `id`* — shelves are groups on one root feed rather than three independent endpoints, so a work appearing on two shelves is now plausible and cheap rather than a dedup problem across three cursors. Keep the fixture; drop the dedup machinery.

**Fixture validation — `src/model/validate.ts`.** With no compiler, a fixture carrying `accessTier: 'elite'` instead of `'ELITE'`, or `contentType: 'audio'` instead of `'AUDIO'`, will sail through and surface as a blank badge on someone else's screen three days later. Write a small validator — no library needed, ~40 lines — that checks required fields are present and that every union-typed field is a member of the corresponding exported array from `types.js`. `MockAdapter` runs it over the whole fixture set on first load in development and **throws loudly**, naming the item id and the offending field.

**Casing is now a real failure mode, not a hypothetical one.** wokay's enums are uppercase (`OPEN_ACCESS`, `PDF`, `CONCURRENT`) while our own vocabularies stay lowercase (`available`, `requires_loan`, `read`). Five authors will get this wrong at least once each. The validator is what catches it on Day 2 instead of Week 3, and `PropTypes.oneOf(ACCESS_TIERS)` is what catches it at the call site.

This is the highest-value 40 lines in the JavaScript version of this plan. It converts the entire fixture set into a contract test that runs every time anyone starts the app.

**Done when:** merged; `MockAdapter` serves every `DataAdapter` method with realistic latency (150–400ms) so loading states are visible during development rather than discovered in Week 3; `validate.ts` passes over all 40 items and 8 institutions; and the fixture set is either **derived from wokay's Week 1 OPDS samples or their absence is recorded on the dependency board** — see §4.

---

### P0-5 · Component conventions and the gallery route

**Owner:** Khushi authors the convention document and the gallery route; agreed by all five
**Duration:** Day 1, ~1 hour of discussion, written up same day
**Blocks:** all 21 components

Five authors produce five styles unless something prevents it. Three things do: tokens written first (P0-2), one agreed skeleton (this item), and the Day 5 consistency review (§8.2).

**The convention:**

```
src/components/ContentCard/
  ContentCard.tsx          // the component + its exported props interface
  ContentCard.gallery.tsx  // every variant, rendered — this is not optional
  index.ts                 // re-export
```

**Props are declared once: one exported interface above the component.** This previously required two declarations — a JSDoc typedef *and* a `propTypes` block. TypeScript makes one sufficient, and `propTypes` never worked on React 19 regardless. Where the contract already defines a union (`AccessTier`, `ActionId`, `ErrorCode`), **import the type rather than retyping the literals** — that is what stops a component drifting from `types.ts`.

Rules, all five sign up to them:

1. **Props in, callbacks out.** A component receives data and emits events. It does not fetch, does not read a store, does not navigate, does not compute access.
2. **Naming.** Visual variations are `variant`. Lifecycle is `state`. Handlers are `on<Event>`. No component invents a fourth vocabulary.
3. **No raw values.** Every colour, size, spacing and radius comes from `src/theme`. A raw hex in `src/components/` fails review.
4. **States are built in, not added later.** Every component that can load, be empty, error or go offline handles those in its own props from the first commit. Retrofitting states in Week 3 is the failure mode Phase 3 exists to catch, and the cheapest place to prevent it is here.
5. **The one rule that matters — a feature may not introduce a component.** If a feature needs something the library lacks, it is added *to the library*, in the library's files, by whoever needs it, reviewed by the component's original author. Never inside a feature folder. The rule being protected is *one implementation, one location*, not *one author*. This is the rule most likely to break quietly under deadline pressure.

**The gallery route** is a hidden route in the app (`/gallery`, reachable via a long-press on the Profile tab or a dev-only entry) rendering every component in every variant on one scrolling screen. It is the Day 5 review surface and the Week 3 state-matrix surface.

---

### P0-6 · App shell and navigation (F4)

**Owner:** Keshav, paired with Khushi
**Duration:** Day 2
**Blocks:** every screen

- Bottom tab navigator: **Catalogue · Search · Library · Profile**
- Four empty routed screens, each rendering its title and nothing else
- A native-stack inside Catalogue and Search for pushed detail screens
- `TopAppBar` mounted in the navigator, navy, with title / back / search-icon variants
- The `/gallery` route registered
- Route params typed — including `InstitutionDetail: { institutionId: string }` and `ItemDetail: { itemId: string }`

**`BottomTabBar` moves here from Khushi's component list.** In the delivery plan's Section 01 table it is one of her seven, but it is navigation wiring rather than a presentational component, and it blocks all four other people. It belongs in the all-hands Phase 0 with the rest of the shell. Khushi pairs on it — she gets the exposure without owning the blocker, which is the point.

**Done when:** all four tabs navigate, the gallery route opens, and a push-then-back cycle works on both platforms.

---

### P0-7 · The core five components

**Owner:** one each — every person builds the component their own Days 3–5 work leans on hardest
**Duration:** Day 2
**Blocks:** the Days 3–5 feature work

The library is 21 components, not the delivery plan's 22 — see §6.1. Six land in Phase 0; the remaining fifteen are built on Days 3–4, per the schedule in §6.2.

#### The admission test — two conditions, both required

Phase 0 is the layer all five people build on top of. Reworking anything in it costs five people, not one. So a component is only pulled forward if it passes **both** tests:

1. **Common** — more than one person consumes it on Day 3.
2. **Not subject to change** — it is downstream of nothing that is still pending ratification.

Test 2 matters because the design specification is **v0.3, and parts of it are explicitly unratified**. Two items are still waiting on leadership: **L-2** (feed scoping — now a statement of backend fact needing ratification rather than a request, since there is no unauthenticated full catalogue to serve) and **L-5** (shelves-as-tabs, where the shelf set is the institution's to configure). Anything whose shape depends on either stays out of Phase 0 and is built config-driven in Days 3–5, so a reversal costs an edit rather than a rewrite.

**L-3 is less settled than this section assumed.** `borrow` is confirmed on Subscription and Elite and Buy stays removed — but **`subscribe` is retained**, because B2C is not cut (reversed 11 Aug). So the vocabulary is six actions, and the sixth has no flow specified yet: wokay will supply the B2C details later. What replaced that risk is bigger: **action derivation moves out of tier-mapping and into acquisition-link interpretation** (§P0-3). `ActionButton` and `ActionBar` are still excluded from Phase 0, but for a different reason — the shape of what feeds them changed on Day 0.

#### The five that pass

| Component | Author | Common because | Stable because |
|---|---|---|---|
| `ContentCard` | Prayas | The most reused component in the app — screens 01, 04, 05, 08, 09, 17, 18 | Design Spec §4.1 guarantees it explicitly: *"The card, badge and layout are identical in every scope; only the set of items and the single resolved action change."* Even if L-2 is reversed, the card does not move. |
| `TopAppBar` | Keshav | Every screen; mounted by the shell in P0-6 | §2.1 fixes the navy; title / back / search variants are structural. Depends on no pending decision. |
| `SearchInput` | Moktik | Both search pipelines — catalogue and institution — share one shell | A text input. L-5 changes what search is *scoped to*, which lives in the pipeline, not the input. |
| `AccessTierBadge` | Akriti | Consumed by ContentCard on Day 3; screens 01, 04, 05, 09, 18 | The three tiers and their three hex codes are fixed in §2.1 and §3.1 and survive the v0.3 correction untouched. **Q-D closed 11 Aug, but not as expected: there is no tier field.** The tier is derived from the acquisition link's `licenceModel` (UNLIMITED → Subscription, CONCURRENT → Elite, absent → Open Access — wokay's own rule). **The badge component is unaffected**, because the derivation happens in the adapter and the badge still receives a plain `tier` prop. That is the props-only rule paying off for the third time. |
| `Skeleton` | Khushi | Eight screens; every surface needs it before it has data | §2.3 and §4.2 ban spinners outright and require dimensions matching final content. A hard, unrevised design rule. |

**One constraint that keeps `ContentCard` in the stable set:** it must take `item: ContentItem` and `access: AccessResult` as props and **compute nothing**. Design Spec §5.1 — *"The UI must never calculate access rights"* — is what makes the card immune to L-2 and to the access-matrix correction. A card that inspects `accessTier` itself, or maps a tier to an action internally, breaks that immunity.

**This constraint just proved its worth.** The access matrix was inverted on two of three tiers, and the resolver signature changed. Because the card computes nothing, **it is unaffected** — the correction lands entirely inside `resolveAccess`. Any component that had taken the shortcut of reading `accessTier` and mapping it to a button would be rework today. That is the argument for the rule, and it is now an observed one rather than a predicted one.

#### Deliberately excluded, and why

| Component | Author | Why it is not in Phase 0 |
|---|---|---|
| `Tabs` | Prayas | The tab model is **L-5, unratified**, and the tab *set* is now discovered from the root feed's navigation rows rather than being three known endpoints. Built Days 3–5 and **driven from a config list, so tabs are data not code** — which the feed shape has now made mandatory rather than merely prudent. |
| `ActionButton` | Akriti | The vocabulary is **six variants including `subscribe`**, which is retained, and **what derives the actions has not been built yet** — the acquisition-link interpreter is Akriti's own Days 3–5 work. Building the button on Day 2 against a derivation that lands on Day 4 is the wrong order. Variants stay driven off the `ActionId` union so adding or removing one is a map entry. |
| `ActionBar` | Akriti | Same. Renders purely from the `actions` array and computes nothing, so the blast radius stays inside the array. |
| `BottomSheet` | Keshav | Fully specified in §2.3 (16px top radius, drag handle, 60–70% height) and genuinely stable — but only Keshav consumes it before Day 5. Fails test 1, not test 2. |

Each Phase 0 component ships with its `.gallery.tsx` entry and its `propTypes` block the same day.

---

### P0-8 · Section 09 dispatch and the two escalations

**Owner:** Akriti (holds the coordination duty in Week 1)
**Duration:** Day 1 afternoon and Day 2 morning
**Blocks:** nothing this week — which is exactly why it is easy to defer, and why it is scheduled

> **Rewritten, because most of the old wokay list is now answered.** wokay have published their source of truth. Sending a list of twenty-three questions, two-thirds of which their document already answers, spends credibility on the four that matter. **Read their §04 before sending anything.** The dispatch below is what survives.

#### The premise has inverted: silence is no longer safe

The old note read *"silence is safe by design — an unanswered question resolves to they accepted our shape."* That holds where **we** send a shape and they do not object. It **fails** on the four asks below, because wokay have published their own silence-defaults, and on those items **their silence means they choose**:

| wokay need from team1 | By | What they do if we are silent |
|---|---|---|
| Which fields the institution list screen needs, and the sort order | **Wk 1** | Ship a superset and refine |
| **Whether we want subject filters.** Yes means they build OPDS facets | **Wk 1** | **Ship content-type and tier filters only** |
| Confirmation that we render the acquisition link rather than deciding buttons ourselves | **Wk 1** | Document it as a contract test on their side |
| Whether the home screen offers a browse-free-content entry beside find-your-institution | Wk 2 | They ship the public feed regardless; it costs us a button |

**The subject-filter one deletes a designed feature.** Screen 01 has a "Browse by Subject" row and §6.4 has a `SubjectChip` component. Subjects are dynamic — they cannot be known per institution without OPDS facets, which is roughly half a day of wokay's time and is offered *only if we ask*. Silence loses it. **Answer yes.**

#### The dispatch

| When | Action |
|---|---|
| Day 1 | **Answer wokay's four asks**, three of which are due this week. See the four rows below. |
| Day 1 | Send the **three remaining wokay questions** — work type, DOI, and the §3.1 conflict note. Everything else on the old list is answered by their document. |
| Day 1 | Send the flambeau, t4targaryen and leadership sets, each with `src/model/types.ts` attached. Lead with what unblocks us, not with questions. |
| Day 1 | Escalate **L-2 bundled with the §3.1 access-matrix correction**. They are one conversation with leadership, and the second half is the more urgent. |
| Day 1 | **Chase wokay's Week 1 fixtures** — OpenAPI file, three OPDS samples, mock endpoints. This is the single highest-leverage thing in the dispatch; see below. |
| Day 3 | Agree the **sign-in handoff contract** with flambeau. Now narrower: always SAML, and the payload is `idpHint`. |
| Fri | Hand the dependency board to next week's lead, every item in exactly one of: Asked · Answered · Stubbed · Integrated. |

**The four answers, written out so they can be pasted:**

1. **Institution list fields, and sort order.** `id`, `code`, `name`, `type`, `country`, `city`, `logoUrl` — which is exactly what `GET /api/v1/institutions` already returns, so the answer is *your current shape is right, do not add to it*. Sort **alphabetically by `name`, ascending**, as the server default. Recently-used pinning is ours and client-side; it needs nothing from them. On detail we additionally need `branding.logoUrl`, `signIn.idpHint` and `catalogueUrl`, all of which `GET /api/v1/institutions/{id}` already returns.
2. **Subject filters: yes.** Please build the OPDS facets. Screen 01's Browse-by-Subject row and the subject filter dimension both depend on knowing which subjects exist for an institution, and we cannot derive that client-side from a page of twenty results.
3. **Acquisition-link rendering: confirmed.** We render from the acquisition link and its properties. `resolveAccess` does not read `accessTier` to decide an action; the tier drives the badge only. Rule table in §P0-3. A contract test on your side is welcome anyway.
4. **Browse-free-content entry: yes** — but as an *addition*, not a replacement. `GET /opds/v1/public/catalogue` is a skip-institution path we want. It does **not** replace the personal-account option on screen 03, because B2C is retained; that screen may carry both entries. Originally this answer said it replaces the "Personal account" option on the Access Gate sheet (screen 03). It is the second entry point into the app, not a nice-to-have.

**The three questions that remain:**

1. **Work type.** Your `contentType` is `PDF | EPUB | AUDIO`, a file format, and we consume it as one. Separately, screens 04 and 05 are *article* detail and *book* detail — they differ by work type (`book | journal | article | audiobook`), which your model does not carry at all. Can you add it? This is the same shape of ask the access-tier field was, and it now has the longest lead time of anything we need. We have put the field in our fixtures and derive it heuristically meanwhile — please correct our shape rather than starting from scratch.
2. **DOI.** You have a top-level `isbn`. There is no DOI anywhere in the schema, and article detail wants one. Is `identifiers.doi` addable, or should we drop DOI from screen 04?
3. **A conflict we cannot resolve ourselves.** Design Specification §3.1 — a signed document — carries an access matrix that contradicts your §02 table on two of three tiers: it says Open Access requires a Borrow once signed in, and that Elite never borrows. Your document says the opposite and we believe your document. We have corrected the design spec to v0.3 and escalated to leadership. Flagging it because **until it is ratified, nobody can write the resolver against a single agreed source**, and it is your contract that is being treated as correct.

#### The Week 1 fixtures are the highest-leverage item in this dispatch

wokay commit to an **OpenAPI file, three OPDS sample fixtures and mock endpoints in Week 1** — and they say plainly that these *"matter more in Week 1 than any endpoint we could actually build."* They are right, and it lands directly on this plan's stated **#1 risk (R2): Prayas designing the normalisation model blind.**

If the fixtures arrive, that risk largely evaporates and his contingency flips from *"invent a shape and tell wokay"* to *"build to their samples."* If they do not, R2 stands at full size. That makes chasing them a **Monday-morning** action, not a Friday one — and their absence a dependency-board item with a name against it, not a footnote.

---

## 4. The Phase 0 merge gate

End of Day 2. Every line true, or Phase 1 does not start.

- [ ] `main` runs on all five machines, iOS and Android simulator
- [ ] `src/theme/tokens.ts` merged — all 11 colours, all 6 type styles, font, elevation, spacing, radius
- [ ] Inter loads and renders on both platforms
- [ ] No raw hex, raw spacing value or inline shadow anywhere in `src/components/`
- [ ] Every Phase 0 component passes the §P0-7 admission test — common **and** not downstream of L-2 or L-5
- [ ] `src/model/types.ts` merged; `tsconfig.json` committed with `strict: true`; **`npm run typecheck` clean in CI**
- [ ] `AUTH_TYPES` is gone, and nothing imports it — `InstitutionRow` and `InstitutionDetailView` both updated in the same commit
- [ ] Every Phase 0 component has one exported props interface, with contract unions imported rather than retyped
- [ ] ~40 items and 8 institutions in fixtures, including every awkward case in §P0-4
- [ ] **Fixtures reconciled against wokay's Week 1 OPDS samples — or their absence recorded on the dependency board with a name and a date against it**
- [ ] `validate.ts` passes over the entire fixture set, including the uppercase-enum cases
- [ ] Adapter conformance suite passes against `MockAdapter`, all seven methods
- [ ] `MockAdapter` serves every `DataAdapter` method with simulated latency
- [ ] Four tabs navigate; detail push and back work; `/gallery` opens
- [ ] The core five components merged, each with a gallery entry
- [ ] Component convention written up in the README, agreed by all five
- [ ] All four Section 09 sets sent, with the typed shape attached
- [ ] **wokay's four Week 1 asks answered in writing — institution fields and sort, subject facets (yes), acquisition-link rendering, browse-free-content entry**
- [ ] **L-2 escalated in writing, bundled with the §3.1 access-matrix correction**

---

## 5. Days 3–5 — per person

Feature work begins. Each person also builds their remaining components; **components merge before the screens that consume them.**

> **All four blocks below changed.** wokay's contract moved work off Prayas, onto Moktik, changed the shape of Keshav's endpoint, and rewrote Akriti's signature. Read your own block again even if you read this document last week.

### Prayas — adapters and normalisation

**Delivers:** F1, F2, A4 · components `SectionHeader`, `SubjectChip`, `Carousel + PageDots`, `Tabs`
**Difficulty: 7/10, down from 8** — still the hardest block in the week, but the blindness is gone

- **One** OPDS 2.0 adapter normalising to one `ContentItem` — not three. There is one root feed per institution carrying navigation rows and groups; a shelf is a group, paged by following its `next` link.
- **A4 shrinks.** Pagination is `next`-link-only for OPDS — *"follow the next link until it is absent, do not count pages"* — and `page`/`size`/`total` exists **only** on the institutions REST endpoint. Two models, but split cleanly by endpoint family, so the "one interface over both" abstraction is no longer needed. Build the two paths plainly.
- **URN extraction is back — this line is reversed.** The previous version said *"`isbn` is a top-level field, so what was a parsing job is now a field read."* That is true of wokay's Mongo record and **false of the feed.** The frozen samples emit `metadata.identifier: 'urn:isbn:9780367211745'`, and `'urn:tf:catalogue:item_env'` where there is no ISBN. Strip the `urn:isbn:` prefix; treat any other URN as "no ISBN". `doi` still does not exist anywhere.
- **The item id is not a field either.** `item_42` appears only in the tail of the publication's `self` href and in the loan href's `?itemId=`. Parse it off `self` — `identifier` is not the id, and `getItemsBatch` needs the real one.
- **`navigation` and `groups` are two different lists, and this changes `getShelves`.** The root-feed sample carries both: three navigation rows (eBooks, Audiobooks, Open access — pointers, no items) and two groups (New this term, Free to read — each with inline `publications`). wokay's wording is *"navigation rows **and** shelves"*. So **Tabs come from the navigation rows; screen 01's sections come from the groups**, rendered straight from their inline items with no second request. They can also overlap — in the sample, one nav row and one group point at the same href under different titles, so dedupe by href. Shelves are still discovered per institution, so nothing may hardcode a feed list.
- **Format is derived, not read.** There is no `contentType` in the feed — only a MIME on the acquisition link (`application/pdf`, `application/epub+zip`, `audio/mpeg`). It is a real field on `items:batch` only. An unmapped MIME must surface as undefined and fail validation, never default to PDF.
- **Never build a URL.** `catalogueUrl` is handed over by the institution detail response; every subsequent hop is an `href` from a feed. The single permitted construction is expanding `".../search{?query}"`.
- The work-type derivation (`book | journal | article | audiobook`) is **unbacked** — keep it in exactly one function so replacing it with a real field is a one-file edit.
- A `useNetworkStatus` hook — moved here from Khushi's `OfflineBanner`, since it is a subscription to native network state rather than presentation

**Contingency:** wokay's Week 1 samples have not landed — build to the field names in §P0-3, which are theirs, and chase the fixtures Monday morning.

**R2 has shrunk, conditionally.** Prayas is no longer designing blind: the `catalogueItems` record is published field by field, and wokay commit to three OPDS samples in Week 1. **If those samples land, R2 largely evaporates.** If they do not, it stands at full size — which is why chasing them is a Day 1 dispatch item (§P0-8) rather than a Wednesday one. Keep the Day 3 pairing hour with Akriti either way; the value has moved from *validating a guess* to *walking their shape against ours*.

### Moktik — search shell and matching

**Delivers:** search shell, B1 · components `FilterChip`, `VoiceOverlay`
**Difficulty: 7/10, up from 6 — and the schedule risk in the week has moved here**

- The shared search shell — the interaction layer both pipelines consume. **Unchanged, and still the safe part.**
- **B1 is contradicted, and this is the correction to read carefully.** The previous version read *"matching, tokenisation and ranking over local fixtures. All three are ours."* **All three are wokay's.** Catalogue search is server-side, non-negotiably: *"results are filtered by what the institution is entitled to. Searching on the client would mean shipping the whole catalogue to the phone and reimplementing entitlement rules there, and a member must never see a result they cannot open. So: we filter, you render."* B1 becomes query-state management, the templated-link expansion, results rendering, paging by `next`, and the empty/error states — not a search engine.
- **Q-E resolves to the contingency.** Fetch-and-filter is fine as a Week 1 stand-in over fixtures, but it must sit **behind the pipeline interface** and it cannot become the design. Anything that would survive into Week 4 as client-side matching is wasted work.
- **Two riders that belong in the UI, not just the code.** Search is **metadata-only** — title, authors, subjects, description. It does **not** search inside books. Put that in the placeholder or helper copy, or a reviewer typing a phrase from page 88 files a bug. And a **zero-result response is a navigation feed, not an empty array**: render `browseInstead` as a browse-instead affordance; a missing `publications` key is not an error.
- Filters are **query parameters, applied before pagination** — never client-side within a fetched page, or the pager says "page 1 of 12" over three visible results. `contentType` and `accessTier` are free, fixed enums: hardcode the chips and send the parameter. Subject filters need facets and depend on our Week 1 answer (§P0-8).

**Contingency:** the search endpoint is a **Week 4** deliverable. Build the pipeline against fixtures behind the interface.

**The new schedule risk in this week, named.** wokay's catalogue search lands **Week 4**, and team1's Week 4 is integration plus the BDD suite. Moktik therefore builds against fixtures for three weeks and integrates the real endpoint in the busiest week of the plan. Nothing about that is unworkable, but it is a genuine risk that did not exist in the previous version, and it should go on the dependency board now rather than be discovered in Week 4. The mitigation is the interface boundary: if fetch-and-filter is strictly behind it, the Week 4 swap is a configuration change.

### Keshav — institutions

**Delivers:** B9, C1 · components `BottomSheet`, `InstitutionRow`, `ListRow`
**Difficulty: 4/10 — but the highest reliability bar in the week**

- B9 institution search — a **separate pipeline** from catalogue search, sharing only the shell
- C1 institution list, with recently-used pinned above the full list
- **Server-side and paged, not client-side.** `GET /api/v1/institutions?q=&country=&page=&size=` returning `{items, page, size, total}`, unauthenticated. The previous version specified *"client-side search over the 8-institution fixture"* — that becomes a paged server query behind the same interface. Recently-used pinning stays ours and client-side.
- **`authType` routing is dead.** Institutional sign-in is **always SAML**; there is no field to branch on. `navigation/authRouting.ts` stops being a branching file and becomes an `idpHint` pass-through: read `signIn.idpHint` from `GET /api/v1/institutions/{id}` and hand it to flambeau. `AUTH_TYPES` is deleted from `types.js`, so `InstitutionRow` must be updated in the same commit.
- An inactive institution returns **404, not 403** — deliberately, so its existence is not disclosed. Treat 404 on institution detail as "not found", never as "forbidden".

**C2 (institution detail) moves to Khushi** — see §7.

**This is the earliest real thing team1 gets from anyone.** The institution endpoints are **Week 2** — ahead of the root feed (Wk 3) and search, shelves and the public feed (Wk 4). Keshav is the first person on the team who can delete a mock.

**Contingency:** none needed on the schema — W-3 is answered. If the Week 2 endpoint slips, the fixture already matches their field names.

**Why the reliability bar is high.** Per Section 04 of the delivery plan: institution search is on the authentication critical path, catalogue search is not. If catalogue search breaks, a user browses instead. If institution search breaks, the user cannot sign in, cannot obtain entitlements, and cannot read anything. Low difficulty, high consequence — give this code review attention, not help.

### Akriti — the access spine

**Delivers:** D1 skeleton, F6 · components `ActionButton`, `ActionBar` · plus the coordination duty
**Difficulty: 7/10 for the design, plus ~25% of the week on coordination**

- **`resolveAccess(item, session, loan, availability)` — the signature changed, and this is the expensive kind of change to get late.** It is a **link-and-properties interpreter, not a rules engine.** No branch may read `item.accessTier` to decide an action; the derivation table is in §P0-3. Tier survives as the badge label only.
- The fourth argument is nullable and is `null` on every list surface, because `copies.available` is not in the feed. `no_seats` is returned only when it is non-null and `available === 0` — which is what makes `no_seats` structurally detail-only.
- F6 pending-intent store against a fake auth round trip. **Persist to disk, not memory** — SAML is confirmed as the only sign-in path, so the round trip is real, and disk is the correct choice either way.
- `ActionButton` variants: Read, Download, Borrow, Sign in, Waitlist, **Subscribe**, disabled. **`Subscribe` is retained** — B2C is not cut, and wokay will supply the details.
- `ActionBar` renders purely from the `actions` array and computes nothing.
- **Own the `ApiError` → `ErrorState` copy map.** Denials carry an enumerated `code` specifically so the app can say *"your library's subscription has expired"* rather than *"not available"*. One map, keyed on `code`, living beside the resolver.

**Contingency:** none on the tier — Q-D is answered, `accessTier` already exists. The live contingency is the **work type** ask: derive it heuristically in Prayas's single function and mark screens 04/05 as running on derived data.

**Note.** Only a skeleton this week, so the grade is for design difficulty rather than volume. `resolveAccess` is the spine — everything downstream of it is presentation — and a wrong signature is expensive. **That is exactly what was found here, on Day 0 rather than in Week 3, which is the good version of this news.** This rises to ~9/10 in Week 2 with the real interpreter, the loan rule and Borrow orchestration.

### Khushi — see §7.

---

## 6. The component library — all 21 components, designed

This section is the canonical component reference for the whole team. Every prop contract in the app is defined here once. If a component's shape needs to change, it changes here first.

### 6.1 Scope — the library is 21 components, not 22

The delivery plan's Section 01 inventory lists 22. **`ProgressBar` is removed**, because it appears only on screens **07 (reader view)** and **08 (library / downloads)** — both t4targaryen, CAP-7. No component team1 builds consumes it, and the library is scoped to team1's own screens.

> **Put this on the dependency board.** t4targaryen question 4 asks *"Who owns the downloaded and offline indication on catalogue items — you or us?"* If the answer is "team1," `ProgressBar` comes back into the library. It is roughly two hours to add, so the risk is small — but it should be a decision, not a discovery.

> ### ⚠ An ownership conflict on My Library — named, dated, and not silently absorbed
>
> **wokay's §03 lists team1 as owning "find institution, browse, search, detail, my library"**, and they pitch `POST /api/v1/catalogue/items:batch` explicitly at a My Library screen: *"Loan records carry item ids only, so a My Library screen needs metadata for twelve titles in one call rather than twelve."* This document excludes screen 08 as t4targaryen's, and removed `ProgressBar` on that basis.
>
> **Our position, and it is the one to send:** screen 08 is the offline shelf. Everything decision-bearing on it — download progress, bytes on disk, the keystore, guest-download persistence — is CAP-7. Screen 07 (reader) and screen 08 are one block and they belong together. The library stays at **21 components** and screen 08 stays out.
>
> **But the conflict is real and it is a whole screen, not two hours.** If wokay are right, `ProgressBar` returns, screen 08 joins team1's list, and Khushi's block is re-cut. So:
>
> - **Owner:** Akriti (coordination duty, Week 1). **Resolve by Friday of Week 1**, not "at the gate".
> - **Send to:** t4targaryen (their question 4, escalated from a footnote to a decision) **and** wokay, as a correction to their §03.
> - **If unresolved by Friday:** it goes to next week's lead as an **open escalation**, not a resolved item.
>
> **One thing to accept from wokay regardless of who owns the screen.** `getItemsBatch(ids)` belongs in our `DataAdapter` either way — it is the fix for "twelve ids, twelve round trips", and any surface holding item ids without metadata needs it. It is in §P0-3. Adding a data-layer method is not the same as accepting a screen.

**team1's screens:** 01, 02, 03, 04, 05, 06, 09, 10, 11, 12, 14, 15, 16, 17, 18.
**Excluded:** 07 and 08 (t4targaryen), 13 (flambeau).

### 6.2 Build schedule — all 21 across Days 1–5

| | Day 1 | Day 2 | Day 3 | Day 4 | Day 5 |
|---|---|---|---|---|---|
| **Prayas** | foundation | **ContentCard** | SectionHeader, SubjectChip, Carousel+PageDots, Tabs | features | features |
| **Moktik** | foundation | **SearchInput** | FilterChip, VoiceOverlay | features | features |
| **Keshav** | foundation | **TopAppBar**, BottomTabBar\* | BottomSheet, InstitutionRow, ListRow | features | features |
| **Akriti** | foundation | **AccessTierBadge** | ActionButton, ActionBar | features | features |
| **Khushi** | tokens | **Skeleton** | EmptyState, ErrorState | OfflineBanner, FormatSelector, gallery | C2, review |

\* Built inside the P0-6 app shell — Keshav with Khushi pairing.

**Bold = Phase 0.** The four with feature work finish their components on Day 3 and start features on Day 4, which honours the delivery plan's Phase 1 rule: *nobody starts a feature until the library is merged.* Khushi runs into Day 4 because she has no feature work, so her spillover blocks nobody.

**Where the three-day line falls: 19 of 21 merged by end of Day 3.** Six land in Phase 0 (Day 2), thirteen more on Day 3. The two outstanding are Khushi's `OfflineBanner` and `FormatSelector`, plus the gallery and C2, which are not components. Neither outstanding item is on anyone else's path.

**Per author:** Prayas 5 · Keshav 4 · Khushi 5 · Moktik 3 · Akriti 3 · shell 1.

#### Two hard ordering constraints

- **`AccessTierBadge` before `ContentCard`** — both Day 2, and the card renders the badge. Badge in the morning, card in the afternoon, or Prayas stubs it and reworks it.
- **`ActionButton` before `ActionBar`** — both Akriti, Day 3. The bar is a layout wrapper around the button; building it first means building it twice.

### 6.3 Rules that apply to every component

> **Read the prop blocks in §6.4 to §6.8 as TypeScript interfaces.** They were written as `propTypes` and have not been transcribed one by one, because the *content* is unchanged — the same props, the same unions, the same required/optional split. Only the syntax differs. The translation is mechanical:
>
> ```ts
> // written as:                          read as:
> tier: PropTypes.oneOf(ACCESS_TIERS)     tier: AccessTier              // import the type
>   .isRequired                           //   — required, no `?`
> size: PropTypes.oneOf(['sm','md'])      size?: 'sm' | 'md'
> onPress: PropTypes.func.isRequired      onPress: () => void
> onVoicePress: PropTypes.func            onVoicePress?: () => void     // absence is meaningful
> item: PropTypes.object.isRequired       item: ContentItem             // now actually checked
> ```
>
> Two things get strictly better in the translation. `PropTypes.object` becomes a real type, so `ContentCard`'s `item` prop is checked rather than merely labelled. And every `oneOf(SOME_ARRAY)` becomes an imported union, so a component cannot drift from `types.ts` without failing the build.

Not repeated in the entries below.

1. **Props in, callbacks out.** No fetching, no store reads, no navigation, no access computation. Design Spec §5.1.
2. **Every value from `theme/tokens.ts`.** A raw hex, a raw spacing number or an inline shadow fails review.
3. **One exported props interface above the component.** No `propTypes` — see §1.5a.
4. **A `.gallery.tsx` entry the same day**, covering every variant listed in its spec.
5. **Union types and values come from `types.ts`** — `tier: AccessTier`, and `ACCESS_TIERS` where a runtime list is needed. Never a retyped literal union.

#### State coverage — which components must handle which states

| Component | loading | empty | error | offline | 3 tiers |
|---|:-:|:-:|:-:|:-:|:-:|
| ContentCard | ● | – | – | ● | ● |
| InstitutionRow | ● | – | – | ● | – |
| SearchInput | – | – | – | ● | – |
| Tabs | ● | – | – | – | – |
| AccessTierBadge | – | – | – | – | ● |
| ActionButton | ● | – | – | ● | ● |
| ActionBar | ● | – | – | – | ● |
| FormatSelector | – | ● | – | ● | – |
| EmptyState | – | ● | – | – | – |

"offline" for a card or row means *degraded but usable* — Design Spec §4.2 requires the library and institution list to keep working behind the banner.

#### Three corrections to this table, from wokay's contract

1. **`ContentCard` can never render `no_seats`.** `copies.available` is not in the feed — the acquisition link carries `copies.total` only, and `available` comes from a separate per-item flambeau call made on the detail screen. So an Elite card with zero free copies is indistinguishable from one with copies free, and resolves to `requires_loan`. FL-11's contingency (*"only surface `no_seats` reactively"*) is now the actual design, not the fallback. **Do not build a `no_seats` variant of the card**, and do not put one in its gallery entry.
2. **`FormatSelector` keys off `canPersist`, not the tier.** An Elite item must not offer Download; the acquisition link says so via `canPersist: false`, and the server refuses the intent anyway (`DOWNLOAD_NOT_PERMITTED`). The selector must not infer this from `accessTier` — same rule as the resolver.
3. **`EmptyState` needs a browse-instead action.** A zero-result search returns a feed carrying a navigation entry pointing back at the catalogue, surfaced as `Page.browseInstead`. That is a *third* functional variant alongside the two copy variants in §4.2, and it takes an `onBrowse` callback. See §6.8, component 18.

---

### 6.4 Prayas — 5 components

#### 1 · `ContentCard` — Phase 0, Day 2 · Difficulty 6

The most reused component in the app, and the one the design specification explicitly guarantees is invariant across both feed scopes — signed-in and anonymous. (There were three; the B2C scope is deleted with individual subscribers.)

**Screens:** 01, 04, 05, 09, 17, 18
**Layout:** thumbnail left, metadata right, 8px radius, `elevation.card` shadow (Design Spec §2.3)
**Variants:** `default` · `compact` (search results) · `featured` (carousel)
**States:** `loading` → `ContentCard.Skeleton` · `offline` (cover falls back to placeholder, no network fetch)

```js
ContentCard.propTypes = {
  item:   PropTypes.object.isRequired,     // ContentItem — see model/types.js
  access: PropTypes.shape({                // AccessResult, already resolved
    tier:    PropTypes.oneOf(ACCESS_TIERS).isRequired,
    state:   PropTypes.oneOf(ACCESS_STATES).isRequired,
    actions: PropTypes.arrayOf(PropTypes.oneOf(ACTION_IDS)).isRequired,
  }).isRequired,
  variant: PropTypes.oneOf(['default', 'compact', 'featured']),
  onPress: PropTypes.func.isRequired,
}
```

**Renders:** cover or placeholder, title, authors, publisher, content-type indicator, `AccessTierBadge` driven by `access.tier`, and **one** action derived from `access.actions[0]`.
**Tokens:** `radius.card`, `elevation.card`, `color.surface`, `color.border`, `type.sectionHeader` (title), `type.meta` (authors, publisher), `space.md`.

**`ContentCard.Skeleton` is authored here, not by Khushi.** It composes Khushi's `Skeleton` primitive, but Prayas owns the dimensions — §2.3 requires placeholders matching the exact dimensions of the final content, and only the card's author knows those.

**Must not:** inspect `item.accessTier`, map a tier to an action, or decide which action to show. It receives `access` already resolved. This constraint is what makes the card immune to L-2 and to the shift of action derivation into link interpretation — see the P0-7 admission test.

**Must not render `no_seats`.** It is unreachable on a card: `copies.available` is not in the feed. `access.actions` will never contain `waitlist` on a list surface. No variant, no gallery entry — see §6.3.

**The content-type indicator is the file format**, `PDF | EPUB | AUDIO`, which is wokay's `contentType`. It is **not** the work type (`book | journal | article`), which the backend does not model. If the indicator is meant to read "Book" or "Article", it is running on derived data — flag it in the gallery entry so the Day 5 review sees it.

**Done when:** all three variants plus skeleton and offline in the gallery; the 180-character-title fixture truncates cleanly in all three; the missing-cover fixture shows the placeholder.

#### 2 · `SectionHeader` — Day 3 · Difficulty 2

**Screens:** 01 ("Featured", "Browse by Subject"), 06 ("Recently used", "All Institutions"), 09
**Variants:** `default` · `with_action` (trailing "See all")

```js
SectionHeader.propTypes = {
  title:       PropTypes.string.isRequired,
  actionLabel: PropTypes.string,
  onAction:    PropTypes.func,
}
```

**Tokens:** `type.sectionHeader`, `color.textPrimary`, `color.primary` (action), `space.md` / `space.lg`.
**Done when:** both variants in the gallery; a long title wraps rather than pushing the action off-screen.

#### 3 · `SubjectChip` — Day 3 · Difficulty 2

Outlined teal pill for the Browse-by-Subject row.

**Screens:** 01, 09 · **States:** `default` · `selected` · `disabled`

```js
SubjectChip.propTypes = {
  label:    PropTypes.string.isRequired,
  selected: PropTypes.bool,
  disabled: PropTypes.bool,
  onPress:  PropTypes.func.isRequired,
}
```

**Tokens:** `color.primary` (border and selected fill), `radius.pill`, `type.smallLabel`, `space.sm`.

**Deliberately distinct from `FilterChip`** — different surface, different semantics (a subject navigates, a filter refines). They look similar and will drift together if not reviewed side by side on Day 5.

#### 4 · `Carousel` + `PageDots` — Day 3 · Difficulty 5

Featured content, top of screen 01. Two files, one folder.

**Screens:** 01
**States:** `loading` (skeleton slides) · `single_item` (no dots) · `empty` (renders nothing, not an empty box)

```js
Carousel.propTypes = {
  items:         PropTypes.array.isRequired,   // ContentItem[]
  renderItem:    PropTypes.func.isRequired,    // caller supplies the card
  onIndexChange: PropTypes.func,
}
```

**Must not** hardcode `ContentCard` — take `renderItem`, so the carousel is reusable and testable with a placeholder.

**Watch:** the trickiest component on Day 3 — horizontal paging, momentum, dot sync. It is also the one component that can safely slip: screen 01 renders without a carousel.

#### 5 · `Tabs` — Day 3 · Difficulty 4 · ⚠ L-5 pending

**Screens:** 01 (feed tabs), 04 (detail sections), 09
**Variants:** `segmented` · `underline`

```js
Tabs.propTypes = {
  tabs: PropTypes.arrayOf(PropTypes.shape({
    id:    PropTypes.string.isRequired,
    label: PropTypes.string.isRequired,
  })).isRequired,
  activeId: PropTypes.string.isRequired,
  variant:  PropTypes.oneOf(['segmented', 'underline']),
  onChange: PropTypes.func.isRequired,
}
```

**Built from a config list, deliberately.** The three-feed-tab model is **L-5, unratified** — design screen 01 still shows a single merged list. Tabs must be *data, not code*, so a reversal is a config edit rather than a rewrite of screens 01 and 09.

Each feed tab holds its own pagination cursor and result count (A0). The component does not manage that, but `onChange` must be the only way the active tab changes, so the consumer can swap cursors cleanly.

---

### 6.5 Moktik — 3 components

#### 6 · `SearchInput` — Phase 0, Day 2 · Difficulty 3

Consumed by **both** search surfaces, which are separate pipelines sharing one shell (delivery plan Section 04). Catalogue search and institution search differ on corpus, criticality and offline behaviour — but they share one input.

**Screens:** 01, 06, 09
**States:** `default` · `focused` · `filled` (clear button) · `disabled` · `offline`

```js
SearchInput.propTypes = {
  value:        PropTypes.string.isRequired,
  placeholder:  PropTypes.string,
  onChangeText: PropTypes.func.isRequired,
  onSubmit:     PropTypes.func,
  onClear:      PropTypes.func,
  onVoicePress: PropTypes.func,   // omitted → no mic icon (institution search)
  disabled:     PropTypes.bool,
}
```

**Tokens:** `color.border`, `color.surface`, `radius.card`, `type.body`, `space.md`.

**The mic icon is opt-in via `onVoicePress`.** Voice (B11) is catalogue-scoped only; institution search must not show it.
**Must not** debounce, search, or hold query state. It is a controlled input — the pipeline owns all of that.

#### 7 · `FilterChip` — Day 3 · Difficulty 2

**Screens:** 09, 12 · **States:** `unselected` · `selected` · `selected_with_count` ("Subject · 3")

```js
FilterChip.propTypes = {
  label:    PropTypes.string.isRequired,
  selected: PropTypes.bool,
  count:    PropTypes.number,
  onPress:  PropTypes.func.isRequired,
  onRemove: PropTypes.func,      // × affordance when selected
}
```

Six filter dimensions eventually feed this. Their status is now known, and it changes which ones are cheap:

| Dimension | Status |
|---|---|
| **Content type** (`PDF`/`EPUB`/`AUDIO`) | **Free.** Fixed enum, a query parameter on endpoints we already call. Hardcode three chips and send the parameter |
| **Access tier** (`OPEN_ACCESS`/`SUBSCRIPTION`/`ELITE`) | **Free.** Also a fixed enum. Q-D is answered — this is no longer blocked on anything |
| **Subject** | **Needs OPDS facets, and wokay build them only if we ask.** Subjects are dynamic, so they cannot be known per institution client-side. Answered *yes* on Day 1 (§P0-8) — if that answer is not sent, this dimension does not exist |
| Publisher, publication year | Unconfirmed that the fields are consistently populated |
| Open Access | Collapses into the access-tier dimension |

**Filters are query parameters, and they apply before pagination.** Never filter inside a fetched page — *"filter within a page of twenty and you show three results while the pager still says page 1 of 12."*

The chip must still render correctly when its dimension has zero available options: the slot is wired but empty, not crashed. That case is now specifically the **subject** dimension, if facets are not built.

#### 8 · `VoiceOverlay` — Day 3 · Difficulty 6

Dark immersive overlay with live transcription.

**Screens:** 11 · **States:** `listening` (animated) · `transcribing` · `error` (no permission, no speech) · `success`

```js
VoiceOverlay.propTypes = {
  visible:      PropTypes.bool.isRequired,
  state:        PropTypes.oneOf(['listening','transcribing','error','success']).isRequired,
  transcript:   PropTypes.string,
  errorMessage: PropTypes.string,
  onCancel:     PropTypes.func.isRequired,
}
```

**Must not** own speech recognition. Voice is "simply a second way to produce a query string" — the recogniser lives in the catalogue search pipeline; the overlay stays a pure view.

---

### 6.6 Keshav — 4 components (including the shell component)

#### 9 · `TopAppBar` — Phase 0, Day 2 · Difficulty 3

**Screens:** every screen. Mounted by the navigator in P0-6.
**Variants:** `title` · `title_with_back` · `title_with_search` · `title_with_back_and_action`

```js
TopAppBar.propTypes = {
  title:    PropTypes.string.isRequired,
  onBack:   PropTypes.func,        // presence renders the back affordance
  onSearch: PropTypes.func,        // presence renders the search icon
  action:   PropTypes.node,
}
```

**Tokens:** `color.navy` (background — §2.1, "top navigation bars"), `type.pageTitle`, white foreground.
**Done when:** all four variants in the gallery; safe-area insets correct on both platforms; long titles truncate rather than wrap.

#### 10 · `BottomTabBar` — Phase 0, Day 2, inside P0-6 · Difficulty 4

Four tabs: **Catalogue · Search · Library · Profile** (Design Spec §4).

**Screens:** 01, 09, 10 — and 08, which is t4targaryen's screen but consumes our shell.
**States:** `active` / `inactive` per tab; optional badge dot on Library.
**Tokens:** `color.primary` (active), `color.textSecondary` (inactive), `type.smallLabel`.

Built as part of the app shell by **Keshav with Khushi pairing**. It is navigation wiring rather than presentation, and it blocks all four other people, so it does not sit on one person's component list.

#### 11 · `BottomSheet` — Day 3 · Difficulty 5

Fully specified in Design Spec §2.3: **16px top radius, drag handle, 60–70% screen height.**

**Screens:** 02 (sign-in), 03 (Access Gate), 12 (filter / sort) — three screens, one sheet
**States:** `hidden` · `visible` · `dragging` · `dismissing`

```js
BottomSheet.propTypes = {
  visible:     PropTypes.bool.isRequired,
  heightRatio: PropTypes.number,     // default 0.65, per §2.3
  onDismiss:   PropTypes.func.isRequired,
  dismissible: PropTypes.bool,       // false = must choose (Access Gate)
  children:    PropTypes.node.isRequired,
}
```

**`dismissible: false` is a product decision, not a styling one.** Screen 03 presents a routing choice; whether it can be swiped away needs an answer. Default to dismissible and raise it at the first demo.
**Must not** know what it contains — sign-in, Access Gate and Filter all pass children.

#### 12 · `InstitutionRow` — Day 3 · Difficulty 2

**Screens:** 06 (list), 10 (current selection in Profile)
**Variants:** `default` · `selected` · `recently_used` (pinned above the list on 06)
**States:** `loading` · `offline`

```js
InstitutionRow.propTypes = {
  institution: PropTypes.shape({
    name:    PropTypes.string.isRequired,
    country: PropTypes.string.isRequired,
    city:    PropTypes.string,
    logoUrl: PropTypes.string,          // was crestUrl — W-17 answered
  }).isRequired,
  variant: PropTypes.oneOf(['default','selected','recently_used']),
  onPress: PropTypes.func.isRequired,
}
```

**`authType` is gone from this contract.** W-15 is answered: institutional sign-in is **always SAML** and there is no field on wokay's institution record to configure it. The prop is deleted, `AUTH_TYPES` is deleted from `types.js`, and there is no sign-in-type affordance to render. **This edit lands in the same commit as §P0-3**, not after it.

**The logo fallback is required, not optional.** W-17 is answered — `logoUrl` exists — but it is optional in their payload, so a null is a normal case rather than a missing field. Fall back to an initials monogram; the fixture set includes a logo-less institution specifically to force this.

#### 13 · `ListRow` — Day 3 · Difficulty 1

Settings and profile rows. The simplest component in the library, and a good first merge on Day 3.

**Screens:** 10
**Variants:** `navigation` (chevron) · `toggle` (switch) · `value` (trailing text) · `destructive` (sign out, `color.error`)

```js
ListRow.propTypes = {
  label:    PropTypes.string.isRequired,
  variant:  PropTypes.oneOf(['navigation','toggle','value','destructive']),
  value:    PropTypes.oneOfType([PropTypes.string, PropTypes.bool]),
  onPress:  PropTypes.func,
  onToggle: PropTypes.func,
}
```

---

### 6.7 Akriti — 3 components

#### 14 · `AccessTierBadge` — Phase 0, Day 2 · Difficulty 2

**Build before `ContentCard`, same day.**

**Screens:** 01, 04, 05, 09, 18
**Variants:** exactly three, fixed by §2.1 and §3.1 — Open Access `color.success`, Subscription `color.subscription`, Elite `color.elite`
**Sizes:** `sm` (card) · `md` (detail)

```js
AccessTierBadge.propTypes = {
  tier: PropTypes.oneOf(ACCESS_TIERS).isRequired,
  size: PropTypes.oneOf(['sm','md']),
}
```

**Tokens:** the three tier colours, `radius.pill`, `type.smallLabel`.
**Must not** accept a `ContentItem` or read `item.accessTier`. It takes a resolved `tier` — Design Spec §3.1: the badge is "driven by the resolver, never by raw licence data."

**Its gallery entry iterates `ACCESS_TIERS`** rather than hardcoding three. This is the payoff for exporting the unions as runtime arrays (§P0-3).

#### 15 · `ActionButton` — Day 3 · Difficulty 5

The most variants of anything in the library.

**Screens:** every screen
**Actions:** the five `ActionId` values — `read` · `download` · `borrow` · `signin` · `waitlist`
**Emphasis:** `primary` · `secondary` · `tertiary`
**States:** `default` · `pressed` · `loading` (in-flight borrow) · `disabled`

```js
ActionButton.propTypes = {
  action:   PropTypes.oneOf(ACTION_IDS).isRequired,
  emphasis: PropTypes.oneOf(['primary','secondary','tertiary']),
  loading:  PropTypes.bool,
  disabled: PropTypes.bool,
  onPress:  PropTypes.func.isRequired,
}
```

**Label and colour derive from a single map keyed by `ActionId`** — not a switch statement scattered through the component. That map is what made deleting `subscribe` a one-line change when B2C was cut, and it is why the same design stays: any future vocabulary change must be one map entry.

**`subscribe` is retained** — reversed 11 Aug, B2C is not cut. It needs a style entry in the `ActionId` map like every other action. What it does **not** yet have is a specified flow: wokay will supply the B2C details later, so build the button and leave the handler to Week 2. Buy stays removed. The button has five actions, not six, and `ACTION_IDS` in §P0-3 reflects that.

`waitlist` uses `color.wait` (amber — §2.1, "no seats available, waitlist actions"). **It is reachable on item detail only**, for Elite items with no free copies, and it creates a hold rather than a loan.

**The `loading` state is real, not defensive.** Borrow is a network call against finite copies, and the optimistic flip (D11) needs somewhere to show in-flight. On Elite it is genuinely two-outcome — `201` with a loan, or `202` with a queue position — so the in-flight state resolves to two different buttons.

#### 16 · `ActionBar` — Day 3, after `ActionButton` · Difficulty 4

Sticky bottom bar on item detail.

**Screens:** 04, 05
**States:** `one_action` · `two_actions` (Read + Download) · `no_action` (Access Restricted, screen 14) · `loading`

```js
ActionBar.propTypes = {
  actions:  PropTypes.arrayOf(PropTypes.oneOf(ACTION_IDS)).isRequired,
  loading:  PropTypes.bool,
  onAction: PropTypes.func.isRequired,   // (actionId) => void
}
```

**Renders purely from the `actions` array and computes nothing** — Design Spec §5.1. Given `[]` it renders the Access Restricted treatment, which is now reachable on **two** paths: an institutional user arriving at a non-entitled Subscription or Elite title through pending intent, and **an item wokay send with no acquisition link at all** — *"if we do not send a link, the button does not exist."* The second path is why `[]` is a normal input rather than an edge case.

**`two_actions` is Open Access and Subscription only.** Elite resolves to `['read']` — one button, never two, because `canPersist: false` and the server refuses a download intent regardless. The gallery entry should show all three shapes side by side, since "Elite has one button" is the single most likely thing to be got wrong on screens 04 and 05.

**Must not** decide which action to show. `resolveAccess` already did.

---

### 6.8 Khushi — 5 components

Her governing rule — **props in, callbacks out**: no `useEffect`, no `async`, no navigation, no adapter imports. See §7 for why, and for her day-by-day ordering.

#### 17 · `Skeleton` — Phase 0, Day 2 · Difficulty 3

Design Spec §2.3 and §4.2 **ban spinners outright**. Every surface needs this before it has data.

**Screens:** 01, 04, 05, 06, 09, 15, 16
**Variants:** `block` (rectangle) · `text` (line, width ratio) · `circle` (avatar, crest)

```js
Skeleton.propTypes = {
  variant:  PropTypes.oneOf(['block','text','circle']),
  width:    PropTypes.oneOfType([PropTypes.number, PropTypes.string]),
  height:   PropTypes.number,
  animated: PropTypes.bool,     // shimmer; default true
}
```

**The primitive only.** Composite skeletons — `ContentCard.Skeleton`, `InstitutionRow.Skeleton` — are built by each component's own author, because §2.3 requires dimensions matching the final content exactly and only that author knows them. This split is deliberate, and it is also why Khushi is blocked on nobody.

#### 18 · `EmptyState` — Day 3 · Difficulty 2

Design Spec §4.2 requires **two distinct copy variants**, and the distinction is functional: "no results for a query" offers no obvious next action, "no results for your filters" does.

**Screens:** 06, 09, 17
**Variants:** `no_query_results` · `no_filter_results` (offers Clear filters) · `no_content` · **`browse_instead`**

```js
EmptyState.propTypes = {
  variant: PropTypes.oneOf(['no_query_results','no_filter_results',
                            'no_content','browse_instead']).isRequired,
  query:   PropTypes.string,
  onClearFilters: PropTypes.func,
  onBrowse:       PropTypes.func,   // browse_instead — follows the feed's
                                    // navigation entry back to the catalogue
}
```

Copy per the design spec: *"No articles or books match…"* / *"Try adjusting your filters…"*

**`browse_instead` is a fourth variant required by the backend contract, not a nicety.** OPDS forbids empty arrays, so a zero-result catalogue search does not return an empty list — it returns **a feed containing a navigation entry pointing back at the catalogue**, surfaced to this component as `Page.browseInstead`. Render it as a browse-instead affordance and call `onBrowse` with the href the feed supplied. Two things must not happen: treating a missing `publications` key as an error, and constructing the browse URL ourselves.

**Search copy belongs here too.** Catalogue search is **metadata-only** — title, authors, subjects, description — and does not search inside books. The `no_query_results` copy is the natural place to say so, so a reviewer typing a phrase from page 88 understands rather than files a bug. Coordinate the exact wording with Moktik, who owns the pipeline.

#### 19 · `ErrorState` — Day 3 · Difficulty 2

**Screens:** 04, 05, 14
**Variants:** `network` (Retry) · `not_found` · **`access_restricted`** (Learn more, no retry) · **`not_ready`** (content still ingesting — retry is meaningful)

```js
ErrorState.propTypes = {
  variant:     PropTypes.oneOf(['network','not_found',
                                'access_restricted','not_ready']),
  message:     PropTypes.string.isRequired,   // a string, never an Error object
  code:        PropTypes.oneOf(ERROR_CODES),  // drives the copy — see below
  onRetry:     PropTypes.func,
  onLearnMore: PropTypes.func,
}
```

**Copy is keyed on `code`, not on HTTP status.** wokay ship an enumerated error model precisely so the app can be specific: *"denials carry a reason so the app can say your library's subscription expired rather than not available."* The map from `code` → variant + copy is owned by Akriti and lives beside the resolver (§5); this component receives the result.

| `code` | Variant | Copy is about |
|---|---|---|
| `NO_ENTITLEMENT` | `access_restricted` | Your library does not hold this title |
| `ENTITLEMENT_EXPIRED` | `access_restricted` | Your library's subscription has expired |
| `ENTITLEMENT_SUSPENDED` | `access_restricted` | Access is suspended — contact your library |
| `DOWNLOAD_NOT_PERMITTED` | `access_restricted` | This title is available online only |
| `CONTENT_NOT_READY` | `not_ready` | Still being prepared — **retry is meaningful here**, unlike the others |
| `FORBIDDEN_INSTITUTION_MISMATCH` | `access_restricted` | Signed in to a different institution |
| `NOT_FOUND` | `not_found` | Deliberately indistinguishable from unknown or archived |

**`NOT_FOUND` is deliberately ambiguous and the copy must not guess.** wokay return the same 404 for unknown, archived and not-entitled *specifically* so the catalogue cannot be mapped by walking ids. Copy that says "you don't have access to this" on a 404 leaks exactly what the design is protecting.

**A naming conflict to resolve.** The design package calls screen 14 *"error states"* (`14-error-states.png`); the delivery plan calls it *"Access Restricted"*. Both are true — screen 14 carries the generic error treatment **and** the D8 not-entitled state. Hence the third variant. Confirm with whoever owns the design package.

**Must not** accept an `Error`. Taking a `string` makes leaking a stack trace structurally impossible, which is what §4.2 requires.

#### 20 · `OfflineBanner` — Day 4 · Difficulty 2

**Screens:** global, 15

Persistent dark grey, pinned top. §4.2: the library and institution list **stay usable behind it** — so it overlays; it does not block interaction or push content down.

```js
OfflineBanner.propTypes = {
  visible: PropTypes.bool.isRequired,
  message: PropTypes.string,
}
```

**Network detection is Prayas's `useNetworkStatus` hook, not this component.** A native subscription is not presentation, and moving it out is what keeps this item on Khushi's list at all.

#### 21 · `FormatSelector` — Day 4 · Difficulty 3

**Screens:** 05

Formats come from the OPDS link-type MIME declaration on `item.formats` — **never a hardcoded list** (Design Spec §5, wokay Q13).

**States:** `single_format` (renders as a label, not a chooser) · `multiple` · `empty` (no formats declared — render nothing) · **`read_only`** (`canPersist: false` — formats shown, download suppressed)

```js
FormatSelector.propTypes = {
  formats: PropTypes.arrayOf(PropTypes.shape({
    mime:      PropTypes.string.isRequired,
    sizeBytes: PropTypes.number,
  })).isRequired,
  canPersist:   PropTypes.bool,      // false ⇒ no download affordance at all
  selectedMime: PropTypes.string,
  onSelect:     PropTypes.func.isRequired,
}
```

**`canPersist` is the input, not the tier.** It comes off the acquisition link. `false` means the Download affordance does not exist — and the server refuses the intent anyway with `DOWNLOAD_NOT_PERMITTED`, so a rendered-but-broken button is a visible bug rather than a harmless one. **Must not** read `item.accessTier` to work this out; that is the same rule as the resolver, and this component is the most likely place in the library to break it, because "Elite means no download" is easy to hardcode.

**Test against two fixtures.** The audiobook — `audio/*` only, no PDF, no EPUB; a selector assuming at least two document formats will break on it. And the Elite item — `canPersist: false`, where the format is still worth displaying but nothing may offer to save it.

---

### 6.9 Screen coverage check

Every team1 screen, and what builds it. No gaps.

| Screen | Components |
|---|---|
| 01 Catalogue home | TopAppBar, Tabs, SearchInput, Carousel+PageDots, SectionHeader, SubjectChip, ContentCard, AccessTierBadge, Skeleton, OfflineBanner, BottomTabBar |
| 02 Sign-in sheet | BottomSheet, InstitutionRow, ActionButton |
| 03 Access Gate | BottomSheet, ActionButton |
| 04 Item detail (article) | TopAppBar, Tabs, AccessTierBadge, ActionBar, ActionButton, Skeleton, ErrorState |
| 05 Item detail (book) | as 04, plus FormatSelector |
| 06 Institution list | TopAppBar, SearchInput, SectionHeader, InstitutionRow, EmptyState, Skeleton |
| 09 Search + filter | TopAppBar, SearchInput, Tabs, FilterChip, ContentCard, AccessTierBadge, EmptyState, Skeleton, BottomTabBar |
| 10 Profile | TopAppBar, InstitutionRow, ListRow, BottomTabBar |
| 11 Voice search | VoiceOverlay |
| 12 Filter / sort sheet | BottomSheet, FilterChip, ActionButton |
| 14 Error / Access Restricted | TopAppBar, ErrorState, ActionBar (`[]`) |
| 15 Offline | OfflineBanner |
| 16 Loading | Skeleton, plus every composite skeleton |
| 17 Empty search | EmptyState |
| 18 Access tiers | AccessTierBadge, ContentCard, ActionButton |

> **One possible gap, flagged rather than filled.** There is no **Toast / Snackbar** in the inventory, and Design Spec §4.2 lists only loading, empty, offline and error. But D13 (Borrow failure handling, Week 3) needs somewhere to surface "seat limit reached" or "already on loan" — and flambeau question 18 asks who renders those outcomes. If the answer is "team1," that is a 22nd component. Ask flambeau in the Day 3 conversation rather than discovering it in Week 3.

---

## 7. Khushi's block — leaf work, in detail

### 7.1 The shape of it and the reason

Six presentational components, the state gallery, and one read-only screen. Two changes from the Section 01 inventory:

| Change | From | To | Why |
|---|---|---|---|
| `BottomTabBar` | Khushi | Phase 0 shell (P0-6), Keshav + Khushi pairing | Navigation wiring, not presentation. Blocks all four other people. |
| `OfflineBanner` network detection | Khushi | Prayas (`useNetworkStatus` hook) | A native subscription, not presentation. Khushi keeps the banner as a pure component taking `visible: boolean`. |
| `C2` institution detail | Keshav | Khushi | A genuine leaf — nothing in the plan depends on it. Gives her something demoable that nobody is waiting on. |

**The governing rule for every item below: props in, callbacks out.** No `useEffect`, no `async`, no navigation calls, no adapter or store imports, no access logic. Every component is a pure function of its props.

This is not a training-wheels version of the real convention — it *is* the convention from P0-5, applied strictly. Presentational components that stay pure are the ones that survive the Week 3 state-matrix pass and the Week 4 adapter swap unchanged. The work is genuinely easier to write and genuinely more correct, which is why it does not need framing as an exception.

### 7.2 The items

**Prop contracts for all five components live in §6.8 and are not repeated here** — one definition, one location, same rule as the library itself. This section covers why each item is hers, what to watch, and how to sequence them.

**K1 · `EmptyState`** — Difficulty 2 · Screens 06, 09, 17

Two copy variants, and the distinction is a design requirement rather than a nicety: *no results for a query* ("No articles or books match…") reads differently from *no results for your filters* ("Try adjusting your filters…"), because only the second one has an obvious user action.

This one is written out in full as the reference pattern — every other component in the library follows the same two-part shape.

```tsx
// src/components/EmptyState/EmptyState.tsx

export interface EmptyStateProps {
  variant: 'no_query_results' | 'no_filter_results' | 'no_content' | 'browse_instead'
  /** Echoed back in the no_query_results copy. */
  query?: string
  /** Rendered only for no_filter_results. */
  onClearFilters?: () => void
  /** browse_instead only — follows the feed's own navigation entry. */
  onBrowse?: () => void
}

export function EmptyState({ variant, query, onClearFilters }: EmptyStateProps) { /* … */ }
```

One declaration instead of three. Note that `browse_instead` is in the union rather than in a comment — a variant the gallery must render and a screen may pass is a variant the type should name.

**K2 · `ErrorState`** — Difficulty 2 · Screens 04, 05, 14

Plain-text message, Retry and Learn-more actions. **No stack traces surfaced, ever** — the component takes a human message, not an `Error` object, which makes leaking one structurally impossible.

Three variants, including `access_restricted` for screen 14 — see §6.8, component 19.

**K3 · `OfflineBanner`** — Difficulty 2 · Global, screen 15

Persistent dark-grey banner, pinned top. Takes `visible` as a prop; the network subscription lives in Prayas's hook and is passed down. The Library and institution list must stay usable behind it, so the banner overlays rather than blocking interaction. Prop contract in §6.8, component 20.

**K4 · `FormatSelector`** — Difficulty 3 · Screen 05

Renders the formats an item declares. Formats come from the OPDS link-type MIME declaration on `ContentItem.formats`, **never from a hardcoded list** — an audiobook fixture with only `audio/*` must render correctly with no PDF or EPUB option present.

Prop contract in §6.8, component 21.

> **`ProgressBar` was removed from her list.** It appears only on screens 07 and 08, both t4targaryen's — see §6.1. If t4targaryen answer their question 4 with "team1 owns the download indication," it returns here, at roughly two hours.

**K5 · State gallery route** — Difficulty 4 · The most valuable thing she builds

One scrolling screen rendering **every** component in **every** variant: default, loading, empty, error, offline, and all three access tiers. Grouped by component, labelled, with a light/dark toggle if time allows.

This is the review surface for Day 5, the regression surface for Week 3's state-matrix pass, and the fastest way for a newer React Native developer to see the whole library at once. It is also the thing that makes the five-way split safe — drift between five authors is invisible in five files and obvious on one screen.

**K6 · `C2` Institution detail** — Difficulty 4 · Screen 06 detail

Read-only screen: crest, name, country, sign-in type, and a description block. Built as a **pure presentational view**:

```tsx
// screens/institution/InstitutionDetailView.tsx  — Khushi, pure
import type { Institution } from '@model/types'

export interface InstitutionDetailViewProps {
  institution: Institution
  onSelect: () => void
  onBack: () => void
}
```

**The whole hand-written shape collapses into one imported type.** Under the JavaScript plan this screen restated eleven of `Institution`'s fields as a `PropTypes.shape`, which is eleven chances to drift from the contract — and `logoUrl` had already been renamed once. Now `Institution` moves and this file follows or fails to build.

**"Sign-in type" is no longer a field on this screen.** It was going to render `authType`; institutional sign-in is always SAML, so there is nothing to display and nothing to choose. The screen becomes logo, name, type, country/city and a description block. `AUTH_TYPES` is deleted — if this file still imports it, it will not resolve.

The remaining shapes are imported from `model/types.ts` rather than retyped — that import is the thing keeping the component and the contract in step and the compiler now enforces it rather than trusting it.

The route wrapper that reads `institutionId` from route params, calls `getInstitution`, and renders the view is **one file, ~15 lines, written by Keshav in P0-6** as part of the shell wiring. Khushi builds against the fixture directly through the gallery, with no navigation and no async involved.

This is worth doing deliberately rather than out of convenience: it is the same container/presentation split the whole plan depends on, and it means C2 can be developed and reviewed entirely inside the gallery.

### 7.3 Khushi's dependencies — deliberately, almost none

| Needs | From | When | If it slips |
|---|---|---|---|
| Design tokens | Herself, Day 1 | Day 1 | n/a |
| Component convention | P0-5, all five | Day 1 | n/a |
| `Institution` type + 8 fixtures | P0-3, P0-4 | Day 2 | Hand-write one institution object locally for K6 |
| Running app shell | P0-6 | Day 2 | Build in the gallery only; the route wrapper is Keshav's |
| Empty-state copy variants | Moktik, Week 3 | Week 3 | Not a Week 1 dependency |

**Nobody depends on any K-item.** Every one can slip to Week 2 without stopping another person. That is the property being engineered for, and it is worth stating out loud in the standup so it does not get quietly undone when something else runs late.

### 7.4 Suggested ordering

| Day | Work |
|---|---|
| Wed AM | K5 gallery route scaffold — build it empty first, then add each component as it lands. Building the review surface first makes everything after it self-checking. |
| Wed PM | K1 `EmptyState`, K2 `ErrorState` — the two simplest; get the convention into muscle memory before anything with logic. |
| Thu AM | K3 `OfflineBanner` |
| Thu PM | K4 `FormatSelector` — the first with real prop-driven branching — then the gallery completion pass |
| Fri AM | K6 `InstitutionDetailView` against the fixture |
| Fri PM | Final gallery sweep, then chair the consistency review (§8.2) |

---

## 8. Working agreements

### 8.1 Merge protocol

- **Daily merge to `main`. No branch lives longer than one day.** Long branches plus a shared component library is the worst combination available to this team.
- Component PRs need one review from any other author. Changes to `model/`, `adapters/`, `access/` or `theme/` need the named owner.
- A component merges **before** the screen that consumes it.

### 8.2 The Day 5 consistency review — Friday 16:00, 45 minutes

**Chair: Khushi** (she owns the gallery). All five attend.

All twenty-one components are looked at side by side in the gallery. Checking for: divergent spacing, off-token colours, inconsistent prop naming, missing state variants, and two components that should be one.

**One item drops off this review and one joins it.** Missing prop declarations are no longer a review concern — the build catches them. What the reviewer should look for instead is **`any` in a props interface, or a union retyped as literals instead of imported from `types.ts`**: both compile cleanly and both are how a component quietly drifts from the contract.

**This review is the single thing that pays for splitting the library five ways.** Skip it and the drift will not surface until Week 3, by which point every screen has consumed the drifted components. Anything found is fixed before Monday, not logged.

### 8.3 Friday handover — 17:00

Dependency board to next week's lead. Every item in exactly one state: **Asked · Answered · Stubbed · Integrated**. The board moves with the rota, never owned by one person for longer than a week.

---

## 9. Risks

| # | Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|---|
| R1 | Phase 0 overruns into Day 3 | High — greenfield always does | Compresses feature work to two days | Timebox the scaffold to Day 1 morning. Take the recommended stack defaults; do not debate them. If Day 2 evening is not green, cut the core five to three (ContentCard, TopAppBar, Skeleton) rather than cutting types or fixtures. |
| R2 | Prayas's normalisation model is wrong | **Closed, down from Medium** | — | **The samples landed.** Three frozen OPDS fixtures are in `wokay_docs/frozen/`, and §P0-3 has been reconciled against them — six corrections, listed in §P0-4. He is building to their shapes, not guessing at them. Keep the Day 3 pairing hour anyway: its purpose is now walking their nesting against ours, which is where the remaining mistakes live. |
| R3 | ~~Q-D (`accessTier`) unanswered~~ → **the work-type field is unanswered** | Medium | Screens 04 and 05 differ by work type, so both run on derived data. Not a blocker — nothing is unbuildable, unlike the old R3 | Q-D is **answered**: `accessTier` exists and is a free filter. The replacement risk is smaller: derive work type in **one** function so a real field is a one-file swap, and send the ask Day 1. |
| R3b | **`resolveAccess` was specified against an inverted access matrix** | **Occurred** | Would have been a Week 2–3 resolver rewrite touching every screen, plus a wrong demo | **Found on Day 0.** Design Spec corrected to v0.3, signature corrected in §P0-3, escalated with L-2. The residual risk is that the correction is not ratified and someone reverts to the v0.2 table — which is why the v0.3 banner is in the signed document rather than only in this one. |
| R3c | **Catalogue search lands Week 4, and team1's Week 4 is integration + BDD** | Medium–High | Moktik builds against fixtures for three weeks and integrates in the busiest week of the plan | Name it on the dependency board in Week 1. Keep fetch-and-filter strictly behind the pipeline interface so the swap is a configuration change. See §5, Moktik. |
| R4 | Five authors, five styles | Medium | Visible inconsistency at demo | Tokens Day 1, one skeleton, Day 5 review. All three are required — any two is not enough. |
| R5 | Feature work introduces components outside the library | High under deadline | Two Cards by Week 3 | The P0-5 rule, enforced at review. The most likely rule to break quietly. |
| R6 | Khushi blocked despite the re-cut | Low | Minimal — nothing depends on her | She can build every K-item against the gallery with no running app. Escalate same day if she is idle. |
| R7 | L-2 and the §3.1 correction not ratified | Medium | Building against a corrected spec with no written cover — and, worse, two contradictory access matrices in circulation | Escalated Day 1, **bundled**: they are one conversation with leadership and the §3.1 half is the more urgent. The letter is stronger than it was, because wokay's contract *cannot serve* the v0.1 model rather than merely differing from it. If no answer by Friday, both go to next week's lead as open escalations. |
| R10 | **My Library ownership unresolved** | Medium | If wokay's §03 is right, `ProgressBar` returns and a whole screen joins the block, re-cutting Khushi's list | Named, owned and dated — resolve by Friday, Week 1 (§6.1). `getItemsBatch` is added to the adapter regardless, since it is the right data-layer shape either way. |
| R8 | Contract drift, undetected | **Low, down from Medium–High** | A shape mismatch between adapter and screen surfacing in Week 4 integration | **Largely closed by the move to TypeScript (11 Aug).** `implements DataAdapter` on both adapters, one contract file, and `npm run typecheck` as a CI gate. What the compiler still cannot see is runtime data, so `validate.ts` over fixtures and real responses, plus the behavioural half of the conformance suite, remain required. |
| R9 | ~~PropTypes and JSDoc quietly skipped~~ → **`any` used as a silencer rather than an escape hatch** | Medium | Types present but meaningless, which reads as safety without being it | `any` is deliberately allowed with no review pushback in Week 1, so the risk is that it becomes the default rather than the exception. Cheapest control: `any` in `model/`, `adapters/` or `access/` needs a one-line reason in the PR. Elsewhere it is nobody's business. |

---

## 10. Week 1 exit criteria

From the delivery plan, plus what this document adds:

- [ ] Catalogue browsable and searchable on mock data
- [ ] Institution list, detail and search working
- [ ] Component library — all 21 merged (§6), each with a gallery entry and a typed props interface
- [ ] State gallery route complete and reviewed
- [ ] Consistency review held, findings fixed before Monday
- [ ] Contracts sent in writing to all four audiences, reconciled against wokay's published schema — **not** re-asking what their document already answers
- [ ] **wokay's four Week 1 asks answered**, subject facets among them
- [ ] **The work-type and DOI asks sent** — the two fields their model genuinely lacks
- [ ] L-2 escalated **bundled with the §3.1 access-matrix correction**; sign-in handoff agreed with flambeau (SAML + `idpHint`)
- [ ] **My Library ownership resolved, or recorded as an open escalation with a name against it**
- [ ] Dependency board handed over, every item in exactly one state

---

## Appendix A — Component ownership, reconciled

21 components. Bold = Phase 0 (Day 2). Full design specs in §6. Screens listed are **team1's only** — 07, 08 and 13 are excluded per §6.1.

| # | Component | Author | Day | Screens | Diff |
|---|---|---|---|---|---|
| 1 | **ContentCard** | Prayas | 2 | 01, 04, 05, 09, 17, 18 | 6 |
| 2 | SectionHeader | Prayas | 3 | 01, 06, 09 | 2 |
| 3 | SubjectChip | Prayas | 3 | 01, 09 | 2 |
| 4 | Carousel + PageDots | Prayas | 3 | 01 | 5 |
| 5 | Tabs | Prayas | 3 | 01, 04, 09 | 4 |
| 6 | **SearchInput** | Moktik | 2 | 01, 06, 09 | 3 |
| 7 | FilterChip | Moktik | 3 | 09, 12 | 2 |
| 8 | VoiceOverlay | Moktik | 3 | 11 | 6 |
| 9 | **TopAppBar** | Keshav | 2 | every screen | 3 |
| 10 | **BottomTabBar** | *P0-6 shell* — Keshav + Khushi | 2 | 01, 09, 10 | 4 |
| 11 | BottomSheet | Keshav | 3 | 02, 03, 12 | 5 |
| 12 | InstitutionRow | Keshav | 3 | 06, 10 | 2 |
| 13 | ListRow | Keshav | 3 | 10 | 1 |
| 14 | **AccessTierBadge** | Akriti | 2 | 01, 04, 05, 09, 18 | 2 |
| 15 | ActionButton | Akriti | 3 | every screen | 5 |
| 16 | ActionBar | Akriti | 3 | 04, 05 | 4 |
| 17 | **Skeleton** | Khushi | 2 | 01, 04, 05, 06, 09, 15, 16 | 3 |
| 18 | EmptyState | Khushi | 3 | 06, 09, 17 | 2 |
| 19 | ErrorState | Khushi | 3 | 04, 05, 14 | 2 |
| 20 | OfflineBanner | Khushi | 4 | global, 15 | 2 |
| 21 | FormatSelector | Khushi | 4 | 05 | 3 |
| — | ~~ProgressBar~~ | — | — | *07, 08 — t4targaryen only. Removed, §6.1* | — |

**Totals:** Prayas 5 · Keshav 4 · Khushi 5 · Moktik 3 · Akriti 3 · shell 1. **19 of 21 merged by end of Day 3.**

Design tokens are deliberately absent from this table. They are written by one person on Day 1, before any component exists, so that no component author ever picks a colour or a spacing value. That single exception is what makes splitting the other twenty-one safe.

---

## Appendix B — Open items this document assumes

Each has a stub committed, so none of them stops work.

#### B.1 — Closed by wokay's source of truth

Do not send these. Sending answered questions spends credibility on the ones that still matter.

| Ref | Question | Answer |
|---|---|---|
| **Q-D / W-4 / D4** | Will wokay add a per-item access-state field? | **Closed 11 Aug — there is no field, and there will not be one.** The `catalogueItems.accessTier` shown in wokay's source-of-truth document does not exist; **that document is outdated by wokay's own statement, and the frozen samples are the contract.** The tier is derived from the acquisition link's `licenceModel`: `UNLIMITED` → Subscription, `CONCURRENT` → Elite, key absent → Open Access. wokay supplied the mapping, so it is a rule rather than a guess. Derived once in the adapter, so components still receive a plain field. |
| **W-1 / Q-B** | Unauthenticated feed? | **Yes, but narrower.** `GET /opds/v1/public/catalogue`, no token — **open access only**, not the full catalogue. Ready Wk 4. |
| **Q-E / W-5** | Search link template, or fetch-and-filter? | Templated search link, **server-side**, entitlement-scoped, metadata-only. Wk 4. Fetch-and-filter is a fixture stand-in only. |
| **W-6** | Pagination — offset, cursor, or next links? | *"Follow the next link until it is absent. Do not count pages."* OPDS is next-link only; `page`/`size`/`total` exists on the institutions endpoint alone. |
| **W-8** | Stable item identifier? | Yes — prefixed strings, `item_42`. |
| **W-9 / Q-G** | URN prefixes for DOI and ISBN? | Moot. `isbn` is a **top-level field**; no URN parsing. **No DOI anywhere** — reopened as a new ask. |
| **W-15** | Institution sign-in-type field? | **Dead.** *"Institutional sign-in is ALWAYS SAML, so there is no `authMethod` field."* `signIn.idpHint` is what we pass. |
| **W-17** | Institution crest / logo URLs? | `logoUrl`, plus `branding.logoUrl` and `branding.primaryColor`. |
| **W-3** | Institution schema? | Published in full — see the `Institution` typedef in §P0-3. |
| **A4** | Dual pagination behind one interface? | **Shrinks.** Two models split cleanly by endpoint family; no unifying abstraction needed. |
| **Q-V** | Does signing in make Open Access harder to reach? | **Closed.** An artefact of the incorrect loan rule. Open Access resolves identically in every session state. |
| **L-3** | Action vocabulary ratification | `borrow` confirmed, Buy stays removed — but **`subscribe` is retained** (B2C not cut, 11 Aug). Six actions. The Subscribe *flow* is unspecified pending wokay's details. |
| — | Do we own a backend module? | **No.** *"team1 and t4targaryen have NO backend module. React Native only."* |

#### B.2 — Still open

| Ref | Question | Stub in place |
|---|---|---|
| **work type** | ✅ **`@type` is official** (confirmed 11 Aug). `schema.org/Book` and `schema.org/Audiobook` are confirmed values | Mapped in `SCHEMA_TYPE_TO_WORK_TYPE`. **Only the journal and article values remain open** — wokay will supply. Two guesses are in the map, both marked; an unmapped `@type` falls back rather than throws. Affects which detail screen renders, which is Week 2 work |
| **formats** | ✅ **One work, one format** (confirmed 11 Aug). So `FormatSelector` — Khushi, Day 4, screen 05 — has nothing to select between | **This is now a decision, not a question.** Either drop the component (library goes to 20) or keep it as a single-format display strip. Recommend dropping it and reclaiming Khushi's Thursday; raise it at Wednesday standup, before she starts |
| **incidental fields** | ✅ **Contractual.** The samples are the current contract and the written document is outdated, so `subtitle`, `editor`, `narrator`, `duration` and `numberOfPages` all stand. A publication may have editors and **no author** | Modelled in `ContentItem`; `ContentCard` falls back to `editors` before rendering a blank byline |
| **DOI** | ✅ **Closed — there is no DOI and there will not be one.** Only `identifier` exists | **Drop DOI from screen 04.** A design change rather than a pending ask; it needs to reach whoever builds article detail (Khushi, Week 2) |
| **missing samples** | ✅ Coming. The zero-result feed, the error envelope and the institutions JSON will follow | Hand-write against §P0-3 meanwhile. Chase the institutions one — Keshav's whole block, and those endpoints land Week 2 |
| **NEW — tier as a filter (Q-12)** | The filter chips were specified on the basis that access tier is *"free as a filter"*. If there is no tier field, is `?accessTier=` still a valid parameter, mapped to `licenceModel` server-side — or are the chips content-type only? | **Ask wokay today.** Screen 12's filter sheet and `FilterChip` both depend on it, and Moktik builds them Day 3 |
| **NEW — DOI** | No DOI field exists anywhere in wokay's schema | Omit from screen 04 until answered |
| **NEW — My Library** | wokay §03 says team1 owns it; §6.1 says t4targaryen | 21 components, screen 08 excluded; escalated, resolve by Friday |
| **NEW — subject facets** | Ours to request; **their silence-default is not to build them** | Answer *yes* on Day 1 (§P0-8) |
| W-11 / Q-H | Sort — facet group, or a query parameter? | Sort locally over the fetched page. `sort=publishedAt,desc` is hinted on `groups` but not confirmed as the contract |
| W-13 | Is item metadata complete in the feed? | Render what exists, leave gaps blank |
| FL-5 | Does SAML leave the app, and how does control return? | Pending intent persisted to disk. Narrower now — SAML is the only path |
| FL-9 / Q-U | Bulk loan-state lookup? | Per-item behind a cache, interface shaped for bulk |
| L-2 | Ratification of the feed-scoping reversal of Design Spec §4.1 | **Character changed** — wokay *cannot serve* the v0.1 model, so this is a statement of fact needing ratification, not a request. Bundle with the §3.1 correction |
| L-5 | Ratification of shelves-as-tabs | Build tabs from a config list — now mandatory, since the shelf set is per-institution configuration |
| — | Per-institution theming from `branding.primaryColor` | **Decided by team1: no.** One fixed palette; the field is carried and unused — §P0-2 |
| — | Spacing, pill radius and card shadow, inferred not specified | §P0-2 values; flagged as inferred |

---

## Appendix C — Traceability against the Design Specification

Audited against `TF_Reader_Design_Package/docs/TF_Reader_Design_Specification.md` **v0.3** — whose §3.1 and §5.2 were corrected against wokay's contract and are **not yet ratified**. Everything traced to §3.1 below therefore rests on an unratified correction; that is deliberate and it is escalated (§P0-8), but a reviewer should know it.

### C.1 Covered and matching

| Design Spec | Requirement | Where it lands |
|---|---|---|
| §2.1 | 11 colours with hex codes | P0-2 `color` — all 11, verbatim |
| §2.2 | 6 type styles, weight / size / line-height | P0-2 `type` — all 6, verbatim |
| §2.2 | Inter font family | P0-2 `font` + P0-1 scaffold row |
| §2.3 | Cards: 8px radius, box shadow, thumbnail-left layout | P0-2 `radius.card`, `elevation.card`; ContentCard (P0-7) |
| §2.3 | Bottom sheets: 16px top radius, drag handle, 60–70% | P0-2 `radius.sheet`; BottomSheet, Days 3–5 (Keshav) |
| §2.3 | Badges: pill-shaped, three access tiers | P0-2 `radius.pill`; AccessTierBadge (P0-7) |
| §2.3, §4.2 | Skeleton loaders replace spinners; dimensions match final content | Skeleton (P0-7) |
| §3.1 | `resolveAccess(item, session, loan, availability)` signature | P0-3 — four arguments; `session`, `loan` and `availability` all nullable |
| §3.1 | Actions derive from the acquisition link, never from the tier | P0-3 derivation table; enforced by the `ContentCard`, `ActionBar` and `FormatSelector` "must not" clauses |
| §3.1 | `tier` — 3 values, **uppercase** | P0-3 `ACCESS_TIERS` — matches `catalogueItems.accessTier` |
| §3.1 | `state` — 5 values | P0-3 `AccessState` — exact match |
| §3.1 | `actions` — 6 values plus empty | P0-3 `ActionId` — `subscribe` **retained**, B2C not cut |
| §3.1 | Loan is per-user, per-item, mutable, from flambeau | P0-3 `Loan`; never a property of `ContentItem` |
| §3.1 | `no_seats` is Elite-only and detail-screen-only | P0-3 `Availability` as the 4th resolver arg; separate `Availability` fixture (P0-4); excluded from `ContentCard` (§6.3) |
| §5.1 | Errors typed, copy keyed on `code` | P0-3 `ApiError` + `ERROR_CODES`; the code→copy map in §6.8 component 19 |
| §5.1 | Follow hrefs, never build URLs; paginate by `next` | P0-3 `DataAdapter` — every catalogue method takes an href, not an id |
| §5.1 | Zero results is a navigation feed, not an empty array | `Page.browseInstead`; `EmptyState` `browse_instead` variant |
| §5.4 | Work type and DOI are unbacked | P0-3 `WorkType` marked unbacked; both are Day 1 asks (§P0-8) |
| §3.2 | Pending-Intent Store survives the auth round trip | F6, Akriti, Days 3–5 — **persisted to disk**, per FL-5 |
| §4 | Four tabs: Catalogue · Search · Library · Profile | P0-6 |
| §4.2 | Two distinct empty-state copy variants | K1 `EmptyState` |
| §4.2 | Persistent dark grey offline banner; library stays usable | K3 `OfflineBanner` |
| §4.2 | Plain-text errors, Retry / Learn more, no stack traces | K2 `ErrorState` — takes a `string`, not an `Error` |
| §5.1 | UI must never calculate access rights | `access/*` ownership rule; ContentCard prop constraint (P0-7) |
| §5.2 | Library and guest downloads must not assume a `userId` | `resolveAccess(item, session: Session \| null, …)` |
| §5.3 | Swappable Mock / Api adapters over one normalised shape | `DataAdapter` (P0-3), `MockAdapter` (P0-4) |

### C.2 In the design specification, deliberately not in Week 1

Not gaps — out of scope for the foundation, listed so nobody assumes they were missed.

| Design Spec | Item | When |
|---|---|---|
| §4.1 | Screen 01 featured carousel and Browse-by-Subject | Week 2 (Prayas — A1, A3) |
| §4.1 | Screen 03 Access Gate sheet | Week 2 (Akriti) |
| §4.1 | Screens 04 / 05 item detail and the dynamic action bar | Week 2 (Khushi — E1–E4) |
| §4.1 | Screen 08 Library, downloads, Guest Downloads | t4targaryen (CAP-7), not team1 |
| §4.2 | "Offline Library" card on the catalogue | Week 3 (F5 states pass) |
| §6 | 18 screen mockups walked for the full state matrix | Week 3, Phase 3 |

### C.3 Where the design specification is silent

Each has a value chosen in this document, flagged as ours rather than design's:

- **No spacing scale** — `space` in P0-2 is inferred from the mockups.
- **No pill radius value** — §2.3 says "pill-shaped" only.
- **No shadow values** — §2.3 says "subtle box shadow" only; React Native needs platform-split values.
- **No dark mode** — §2.1 mentions navy for "dark mode overlays" but no dark palette exists. Week 1 is light only; raise it if dark mode is expected at demo.
- **No institution model** — OPDS does not model institutions and neither does the design specification. The `Institution` shape in P0-3 is entirely ours and is the one being sent to wokay.
