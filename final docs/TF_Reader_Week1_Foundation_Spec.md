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
- **JavaScript, not TypeScript.** A team decision. It removes the compiler that the delivery plan's contract-first approach was resting on, so three replacements are specified in its place — see §1.5.
- **Khushi is newer to React Native.** Her block is re-cut to props-only presentational components — no navigation, no async, no effects, no adapter access. §7 sets out the rule and why it is also good architecture rather than a special case.

### 1.5 Building in JavaScript — what has to replace the compiler

The delivery plan's central mechanism is *contracts as types*: one file defines `ContentItem`, `Institution`, `Session`, `DataAdapter` and `resolveAccess`, four people code against it, and the same file is attached to the Section 09 question sets so that wokay's and flambeau's silence safely means *they accepted our shape*.

Without TypeScript, nothing mechanically enforces that. Three specific things go missing, and each gets a replacement:

| What the compiler was doing | Replacement | Where |
|---|---|---|
| Guaranteeing the whole team builds against one shape | **JSDoc `@typedef` in `src/model/types.js`** — still one file, still the single contract, still the Section 09 attachment. With a `jsconfig.json` setting `checkJs`, VS Code type-checks the JSDoc and flags mismatches inline, with no build step and no `.ts` files. | P0-3 |
| Catching a malformed fixture at compile time | **Fixture validation on load** — `MockAdapter` asserts every fixture against the contract in dev and throws loudly on a mismatch. | P0-4 |
| Guaranteeing `MockAdapter` and `ApiAdapter` stay interchangeable | **An adapter conformance test** both must pass. This is what makes the Week 4 "integration is a configuration change" claim survive. | P0-3 |

Plus **PropTypes on every component**, which gives runtime warnings in development for the prop contracts that interfaces were carrying.

None of this is as strong as a compiler, and it is worth being honest that the class of bug it stops catching — a shape mismatch between the adapter and a screen — is exactly the one Week 4 integration is most exposed to. The mitigation is that the contract still lives in exactly one file, so a change is still a one-file edit rather than a fifteen-file hunt.

**`jsconfig.json`, committed in P0-1:**

```json
{
  "compilerOptions": { "checkJs": true, "target": "esnext", "moduleResolution": "bundler" },
  "include": ["src/**/*"]
}
```

Editor-only. It fails nothing at build time and blocks nobody, but everyone sees a red squiggle when they pass the wrong shape.

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
| Language | **JavaScript** — team decision | See §1.5 for what replaces the compiler. |
| Contract checking | **`jsconfig.json` with `checkJs`** + JSDoc typedefs | Editor-level checking of the one file that matters. Zero build cost, blocks nobody. |
| Prop contracts | **`prop-types`** on every component | Runtime warnings in dev. Especially useful for anyone newer to React Native — the feedback arrives on screen, not in a compiler. |
| Font | **`expo-font` + `@expo-google-fonts/inter`**, loaded at app start | Design Spec §2.2 names Inter. Retrofitting it after the library exists shifts every line height at once. |
| Navigation | **React Navigation** — `bottom-tabs` + `native-stack` | Four tabs and pushed detail screens is exactly its shape. Better documented than the alternatives, which matters with a mixed-experience team. |
| State | **Zustand** for session, selection and pending intent | Three small stores. Redux is more ceremony than this needs. |
| Persistence | **AsyncStorage** behind a `Storage` interface we own | Institution selection and pending intent must survive an app restart (see FL-5). Keep it behind an interface so swapping to MMKV is one file. |
| Testing | **Jest + React Native Testing Library** | Needed for the Week 4 BDD suite; cheaper to add on Day 1 than Week 4. |
| Quality | **ESLint + Prettier, pre-commit hook** | Five authors, one branch, daily merges. Non-negotiable. |

**Folder structure** — matches the file-ownership table in the delivery plan, so ownership is visible in the tree:

```
src/
  theme/          tokens.js                        [Khushi — single author]
  model/          types.js, fixtures/, validate.js [Prayas — Akriti 2nd reviewer]
  adapters/       MockAdapter.js, conformance.test.js  [Prayas]
  access/         resolveAccess.js, session.js     [Akriti — only place access logic may live]
  components/     one folder per component         [split five ways]
  screens/        catalogue/ search/ institution/ detail/ library/ profile/
  search/         shell/ catalogue/ institution/   [Moktik — Keshav 2nd reviewer]
  storage/        pendingIntent.js, selection.js
  gallery/        StateGallery.jsx                 [Khushi]
  navigation/     RootNavigator.jsx, tabs.js
```

