# T&F Reader — Design Specification Document

**Project:** T&F Reader (Unified Taylor & Francis App)
**Platform:** React Native (Platform-Neutral)
**Version:** 0.2 (Revised — feed scoping, access actions, three feed tabs, Borrow/loan model; sections marked *Revised — v0.2* are pending ratification per delivery plan L-2, L-3 and L-5)
**Author:** Manus AI

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
resolveAccess(item, session, loan) → {
  tier:    'open_access' | 'subscription' | 'elite',
  state:   'available' | 'requires_loan' | 'requires_signin'
         | 'not_entitled' | 'no_seats',
  actions: ['read'] | ['read','download'] | ['borrow']
         | ['signin'] | ['subscribe'] | ['waitlist'] | []
}
```

The resolver takes a **third input**: the user's current loan state for this item, read from flambeau (CAP-4). It is per-user, per-item and mutable, so it cannot be carried as a static property of the OPDS feed.

**The loan rule, in one line: a loan is required for anything the user can take a copy of.**

*   **Open Access, logged out:** `available` with `['read','download']`. No loan — there is no borrower to lend to.
*   **Open Access, signed in:** `requires_loan` with `['borrow']` on first tap; `available` with `['read','download']` once on loan.
*   **Subscription, entitled B2B:** `requires_loan` with `['borrow']` on first tap; `available` with `['read','download']` once on loan. Guests get `requires_signin`.
*   **Elite:** `available` with **`['read']` only — never Download, and never Borrow.** The user takes no copy, so no loan is created. Guests get `requires_signin`.
*   **No copies free:** `no_seats` with `['waitlist']`, on any tier that requires a loan.
*   **Not entitled:** `not_entitled`. For **B2C** the action is `['subscribe']`; for **B2B** it is `[]` — Access Restricted, since Subscribe is B2C-only. Reachable only via the Pending-Intent Store: a guest taps a gated item, signs in, and the resolved session turns out not to cover it. Because the post-sign-in feed is scoped, this state never appears in the feed itself.

> **Revised — v0.2.** Three changes. (1) `subscribe` is a new action and is **B2C-only** — a B2B session never surfaces it. (2) `Buy` is removed from the prototype. (3) **`borrow` is added, and does not mean what the removed Borrow meant.** The removed action was a purchase-adjacent path for B2C users on content they were *not* entitled to. This one is loan creation on content the user **is** entitled to — the library lending model, flambeau CAP-4. Consequently `no_seats` is now a live path rather than defensive scaffolding, since loans draw against finite copies.

> **Open question (Q-V).** Signing in makes Open Access *harder* to reach: a guest gets Read and Download immediately, while a signed-in user must Borrow first. This falls directly out of the loan rule and may be intended, but it is flagged for review at the first demo.

### 3.2 The Pending-Intent Store

To ensure a seamless authentication flow, the app utilizes a Pending-Intent Store. When a user taps a gated item, the item's ID is stored before launching the Access Gate or Sign-In flows. Upon successful authentication, the app reads this store and returns the user directly to the original item, immediately re-resolving its access state — including its **loan state**, which must be fetched before the action bar can render. An item that showed *Sign in* to a guest may resolve to *Borrow* rather than *Read* once the session exists.

---

## 4. Screen Specifications

The application is structured around a bottom tab navigation system with four primary tabs: **Catalogue**, **Search**, **Library**, and **Profile**.

### 4.1 Priority Screens

#### 01 — Catalogue Home
The primary discovery surface. Features a featured content carousel at the top, followed by a "Browse by Subject" section with horizontal pill selectors. The main content area is a vertical list of article and book cards.

> **Revised — v0.2.** This screen has **three feed scopes**, not one. Logged out, it displays the full catalogue — every institution, all four content types. Signed in as **B2B**, it is scoped to entitled content + Open Access + Elite. Signed in as **B2C**, it shows Open Access plus anything the individual holds. This reverses the v0.1 statement that the screen is "never filtered by institution", and is pending ratification by leadership (see delivery plan L-2). The card, badge and layout are identical in every scope; only the set of items and the single resolved action change.

> **Revised — v0.2.** wokay serve **three separate OPDS endpoints**, and this screen surfaces them as **three tabs** rather than the single mixed list shown in the v0.1 mockup. A segmented control sits above the content area; the featured carousel and subject chips are shared across tabs, and the vertical list below is scoped to the active feed. Merging the three would require cross-feed deduplication, merged pagination across three independent cursors, and re-ranking a union of three result sets carrying no comparable relevance scores. Search (screen 09) is likewise scoped to the active tab — its existing content-type tabs map onto this directly. Pending ratification (delivery plan L-5).

#### 03 — Access Gate Sheet
A bottom sheet modal triggered when a user interacts with gated content. It presents two clear routing options: "Through my institution" (for SAML/SSO flows) and "Personal account" (for email authentication). This screen bridges the gap between the catalogue and the authentication mechanisms.

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

1.  **No Hardcoded Logic:** The UI must never calculate access rights. It must rely entirely on the `resolveAccess` function.
2.  **Session Independence:** The Library screen and guest download features must be built without assuming a `userId` exists.
3.  **Data Adapters:** The design accommodates swappable data layers. The UI expects a normalized internal shape, regardless of whether the data comes from the `MockAdapter` (local fixtures) or the `ApiAdapter` (real OPDS 2.0 feeds).

---

## 6. Deliverables

The accompanying files provide the complete visual reference for this specification:

*   **`index.html`**: A fully interactive HTML design system and screen showcase. Open this file in a web browser to view the color palette, typography, components, and all 18 generated screen mockups in a structured gallery.
*   **`/screens` Directory**: Contains the 18 high-fidelity, AI-generated mobile mockups (1440x2560px) ready for import into Figma as reference images or assets.

*Note: As direct `.fig` file generation is not supported by the current toolchain, the `index.html` file serves as the definitive, interactive design system reference, and the PNG assets are provided for direct import into your Figma workspace.*
