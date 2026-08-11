# T&F Reader — Design Specification Document

**Project:** T&F Reader (Unified Taylor & Francis App)
**Platform:** React Native (Platform-Neutral)
**Version:** 0.3 (Revised — the §3.1 access matrix corrected against wokay's published contract, and §5 extended with the backend's four hard client rules; `subscribe` removed with B2C; `no_seats` scoped to Elite; Q-V closed. Sections marked *Revised — v0.2* remain pending ratification per delivery plan L-2 and L-5)
**Author:** Manus AI

> ### ⚠ v0.3 — §3.1 is corrected, and the correction is not yet ratified
>
> The v0.2 access matrix was **inverted on Elite and on Open Access**. It was derived from the rule *"a loan is required for anything the user can take a copy of"*, which is not the rule the backend implements. wokay's source of truth (§02 table, Flows A/B/C) states the actual rule: **a loan exists for anything encrypted — Subscription and Elite — and a lease or queue exists only where copies are finite, which is Elite alone.** Open Access has no entitlement, no licence and no loan record, ever.
>
> Concretely, v0.2 said Open Access requires a Borrow once signed in (it does not), and that Elite never creates a loan and never shows Borrow (it is the *only* tier that does). §3.1 below carries the corrected matrix, because a builder coding against the v0.2 table would write a resolver that is wrong on two of three tiers. §5 gains four rules the backend states as non-negotiable for clients, and a new §5.4 listing the two fields it does not have.
>
> **This edit is to a signed document and requires leadership ratification.** It is bundled with the L-2 escalation in the delivery plan, and it is the more urgent half of that conversation. Until ratified, treat §3.1 as *corrected-but-unratified*: build to it, and say so in writing.

---

## 1. Executive Summary

This document outlines the design specification for the **T&F Reader**, a unified React Native mobile application designed to consolidate Taylor & Francis books, journals, articles, and multimedia content into a single interface. The design language is derived directly from the existing Taylor & Francis Online (tandfonline.com) and Taylor & Francis Group eBook platforms, ensuring brand consistency while providing a modern, mobile-first user experience.

The design system prioritizes clarity, academic readability, and a strict separation of concerns between content presentation and access resolution. The frontend architecture explicitly decouples UI rendering from licensing logic, relying on a single `resolveAccess` contract to determine available actions for any given item.

---

## 2. Design System & Visual Language

The visual language is built upon a professional, academic aesthetic characterized by generous whitespace, clear typographic hierarchy, and a distinct color palette anchored by the T&F brand teal.

### 2.1 Color Palette

The color system utilizes a core set of brand colors for primary interactions, supported by semantic colors for status indication and access tiers.

| Role | Color Name | Hex Code | Usage Context |
|------|------------|----------|---------------|
| Primary | T&F Teal | `#00A19D` | Primary buttons, active tabs, links, interactive elements |
| Navigation | Deep Navy | `#1A3A5C` | Top navigation bars, dark mode overlays |
| Text (Primary) | Charcoal | `#1A1A2E` | Headings, body text, primary labels |
| Text (Secondary) | Medium Gray | `#6B7280` | Metadata, subtitles, disabled states |
| Surface | Off-White | `#F8F9FA` | Card backgrounds, subtle section backgrounds |
| Border | Light Gray | `#E5E7EB` | Input fields, card borders, dividers |
| Success | Green | `#10B981` | Open Access badges, successful downloads |
| Error | Red | `#EF4444` | Access restricted states, destructive actions |
| Wait | Amber | `#F59E0B` | No seats available, waitlist actions |
| Subscription | Blue | `#2563EB` | Subscription access badges |
| Elite | Purple | `#7C3AED` | Elite DRM access badges |

### 2.2 Typography

The typography scale utilizes the **Inter** font family (or closest system sans-serif equivalent), chosen for its high legibility and clean, modern appearance that complements academic content.