**Done when:** `main` contains a running Expo app; all five people have cloned it, installed, and launched it on their own machine; lint and a trivial test pass in CI or via a documented command. **Every one of the five confirms this before anyone goes further** — a scaffold that only runs on the author's laptop is not a scaffold.

---

### P0-2 · Design tokens

**Owner:** Khushi — **single author, never split**
**Duration:** ~2 hours, Day 1, immediately after the scaffold runs
**Blocks:** every component

Transcribe §2.1, §2.2 and the measurable parts of §2.3 of `TF_Reader_Design_Package/docs/TF_Reader_Design_Specification.md` **verbatim** into typed constants. Transcription, not interpretation — do not add a colour, do not adjust a size, do not invent an intermediate grey.

```ts
// src/theme/tokens.js
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

```js
// src/model/types.js
// The single contract. Four people code against this file, and this file is
// what goes out with the Section 09 question sets. JSDoc, checked by jsconfig.

/** @typedef {'open_access'|'subscription'|'elite'} AccessTier */
/** @typedef {'available'|'requires_loan'|'requires_signin'|'not_entitled'|'no_seats'} AccessState */
/** @typedef {'read'|'download'|'borrow'|'signin'|'subscribe'|'waitlist'} ActionId */
/** @typedef {'book'|'journal'|'article'|'audiobook'} ContentType */
/** @typedef {string} FeedId — wokay's three endpoints; names TBC (wokay Q1) */

/**
 * @typedef {Object} ContentItem
 * @property {string}      id             stable across sessions — wokay Q14
 * @property {FeedId}      feedId
 * @property {ContentType} type
 * @property {string}      title
 * @property {string[]}    authors
 * @property {string}     [publisher]
 * @property {number}     [publishedYear]
 * @property {string[]}    subjects
 * @property {string}     [coverUrl]
 * @property {string}     [abstract]
 * @property {{doi?: string, isbn?: string, raw: string[]}} identifiers
 *           doi and isbn extracted from URNs — wokay Q9. raw is
 *           identifier + altIdentifier, untouched.
 * @property {Array<{mime: string, url?: string, sizeBytes?: number}>} formats
 *           from the link type MIME — wokay Q13. Never a hardcoded list.
 * @property {AccessTier}  accessTier     ⚠ THE Q-D ASK — no OPDS equivalent
 * @property {{total: number, available: number}} [copies]
 */

/**
 * OPDS models nothing here — this shape is entirely ours.
 * @typedef {Object} Institution
 * @property {string} id
 * @property {string} name
 * @property {string} country
 * @property {string} [crestUrl]                          wokay Q19
 * @property {'saml'|'oidc'|'email'|'unknown'} authType    flambeau Q1
 */

/**
 * Per the PRD — flambeau Q9 confirms.
 * @typedef {Object} Session
 * @property {string}   userId
 * @property {'b2b'|'b2c'} type
 * @property {string}  [institutionId]
 * @property {string[]} roles
 * @property {string[]} collections
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
 * @typedef {Object} AccessResult
 * @property {AccessTier}  tier
 * @property {AccessState} state
 * @property {ActionId[]}  actions
 */

/**
 * @callback ResolveAccess
 * @param {ContentItem}  item
 * @param {Session|null} session   null = logged out. Design Spec §5.2.
 * @param {Loan|null}    loan
 * @returns {AccessResult}
 */

/**
 * @template T
 * @typedef {Object} Page
 * @property {T[]}     items
 * @property {string} [nextCursor]
 * @property {number} [total]
 */

/**
 * @typedef {Object} DataAdapter
 * @property {() => Promise<Array<{id: FeedId, label: string}>>} getFeeds
 * @property {(feedId: FeedId, cursor?: string) => Promise<Page<ContentItem>>} getCatalogue
 * @property {(feedId: FeedId, q: string, cursor?: string) => Promise<Page<ContentItem>>} searchCatalogue
 * @property {(id: string) => Promise<ContentItem>} getItem
 * @property {() => Promise<Institution[]>} getInstitutions
 * @property {(id: string) => Promise<Institution>} getInstitution
 */

// Runtime unions. In TypeScript these were types only; in JavaScript they must
// exist as values, because validation, PropTypes oneOf() and the state gallery
// all enumerate them. One definition, three consumers.
export const ACCESS_TIERS  = ['open_access', 'subscription', 'elite']
export const ACCESS_STATES = ['available', 'requires_loan', 'requires_signin',
                              'not_entitled', 'no_seats']
export const ACTION_IDS    = ['read', 'download', 'borrow',
                              'signin', 'subscribe', 'waitlist']
export const CONTENT_TYPES = ['book', 'journal', 'article', 'audiobook']
export const AUTH_TYPES    = ['saml', 'oidc', 'email', 'unknown']
```

**`accessTier` is the single most important line in this file.** It is the field OPDS 2.0 does not define and wokay must add — register item Q-D / D4, and question 7 on wokay's list. Everything downstream of it (badges, filters, the resolver, the action bar) is unbuildable without it, and it has the longest lead time of anything team1 is asking for. Send it Day 1 with the shape already written, and ask wokay to correct it rather than design it.

**Exporting the unions as arrays is not a stylistic choice.** It is the JavaScript replacement for a union type: `ACCESS_TIERS` is what `validate.js` checks against, what `PropTypes.oneOf()` consumes, and what the state gallery iterates to render every tier variant. Three consumers, one definition — which is the property TypeScript was providing for free.

### The adapter conformance test

**Owner:** Prayas, same day

Without an `interface DataAdapter`, nothing stops `MockAdapter` and the later `ApiAdapter` from drifting apart — and "integration is a configuration change" in Week 4 depends entirely on them not drifting.

Write one test suite, exported, that takes any adapter and asserts it: implements all six methods; returns a `Page` shape with an `items` array from both list methods; returns objects passing `validate.js` from `getItem` and `getInstitution`; and rejects an unknown id rather than returning `undefined`.

`MockAdapter` passes it in Week 1. `ApiAdapter` must pass the identical suite in Week 3 before it is wired up. That test is the contract, now that the type is gone.

**Done when:** merged; `checkJs` reports clean in the editor; the conformance suite passes against `MockAdapter`; and the typedef block is pasted into the four Section 09 dispatches.

> **On what to send wokay and flambeau.** JSDoc is readable enough to send as-is. If it reads awkwardly to a Spring Boot team, paste the same shapes in TypeScript-style notation in the question set instead — the dispatch is documentation, not code, and the shape is what matters. What must not happen is sending prose descriptions of the fields; the whole point of contract-first is that they correct a concrete artefact.

---

### P0-4 · Mock fixtures

**Owner:** Prayas
**Duration:** Day 1 afternoon into Day 2
**Blocks:** everyone on Days 3–5

Roughly **40 content items** and **8 institutions**, hand-written, spanning all four content types across all three access tiers. Coverage matters more than volume, and the awkward cases matter more than the clean ones — a fixture set of forty tidy books will pass every test and fail the demo.

Mandatory awkward cases:

| Case | Exercises |
|---|---|
| Missing `coverUrl` | ContentCard placeholder path |
| No DOI and no ISBN | Identifier extraction and detail rendering |
| 180-character title | Truncation across card, detail and search result |
| Elite item with `copies.available = 0` | `no_seats` — reachable only on loan-requiring tiers |
| Same work in two feeds, same `id` | Cross-feed dedup — wokay Q2 |
| Audiobook, no ISBN, `audio/*` only | FormatSelector with a single non-document format |
| Article with no abstract | Detail screen with an empty body region |
| Institution with no crest | InstitutionRow initials fallback — W-17 |
| Institution with `authType: 'unknown'` | Sign-in routing fallback — CAP-3 |
| Subscription item, not in any fixture entitlement | `not_entitled` / screen 14 via pending intent |

**Fixture validation — `src/model/validate.js`.** With no compiler, a fixture missing `accessTier` or carrying `type: 'audio'` instead of `'audiobook'` will sail through and surface as a blank badge on someone else's screen three days later. Write a small validator — no library needed, ~40 lines — that checks required fields are present and that every union-typed field is a member of the corresponding exported array from `types.js`. `MockAdapter` runs it over the whole fixture set on first load in development and **throws loudly**, naming the item id and the offending field.

This is the highest-value 40 lines in the JavaScript version of this plan. It converts the entire fixture set into a contract test that runs every time anyone starts the app.

**Done when:** merged; `MockAdapter` serves every `DataAdapter` method with realistic latency (150–400ms) so loading states are visible during development rather than discovered in Week 3; and `validate.js` passes over all 40 items and 8 institutions.

---

### P0-5 · Component conventions and the gallery route

**Owner:** Khushi authors the convention document and the gallery route; agreed by all five
**Duration:** Day 1, ~1 hour of discussion, written up same day
**Blocks:** all 21 components

Five authors produce five styles unless something prevents it. Three things do: tokens written first (P0-2), one agreed skeleton (this item), and the Day 5 consistency review (§8.2).

**The convention:**

```
src/components/ContentCard/
  ContentCard.jsx          // the component + its propTypes block
  ContentCard.gallery.jsx  // every variant, rendered — this is not optional
  index.js                 // re-export
```

Props are declared two ways, and both are required: a **JSDoc `@typedef` above the component** so the editor checks call sites, and a **`propTypes` block below it** so development builds warn at runtime. They are three lines each and they catch different mistakes — the typedef catches the caller, PropTypes catches the data.

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

Test 2 matters because the design specification is **v0.2, and parts of it are explicitly unratified**. Every section carrying a *Revised — v0.2* note is waiting on leadership: **L-2** (feed scoping), **L-3** (the action vocabulary — Buy removed, `borrow` and `subscribe` added), **L-5** (three feed tabs rather than one merged list). Anything whose shape depends on one of those decisions stays out of Phase 0 and is built config-driven in Days 3–5, so a reversal costs an edit rather than a rewrite.

#### The five that pass

| Component | Author | Common because | Stable because |
|---|---|---|---|
| `ContentCard` | Prayas | The most reused component in the app — screens 01, 04, 05, 08, 09, 17, 18 | Design Spec §4.1 guarantees it explicitly: *"The card, badge and layout are identical in every scope; only the set of items and the single resolved action change."* Even if L-2 is reversed, the card does not move. |
| `TopAppBar` | Keshav | Every screen; mounted by the shell in P0-6 | §2.1 fixes the navy; title / back / search variants are structural. Depends on no pending decision. |
| `SearchInput` | Moktik | Both search pipelines — catalogue and institution — share one shell | A text input. L-5 changes what search is *scoped to*, which lives in the pipeline, not the input. |
| `AccessTierBadge` | Akriti | Consumed by ContentCard on Day 3; screens 01, 04, 05, 08, 09, 18 | The three tiers and their three hex codes are fixed in §2.1 and §3.1 and are untouched by the v0.2 revisions. Q-D is about whether wokay *supply* the tier, not what the tiers are. |
| `Skeleton` | Khushi | Eight screens; every surface needs it before it has data | §2.3 and §4.2 ban spinners outright and require dimensions matching final content. A hard, unrevised design rule. |

**One constraint that keeps `ContentCard` in the stable set:** it must take `item: ContentItem` and `access: AccessResult` as props and **compute nothing**. Design Spec §5.1 — *"The UI must never calculate access rights"* — is what makes the card immune to L-2 and L-3. A card that inspects `accessTier` itself, or maps a tier to an action internally, breaks that immunity and becomes rework the moment leadership answers.

#### Deliberately excluded, and why

| Component | Author | Why it is not in Phase 0 |
|---|---|---|
| `Tabs` | Prayas | The three-feed-tab model is **L-5, unratified** — design screen 01 still shows a single mixed list. Built Days 3–5 and **driven from a config list, so tabs are data not code**. If leadership merges the feeds, that is a config edit. |
| `ActionButton` | Akriti | The action vocabulary is **L-3, unratified**. v0.2 removed Buy, added `borrow`, and made `subscribe` B2C-only. Built Days 3–5, with variants driven off the `ActionId` union so adding or removing one is a type change plus a style entry. |
| `ActionBar` | Akriti | Same L-3 exposure. Renders purely from the `actions` array and computes nothing, so the blast radius stays inside the array. |
| `BottomSheet` | Keshav | Fully specified in §2.3 (16px top radius, drag handle, 60–70% height) and genuinely stable — but only Keshav consumes it before Day 5. Fails test 1, not test 2. |

Each Phase 0 component ships with its `.gallery.jsx` entry and its `propTypes` block the same day.

---

### P0-8 · Section 09 dispatch and the two escalations

**Owner:** Akriti (holds the coordination duty in Week 1)
**Duration:** Day 1 afternoon and Day 2 morning
**Blocks:** nothing this week — which is exactly why it is easy to defer, and why it is scheduled

| When | Action |
|---|---|
| Day 1 | Send all four Section 09 question sets — wokay, flambeau, t4targaryen, leadership — each with `src/model/types.js` attached as the assumed contract. Lead each list with the mock-data request, not the questions. |
| Day 1 | Escalate **L-2**, the feed-scoping reversal, to leadership in writing. It contradicts Design Specification §4.1, which is a signed document, and needs ratification rather than tacit acceptance. |
| Day 2 | Confirm with wokay that an **unauthenticated full catalogue feed** will exist (W-1). The whole logged-out model depends on it, and CAP-5 currently says the opposite. |
| Day 2 | Raise the **custom access-state field** with wokay (Q-D). Longest lead time of anything asked for, because it is an addition to their model rather than an exposure of something they already have. |
| Day 3 | Agree the **sign-in handoff contract** with flambeau — what we pass, how control returns, where the user lands — even if their screen lands later. |
| Fri | Hand the dependency board to next week's lead, every item in exactly one of: Asked · Answered · Stubbed · Integrated. |

Silence is safe by design: the contract-first approach means an unanswered question resolves to *they accepted our shape*. That only holds if the shape was actually sent, which is why this is a Day 1 item and not a Friday one.

---

## 4. The Phase 0 merge gate

End of Day 2. Every line true, or Phase 1 does not start.

- [ ] `main` runs on all five machines, iOS and Android simulator
- [ ] `src/theme/tokens.js` merged — all 11 colours, all 6 type styles, font, elevation, spacing, radius
- [ ] Inter loads and renders on both platforms
- [ ] No raw hex, raw spacing value or inline shadow anywhere in `src/components/`
- [ ] Every Phase 0 component passes the §P0-7 admission test — common **and** not downstream of L-2, L-3 or L-5
- [ ] `src/model/types.js` merged; `jsconfig.json` committed; `checkJs` reports clean
- [ ] Every Phase 0 component has a `propTypes` block
- [ ] ~40 items and 8 institutions in fixtures, including all ten awkward cases
- [ ] `validate.js` passes over the entire fixture set
- [ ] Adapter conformance suite passes against `MockAdapter`
- [ ] `MockAdapter` serves every `DataAdapter` method with simulated latency
- [ ] Four tabs navigate; detail push and back work; `/gallery` opens
- [ ] The core five components merged, each with a gallery entry
- [ ] Component convention written up in the README, agreed by all five
- [ ] All four Section 09 sets sent, with the typed shape attached
- [ ] L-2 escalated in writing

---

## 5. Days 3–5 — per person

Feature work begins. Each person also builds their remaining components; **components merge before the screens that consume them.**

### Prayas — adapters and normalisation

**Delivers:** F1, F2, A4 · components `SectionHeader`, `SubjectChip`, `Carousel + PageDots`, `Tabs`
**Difficulty: 8/10** — the hardest block in the week

- Three OPDS 2.0 adapters normalising to one `ContentItem`
- URN identifier extraction for DOI and ISBN — OPDS has no `doi` or `isbn` field; both collapse into `identifier` and `altIdentifier` as URNs
- Pagination behind one interface supporting **both** offset and OPDS next-links (A4), because wokay have not said which they emit (W-6)
- A `useNetworkStatus` hook — moved here from Khushi's `OfflineBanner`, since it is a subscription to native network state rather than presentation

**Contingency:** no wokay sample feeds — build the model from the Readium metadata context and our own fixtures, then send wokay the shape assumed. Build all three adapters against one fixture shape until told otherwise.

**The real risk this week.** Prayas is designing a normalisation model blind. If the guess is wrong, F1 is rewritten in Week 3 and takes A4 and part of D1 with it. Mitigation: pair him with Akriti for an hour on Day 3 to walk the fixture shape against the design specification, and make sure his assumed shape is inside the Day 1 wokay dispatch rather than sent later.

### Moktik — search shell and matching

**Delivers:** search shell, B1 · components `FilterChip`, `VoiceOverlay`
**Difficulty: 6/10**

- The shared search shell — the interaction layer both pipelines consume
- B1 catalogue search: matching, tokenisation and ranking over local fixtures. All three are ours; OPDS supplies at most a search link and its template.

**Contingency:** Q-E unresolved (search link template vs fetch-and-filter) — build fetch-and-filter first. It works either way and is the fallback needed regardless.

**Note.** Fiddly but fully self-contained and testable against fixtures. Nobody waits on Moktik this week.

### Keshav — institutions

**Delivers:** B9, C1 · components `BottomSheet`, `InstitutionRow`, `ListRow`
**Difficulty: 4/10 — but the highest reliability bar in the week**

- B9 institution search — a **separate pipeline** from catalogue search, sharing only the shell
- C1 institution list, with recently-used pinned above the full list
- Client-side search over the 8-institution fixture

**C2 (institution detail) moves to Khushi** — see §7.

**Contingency:** no wokay institution schema — the `Institution` shape in P0-3 is ours; send it to them. OPDS does not model institutions at all, so they may not have considered it.

**Why the reliability bar is high.** Per Section 04 of the delivery plan: institution search is on the authentication critical path, catalogue search is not. If catalogue search breaks, a user browses instead. If institution search breaks, the user cannot sign in, cannot obtain entitlements, and cannot read anything. Low difficulty, high consequence — give this code review attention, not help.

### Akriti — the access spine

**Delivers:** D1 skeleton, F6 · components `ActionButton`, `ActionBar` · plus the coordination duty
**Difficulty: 7/10 for the design, plus ~25% of the week on coordination**

- `resolveAccess(item, session, loan)` — signature, stubbed resolver, session shape. Skeleton only this week; the real resolver is Week 2.
- F6 pending-intent store against a fake auth round trip. **Persist to disk, not memory** — if SAML leaves the app (FL-5), an in-memory store dies on the round trip, and disk is the correct choice either way.
- `ActionButton` has more variants than anything else in the library: Read, Download, Borrow, Subscribe, Sign in, Waitlist, disabled.
- `ActionBar` renders purely from the `actions` array and computes nothing.

**Contingency:** Q-D unanswered (no custom access-state field) — read the tier from the `accessTier` fixture field invented in P0-3, and send wokay that definition as a formal ask.

**Note.** Only a skeleton this week, so the grade is for design difficulty rather than volume. `resolveAccess` is the spine — everything downstream of it is presentation — and a wrong signature is expensive. This rises to ~9/10 in Week 2 with the real resolver, the loan rule and Borrow orchestration.

### Khushi — see §7.

---

## 6. The component library — all 21 components, designed

This section is the canonical component reference for the whole team. Every prop contract in the app is defined here once. If a component's shape needs to change, it changes here first.

### 6.1 Scope — the library is 21 components, not 22

The delivery plan's Section 01 inventory lists 22. **`ProgressBar` is removed**, because it appears only on screens **07 (reader view)** and **08 (library / downloads)** — both t4targaryen, CAP-7. No component team1 builds consumes it, and the library is scoped to team1's own screens.

> **Put this on the dependency board.** t4targaryen question 4 asks *"Who owns the downloaded and offline indication on catalogue items — you or us?"* If the answer is "team1," `ProgressBar` comes back into the library. It is roughly two hours to add, so the risk is small — but it should be a decision, not a discovery.

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

Not repeated in the entries below.

1. **Props in, callbacks out.** No fetching, no store reads, no navigation, no access computation. Design Spec §5.1.
2. **Every value from `theme/tokens.js`.** A raw hex, a raw spacing number or an inline shadow fails review.
3. **JSDoc `@typedef` above, `propTypes` below.** Both — see §1.5.
4. **A `.gallery.jsx` entry the same day**, covering every variant listed in its spec.
5. **Union values come from `types.js`** — `PropTypes.oneOf(ACCESS_TIERS)`, never a retyped array literal.

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

"offline" for a card or row means *degraded but usable* — Design Spec §4.2 requires the library and institution list to keep working behind the banner.

---

### 6.4 Prayas — 5 components

#### 1 · `ContentCard` — Phase 0, Day 2 · Difficulty 6

The most reused component in the app, and the one the design specification explicitly guarantees is invariant across all three feed scopes.

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

**Must not:** inspect `item.accessTier`, map a tier to an action, or decide which action to show. It receives `access` already resolved. This constraint is what makes the card immune to L-2 and L-3 — see the P0-7 admission test.

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

Six filter dimensions eventually feed this — content type, subject, publisher, access state, publication year, Open Access (B2). **The access-state dimension depends on Q-D**, so the chip must render correctly when its dimension has zero available options: the slot is wired but empty, not crashed.

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
    name:     PropTypes.string.isRequired,
    country:  PropTypes.string.isRequired,
    crestUrl: PropTypes.string,
    authType: PropTypes.oneOf(AUTH_TYPES).isRequired,
  }).isRequired,
  variant: PropTypes.oneOf(['default','selected','recently_used']),
  onPress: PropTypes.func.isRequired,
}
```

**The crest fallback is required, not optional.** W-17 is unanswered — wokay may have no crest URLs. Fall back to an initials monogram; the fixture set includes a crest-less institution specifically to force this.
**Must not** render `authType` as a raw string — surface it as a human label ("Institution sign-in") or not at all.

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

#### 15 · `ActionButton` — Day 3 · Difficulty 5 · ⚠ L-3 pending

The most variants of anything in the library.

**Screens:** every screen
**Actions:** the six `ActionId` values — `read` · `download` · `borrow` · `signin` · `subscribe` · `waitlist`
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

**Label and colour derive from a single map keyed by `ActionId`** — not a switch statement scattered through the component. The action vocabulary is **L-3, unratified**: v0.2 removed Buy, added `borrow`, and made `subscribe` B2C-only. When leadership answers, adding or removing an action must be one map entry.

`waitlist` uses `color.wait` (amber — §2.1, "no seats available, waitlist actions").

**The `loading` state is real, not defensive.** Borrow is a network call against finite copies, and the optimistic flip (D11) needs somewhere to show in-flight.

#### 16 · `ActionBar` — Day 3, after `ActionButton` · Difficulty 4 · ⚠ L-3 pending

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

**Renders purely from the `actions` array and computes nothing** — Design Spec §5.1. Given `[]` it renders the Access Restricted treatment, which is the one path on which D8 and screen 14 are reachable: a B2B user arriving at a non-entitled Subscription title through pending intent.
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
**Variants:** `no_query_results` · `no_filter_results` (offers Clear filters) · `no_content`

```js
EmptyState.propTypes = {
  variant: PropTypes.oneOf(['no_query_results','no_filter_results','no_content']).isRequired,
  query:   PropTypes.string,
  onClearFilters: PropTypes.func,
}
```

Copy per the design spec: *"No articles or books match…"* / *"Try adjusting your filters…"*

#### 19 · `ErrorState` — Day 3 · Difficulty 2

**Screens:** 04, 05, 14
**Variants:** `network` (Retry) · `not_found` · **`access_restricted`** (Learn more, no retry)

```js
ErrorState.propTypes = {
  variant:     PropTypes.oneOf(['network','not_found','access_restricted']),
  message:     PropTypes.string.isRequired,   // a string, never an Error object
  onRetry:     PropTypes.func,
  onLearnMore: PropTypes.func,
}
```

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

**States:** `single_format` (renders as a label, not a chooser) · `multiple` · `empty` (no formats declared — render nothing)

```js
FormatSelector.propTypes = {
  formats: PropTypes.arrayOf(PropTypes.shape({
    mime:      PropTypes.string.isRequired,
    sizeBytes: PropTypes.number,
  })).isRequired,
  selectedMime: PropTypes.string,
  onSelect:     PropTypes.func.isRequired,
}
```

**Test against the audiobook fixture** — `audio/*` only, no PDF, no EPUB. A selector assuming at least two document formats will break on it.

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

```jsx
// src/components/EmptyState/EmptyState.jsx
import PropTypes from 'prop-types'

/**
 * @typedef {Object} EmptyStateProps
 * @property {'no_query_results'|'no_filter_results'|'no_content'} variant
 * @property {string}     [query]           echoed back in the no_query_results copy
 * @property {() => void} [onClearFilters]  rendered only for no_filter_results
 */