| Element | Weight | Size | Line Height | Usage |
|---------|--------|------|-------------|-------|
| Page Title | Bold (700) | 24px | 32px | Top-level screen headings |
| Section Header | Semi-Bold (600) | 18px | 24px | Card titles, section dividers |
| Body Text | Regular (400) | 15px | 22px | Standard article/book text |
| Meta/Caption | Regular (400) | 13px | 18px | Author names, dates, file sizes |
| Button | Semi-Bold (600) | 15px | 20px | Primary and secondary actions |
| Small Label | Medium (500) | 12px | 16px | Badges, tags, form labels |

### 2.3 Component Architecture

The component library is designed for modularity and reuse across all screens.

*   **Cards:** Used for article and book listings. Features an 8px border radius, subtle box shadow, and a consistent layout with a thumbnail on the left and metadata on the right.
*   **Bottom Sheets:** Used for authentication flows and filtering. Features a 16px top border radius, a drag handle, and covers approximately 60-70% of the screen.
*   **Badges:** Pill-shaped indicators for access tiers (Open Access, Subscription, Elite) and download status.
*   **Skeleton Loaders:** Replaces traditional spinners. Uses rectangular placeholders matching the exact dimensions of the final content to prevent layout shifts during loading.

---

## 3. Core Architecture & Access Model

The most critical architectural decision in this design is the separation of the UI from the access resolution logic. The frontend consumes a single contract to render the appropriate interface, ensuring that swapping the mock resolver for the real implementation requires zero UI changes.

### 3.1 The `resolveAccess` Contract

Every screen renders based on the output of the following function:

```typescript
resolveAccess(item, session, loan, availability) → {
  tier:    'OPEN_ACCESS' | 'SUBSCRIPTION' | 'ELITE',
  state:   'available' | 'requires_loan' | 'requires_signin'
         | 'not_entitled' | 'no_seats',
  actions: ['read'] | ['read','download'] | ['borrow']
         | ['signin'] | ['waitlist'] | []
}
```

The resolver takes a **third input**: the user's current loan state for this item, read from flambeau (CAP-4). It is per-user, per-item and mutable, so it cannot be carried as a static property of the OPDS feed.

It takes a **fourth, nullable input**: live copy availability. This is `null` on every list surface and non-null only on item detail, because `copies.available` does not exist in the feed — the feed carries `copies.total` only, and `available` comes from a separate per-item call (`GET /api/v1/availability?itemId=`, flambeau, Elite only). See §5.4.

#### The resolver reads the acquisition link. It does not reason from the tier.

**This is the single most important rule in this section, and v0.2 got it wrong.** The backend sends an OPDS acquisition link on every publication, and *that link* determines which buttons exist. The tier is a **label on a badge**, not an input to action derivation:

| Acquisition link | Actions |
|---|---|
| `rel=open-access` | `['read','download']` |
| `rel=acquisition` + `canPersist: true` | `['borrow']` → once on loan, `['read','download']` |
| `rel=borrow` + `canPersist: false` | `['borrow']` → once on loan, `['read']` |
| no link sent | `[]` — the button does not exist |

Session still decides signed-in versus `requires_signin`. Loan state still comes from flambeau. Availability still gates `no_seats`. But **no branch in the resolver may read `item.accessTier` to decide an action.** A resolver that maps tier → actions duplicates the backend's entitlement logic in the client, which is exactly what §5.1 forbids, and it desynchronises the moment the backend changes a licence model.

#### The corrected access matrix

**The loan rule, in one line: a loan exists for anything encrypted — Subscription and Elite. A lease and a queue exist only where copies are finite, which is Elite alone.**

| Tier | Logged out | Signed in, entitled | Signed in, not entitled |
|---|---|---|---|
| `OPEN_ACCESS` | `available` · `['read','download']` | `available` · `['read','download']` | `available` · `['read','download']` |
| `SUBSCRIPTION` | `requires_signin` · `['signin']` | `requires_loan` · `['borrow']` → `available` · `['read','download']` | `not_entitled` · `[]` |
| `ELITE` | `requires_signin` · `['signin']` | `requires_loan` · `['borrow']` → `available` · **`['read']`** | `not_entitled` · `[]` |
| `ELITE`, no copies free | — | `no_seats` · `['waitlist']` (creates a hold, returns queue position) | — |