/** @param {EmptyStateProps} props */
export function EmptyState({ variant, query, onClearFilters }) { /* … */ }

EmptyState.propTypes = {
  variant: PropTypes.oneOf(['no_query_results', 'no_filter_results', 'no_content'])
             .isRequired,
  query: PropTypes.string,
  onClearFilters: PropTypes.func,
}
```

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

```js
// screens/institution/InstitutionDetailView.jsx  — Khushi, pure
/**
 * @param {{ institution: import('../../model/types').Institution,
 *           onSelect: () => void, onBack: () => void }} props
 */
InstitutionDetailView.propTypes = {
  institution: PropTypes.shape({
    name:     PropTypes.string.isRequired,
    country:  PropTypes.string.isRequired,
    crestUrl: PropTypes.string,
    authType: PropTypes.oneOf(AUTH_TYPES).isRequired,
  }).isRequired,
  onSelect: PropTypes.func.isRequired,
  onBack:   PropTypes.func.isRequired,
}
```

`AUTH_TYPES` is imported from `model/types.js` rather than retyped — that import is the thing keeping the component and the contract in step now that there is no compiler to do it.

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

All twenty-one components are looked at side by side in the gallery. Checking for: divergent spacing, off-token colours, inconsistent prop naming, missing state variants, two components that should be one — and, in the JavaScript build, **any component missing its `propTypes` block or its JSDoc typedef.**

**This review is the single thing that pays for splitting the library five ways.** Skip it and the drift will not surface until Week 3, by which point every screen has consumed the drifted components. Anything found is fixed before Monday, not logged.

### 8.3 Friday handover — 17:00

Dependency board to next week's lead. Every item in exactly one state: **Asked · Answered · Stubbed · Integrated**. The board moves with the rota, never owned by one person for longer than a week.

---

## 9. Risks

| # | Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|---|
| R1 | Phase 0 overruns into Day 3 | High — greenfield always does | Compresses feature work to two days | Timebox the scaffold to Day 1 morning. Take the recommended stack defaults; do not debate them. If Day 2 evening is not green, cut the core five to three (ContentCard, TopAppBar, Skeleton) rather than cutting types or fixtures. |
| R2 | Prayas's normalisation model is wrong | Medium | F1 rewrite in Week 3, drags A4 and D1 | Pair with Akriti on Day 3. Get the assumed shape into the Day 1 wokay dispatch, not a later one. |
| R3 | Q-D (`accessTier`) unanswered by end of Week 2 | Medium | B5 filter unbuildable; badges and actions run on invented data indefinitely | Asked Day 1, chased Day 2, escalated end of Week 2. Fixture field already invented so nothing stops. |
| R4 | Five authors, five styles | Medium | Visible inconsistency at demo | Tokens Day 1, one skeleton, Day 5 review. All three are required — any two is not enough. |
| R5 | Feature work introduces components outside the library | High under deadline | Two Cards by Week 3 | The P0-5 rule, enforced at review. The most likely rule to break quietly. |
| R6 | Khushi blocked despite the re-cut | Low | Minimal — nothing depends on her | She can build every K-item against the gallery with no running app. Escalate same day if she is idle. |
| R7 | L-2 not ratified | Medium | Building against a reversed spec with no written cover | Escalated Day 1. If no answer by Friday, it goes on the dependency board as an open escalation, not a resolved item. |
| R8 | **Contract drift, undetected — the JavaScript risk** | Medium–High | A shape mismatch between adapter and screen surfaces in Week 4 integration, which is the worst possible week for it | The four replacements in §1.5, all landing in Phase 0: `checkJs` + JSDoc, `validate.js` over fixtures, the adapter conformance suite, PropTypes on every component. **None of these is optional in the JavaScript version** — together they are the compiler. |
| R9 | PropTypes and JSDoc quietly skipped under deadline pressure | High | R8 becomes certain rather than possible | Both are review-blocking, same as the raw-hex rule. Cheapest place to enforce is the Day 5 consistency review, where the gallery makes a missing `propTypes` block immediately visible. |

---

## 10. Week 1 exit criteria

From the delivery plan, plus what this document adds:

- [ ] Catalogue browsable and searchable on mock data
- [ ] Institution list, detail and search working
- [ ] Component library — all 21 merged (§6), each with a gallery entry and a `propTypes` block
- [ ] State gallery route complete and reviewed
- [ ] Consistency review held, findings fixed before Monday
- [ ] Contracts sent in writing to all four audiences, including the `accessTier` request
- [ ] L-2 escalated; W-1 and Q-D raised with wokay; sign-in handoff agreed with flambeau
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

| Ref | Question | Stub in place |
|---|---|---|
| W-1 | Can an unauthenticated user fetch the full catalogue? | Fixtures serve everything to everyone |
| Q-D / D4 | Will wokay add a per-item access-state field? | `ContentItem.accessTier`, invented in P0-3 |
| Q-E / W-5 | Search link template, or fetch-and-filter? | Fetch-and-filter behind an interface |
| W-6 | Pagination — offset, cursor, or next links? | A4 supports offset and next links both |
| W-13 | Is item metadata complete in the feed? | Render what exists, leave gaps blank |
| W-17 | Institution crest URLs? | Initials placeholder in InstitutionRow |
| FL-5 | Does SAML leave the app, and how does control return? | Pending intent persisted to disk |
| L-2 | Ratification of the feed-scoping reversal of Design Spec §4.1 | Build the scoped model; escalated Day 1 |
| L-3 | Ratification of the action vocabulary — Buy removed, `borrow` and `subscribe` added | `ActionId` union; ActionButton excluded from Phase 0 |
| L-5 | Ratification of three feed tabs over one merged catalogue | Build tabs from a config list |
| Q-V | Signing in makes Open Access *harder* to reach — guest gets Read/Download, signed-in user must Borrow first | Falls out of the loan rule; implement as specified, flagged for the first demo |
| — | Spacing, pill radius and card shadow, inferred not specified | §P0-2 values; flagged as inferred |

---

## Appendix C — Traceability against the Design Specification

Audited against `TF_Reader_Design_Package/docs/TF_Reader_Design_Specification.md` v0.2.

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
| §3.1 | `resolveAccess(item, session, loan)` signature | P0-3 — three arguments, `session` and `loan` both nullable |
| §3.1 | `tier` — 3 values | P0-3 `AccessTier` — exact match |
| §3.1 | `state` — 5 values | P0-3 `AccessState` — exact match |
| §3.1 | `actions` — 6 values plus empty | P0-3 `ActionId` — exact match |
| §3.1 | Loan is per-user, per-item, mutable, from flambeau | P0-3 `Loan`; never a property of `ContentItem` |
| §3.1 | `no_seats` is a live path — loans draw on finite copies | `ContentItem.copies`; fixture case "Elite with 0 available" (P0-4) |
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