*   **Open Access** has no entitlement, no licence and no loan record, in any session state — including anonymous. It is stored as plaintext precisely so an anonymous reader can open it. Signing in changes nothing about it.
*   **Subscription** is encrypted and loan-backed but **unlimited**: there is nothing to reserve and nothing to run out of, so **no queue can ever form on it.** Download is permitted (`canPersist: true`); access ends at the due date.
*   **Elite** is the only tier that borrows against a finite pool. Borrow consumes one of N copies via a Redis lease; when none are free the request writes a **hold** and returns a queue position. **Download is refused server-side** (`canPersist: false`, `DOWNLOAD_NOT_PERMITTED`), so Elite is `['read']` only — online reading, nothing written to the device.
*   **`no_seats` is Elite-only, and detail-screen-only.** It cannot be resolved on a `ContentCard`, because `copies.available` is not in the feed. On a list surface an Elite item with zero free copies is indistinguishable from one with copies free; it resolves to `requires_loan` and the hold is discovered on the detail screen or on the Borrow response.
*   **Not entitled:** `not_entitled` with `[]` — Access Restricted. Reachable only via the Pending-Intent Store: a guest taps a gated item, signs in, and the resolved session turns out not to cover it. Because the post-sign-in feed is scoped, this state never appears in the feed itself.

> **Revised — v0.3.** Four changes, all reconciling this section against wokay's published contract. (1) The matrix above **replaces** the v0.2 list, which was inverted on Elite and Open Access. (2) ~~`subscribe` is removed from the action vocabulary, along with the B2C session type.~~ **AMENDED 11 August — `subscribe` and the B2C session type are RETAINED.** wokay recommended cutting individual subscribers at their own gate (decision 7) and that recommendation is not being taken; they will supply the B2C details later. The action vocabulary is six values, not five. The anonymous public feed is *a* skip-institution path, not the only one. (3) **`no_seats` is scoped to Elite** and is unreachable on list surfaces. (4) Tier values are **uppercase**. *Amended 11 August: this was justified as matching `catalogueItems.accessTier`, but **no such field exists** — the tier is derived from the acquisition link's `licenceModel` (`UNLIMITED` → Subscription, `CONCURRENT` → Elite, absent → Open Access, wokay's own mapping). The uppercase convention stands; the reason for it has changed. The derivation happens during normalisation, never in a component, so §5.1 still holds.*

> **`borrow` still does not mean what the removed Borrow meant.** The action removed in v0.2 was a purchase-adjacent path for B2C users on content they were *not* entitled to; it stays removed, along with Buy. This one is loan creation on content the user **is** entitled to — the library lending model, flambeau CAP-4.

> **Q-V is closed.** The v0.2 oddity — *"signing in makes Open Access harder to reach"* — was an artefact of the incorrect loan rule and does not exist. Open Access resolves identically in every session state.

### 3.2 The Pending-Intent Store

To ensure a seamless authentication flow, the app utilizes a Pending-Intent Store. When a user taps a gated item, the item's ID is stored before launching the Access Gate or Sign-In flows. Upon successful authentication, the app reads this store and returns the user directly to the original item, immediately re-resolving its access state — including its **loan state**, which must be fetched before the action bar can render. An item that showed *Sign in* to a guest may resolve to *Borrow* rather than *Read* once the session exists.

---

## 4. Screen Specifications

The application is structured around a bottom tab navigation system with four primary tabs: **Catalogue**, **Search**, **Library**, and **Profile**.

### 4.1 Priority Screens

#### 01 — Catalogue Home
The primary discovery surface. Features a featured content carousel at the top, followed by a "Browse by Subject" section with horizontal pill selectors. The main content area is a vertical list of article and book cards.

> **Revised — v0.3.** This screen has **two feed scopes**, not three. Signed in, it renders the institution's root catalogue feed, which the backend has already scoped to what that institution is entitled to — plus Open Access, which appears for every institution regardless of entitlement. Anonymous, it renders `GET /opds/v1/public/catalogue`, which is **open access only** — not the full catalogue. The third (B2C) scope is deleted along with individual subscribers (wokay gate decision #7). The card, badge and layout are identical in both scopes; only the set of items and the single resolved action change.
>
> **L-2 changes character.** It is no longer *"will leadership ratify the reversal"* — the backend **cannot serve** the v0.1 model. There is no unauthenticated full catalogue; the anonymous feed is open-access-only, full stop. This still needs escalating, but as a statement of fact backed by the backend team's published contract rather than as a request. That is a stronger letter.

> **Revised — v0.3.** The "three separate OPDS endpoints" premise in v0.2 is not what the backend serves. wokay serve **one root catalogue feed per institution** (`GET /opds/v1/institutions/{id}/catalogue`) carrying **navigation rows and groups** — shelves — and one root feed draws this whole screen. A shelf is then paged via `GET .../groups/{gid}`. Shelves are therefore **discovered per institution and configured per institution** (the admin console has a per-institution *catalogue config*: feed title, shelf order, page size), not three fixed endpoints known at build time.
>
> This screen still surfaces shelves as a segmented control above the content area, with the featured carousel and subject chips shared across it, and search (screen 09) still scoped to the active shelf. What changes is that **the tab set is data, read from the root feed's navigation rows, never a hardcoded list of three.** The v0.2 reasoning for tabs-over-merge still holds — it is simply now enforced by the feed shape rather than chosen by us.
>
> **Content type is a filter, not a shelf split.** wokay's `contentType` is `PDF | EPUB | AUDIO` — a *file format* — offered as a free query parameter on the endpoints already being called. It is not a work type, and it does not partition the feed. See §5.4 on the missing work-type field.
>
> L-5 stays open rather than closing: the shelf set is the institution's to configure, so leadership should still ratify that the home screen presents shelves as tabs, and the backend may re-cut shelves at any point. Because the tab bar is config-driven, a re-cut costs nothing.

#### 03 — Access Gate Sheet
A bottom sheet modal triggered when a user interacts with gated content. It presents two routing options: **"Through my institution"** and **"Browse free content"**.

> **Revised — v0.3, then amended 11 August.** The v0.3 revision removed the second option, *"Personal account (email authentication)"*, on the basis that individual subscribers were cut. **That basis no longer holds — B2C is retained, so this screen's second option is reopened rather than settled.** It may carry the personal-account entry, the browse-free-content entry, or both; wokay will supply the B2C details before this screen is built in Week 2. Original v0.3 note follows. Individual subscribers were cut from scope (wokay gate decision #7) and **institutional sign-in is always SAML** — there is no `authMethod` to branch on and no email path to route to. The second option therefore becomes the **anonymous open-access entry point** (`GET /opds/v1/public/catalogue`), which is the skip-institution path the B2C option used to be. Screen 13 (email sign-in) has no owner under this model.

#### 04 & 05 — Item Details (Article & Book)
These screens present the metadata for a specific item. They include the title, author, journal/publisher info, and the abstract or description. The action bar at the bottom dynamically renders the `Read` and `Download` buttons based on the `resolveAccess` state.

#### 08 — Library / Downloads
A critical screen that must function flawlessly both with and without an active session. It lists downloaded content, shows download progress bars, and includes a specific section for "Guest Downloads" that warns the user these items may not persist across devices.

### 4.2 Supporting Screens & States

The design system requires specific handling for edge cases to maintain a polished user experience.

*   **Loading State:** All screens utilize skeleton loaders matching the final layout. No spinners are used to avoid layout jumping.
*   **Empty States:** Distinct messages are used for empty search results ("No articles or books match...") versus empty filter results ("Try adjusting your filters...").
*   **Offline State:** A persistent dark gray banner appears at the top of the screen. The library remains fully functional, and an "Offline Library" card is prominently displayed on the catalogue.
*   **Error States:** Plain text error messages with a "Retry" or "Learn more" action. No stack traces are exposed to the user.

---

## 5. Implementation Guidelines

To ensure the development teams can execute this design effectively:

1.  **No Hardcoded Logic:** The UI must never calculate access rights. It must rely entirely on the `resolveAccess` function — and `resolveAccess` must in turn derive its actions from the acquisition link, not from the access tier. §3.1. These are the same rule stated at two altitudes, and the backend states it a third time as *"the acquisition link decides the buttons, not your own logic. If we do not send a link, the button does not exist."*
2.  **Session Independence:** The Library screen and guest download features must be built without assuming a `userId` exists.
3.  **Data Adapters:** The design accommodates swappable data layers. The UI expects a normalized internal shape, regardless of whether the data comes from the `MockAdapter` (local fixtures) or the `ApiAdapter` (real OPDS 2.0 feeds).
4.  **Follow hrefs; never build URLs.** The only URL construction permitted anywhere in the app is expanding the templated search link, `".../search{?query}"`. The institution detail response hands over `catalogueUrl`; every feed hands over its own `next` link. **Paginate by following `next` until it is absent — never count pages.** Two consequences: there is no page arithmetic to get wrong, and OPDS pagination and the institutions REST endpoint (`?page=&size=`, which *does* return `{items, page, size, total}`) are two different models split cleanly by endpoint family.
5.  **An empty result is not an empty array.** The OPDS schema forbids empty arrays, so a zero-result search returns a **feed containing a navigation entry** pointing back at the catalogue. Render that as a *browse instead* affordance. A missing `publications` key is not an error and must not surface as one.
6.  **Errors are typed, and the copy is driven by the code.** Every failure arrives in one envelope — `{timestamp, status, code, message, path, traceId}` — with an enumerated `code`: `NO_ENTITLEMENT`, `ENTITLEMENT_EXPIRED`, `ENTITLEMENT_SUSPENDED`, `CONTENT_NOT_READY`, `DOWNLOAD_NOT_PERMITTED`, `FORBIDDEN_INSTITUTION_MISMATCH`, `INVALID_DEVICE_PUBLIC_KEY`, `NOT_FOUND`. Denials carry a reason specifically so the app can say *"your library's subscription has expired"* rather than *"not available"*. `ErrorState` copy is therefore keyed on `code`, never on HTTP status alone.

### 5.4 Two fields the backend does not have

Flagged here rather than assumed, because screens depend on both:

*   **Work type. ✅ Largely resolved, 11 August.** wokay's sample feeds label every publication with a schema.org type — `http://schema.org/Book`, `http://schema.org/Audiobook` — and wokay have confirmed it is contractual. Screens **04 (article)** and **05 (book)** therefore have the axis they need. **Only the journal and article values remain outstanding**; wokay will supply them. Until then the adapter maps two marked guesses and falls back on anything unrecognised.
*   **DOI. ✅ Closed, 11 August — there is no DOI, and there will not be one.** Only wokay's own `identifier` exists. **DOI is removed from screen 04 (deliverable E2).** This is a design change to make, not an answer to wait for.
*   **ISBN — corrected, and it moved the wrong way.** The top-level `isbn` on `catalogueItems` does **not** appear in the feed. The feed emits `metadata.identifier` as a URN (`urn:isbn:9780367211745`), and publications without an ISBN carry an internal URN instead (`urn:tf:catalogue:item_env`). Unwrapping is required after all.

Also note: `copies.available` is not in the feed (see §3.1). **`publishedYear` is resolved** — the samples carry `metadata.published` as an ISO date (`2020-09-30`), a real publication date rather than the ingest `createdAt` this section previously feared.

> **A precedence rule that now governs this whole section.** wokay confirmed on 11 August that **their sample feeds are the current contract and their source-of-truth document is out of date.** Where the two disagree, the sample wins. Two fields this specification took from that document — `accessTier` and the top-level `isbn` — turned out not to exist in the feed at all. Treat anything sourced only from their prose as unverified until a sample shows it.

---

## 6. Deliverables

The accompanying files provide the complete visual reference for this specification:

*   **`index.html`**: A fully interactive HTML design system and screen showcase. Open this file in a web browser to view the color palette, typography, components, and all 18 generated screen mockups in a structured gallery.
*   **`/screens` Directory**: Contains the 18 high-fidelity, AI-generated mobile mockups (1440x2560px) ready for import into Figma as reference images or assets.

*Note: As direct `.fig` file generation is not supported by the current toolchain, the `index.html` file serves as the definitive, interactive design system reference, and the PNG assets are provided for direct import into your Figma workspace.*
