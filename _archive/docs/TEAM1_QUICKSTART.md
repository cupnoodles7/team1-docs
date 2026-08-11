# Team1 Quickstart — Discovery & Selection

Short version of [GITHUB_WORKFLOW_AND_CICD.md](GITHUB_WORKFLOW_AND_CICD.md), written from team1's seat. When the two disagree, the master doc wins.

**Team1 owns:** CAP-2 (institution listing) + CAP-3 (institute selection)
**People:** Akriti Khetan, Khushi S Shukla, Prayas Yadav, Moktik, Keshav Sharma — lead **TBD** (lead forks, other four clone)

---

## Our repos

| Ref | Fork we work in | Upstream | Do we own backend code here? |
|---|---|---|---|
| `b1` | `tf_reader_mobile_app` fork | `b` | **Yes** — CAP-2/CAP-3 UI is React Native |
| ~~`a1`~~ | ~~`tf_reader_backend` fork~~ | `a` | **No — settled.** Don't fork it |
| ~~`c1`~~ | ~~`tf_reader_admin` fork~~ | `c` | **No — settled.** Admin is wokay's |

**Resolved by wokay's source of truth:** *"team1 and t4targaryen have NO backend module. React Native only."* The backend is one Spring Boot process with four modules, all wokay's and flambeau's, and the admin console is wokay's React app against their own module.

**So the lead forks one repo, not three** — and `backend-dev-team1` never exists, which removes two environments and two sets of fork secrets from the setup below.

---

## One-time setup

```bash
# lead only: fork b via GitHub UI → this is b1. Do NOT fork a or c; we own neither.

# everyone else:
git clone git@github.com:<lead-or-team-org>/tf_reader_mobile_app.git
cd tf_reader_mobile_app
git remote add upstream git@github.com:<org>/tf_reader_mobile_app.git
```

**Remember:** in `b1`, `main` is *team1's dev line*, not production. Production is `main` on upstream `b`, which we never touch.

**Lead also does, once:**
- Set repo variable `TEAM_SLUG = team1` (Settings → Variables → Actions) — this is what names our dev env `mobile-dev-team1`
- Create the `dev` GitHub Environment in the fork and add our dev secrets. Upstream secrets do **not** reach forks.
- Add `CODEOWNERS`, `CODE_REVIEW.md`, `CONTRIBUTORS.md`
- Turn on branch protection for `b1:main` (PR required, 1 approval, CI green, linear history)

---

## Our branches

```
feature/CAP-2-institution-list
feature/CAP-2-institution-search
feature/CAP-2-institution-detail
feature/CAP-3-select-institution
feature/CAP-3-persist-selection
feature/CAP-3-auth-routing
```

---

## Daily loop

```bash
# 1. sync (every session)
git checkout main && git pull --ff-only origin main

# 2. branch
git checkout -b feature/CAP-2-institution-list

# 3. work. rebase onto main daily:
git checkout main && git pull --ff-only origin main
git checkout - && git rebase main

# 4. push every 2-3 days, even if unfinished
git push -u origin feature/CAP-2-institution-list
```

---

## Before every PR

- [ ] Rebased on `b1:main`
- [ ] Tests pass
- [ ] **Read your own `git diff` in VS Code** — `git diff main...HEAD`
- [ ] README for the feature if it adds setup/env vars/a new screen contract
- [ ] Claude Code: `/security-review` on the diff
- [ ] Claude Code: optimization + regression + code-quality pass
- [ ] No secrets, no real institution data

Then: **PR into `b1:main`** — never into upstream. 1 approval from a teammate. Squash merge.

---

## Our CODEOWNERS starting point

Drop into `.github/CODEOWNERS` in `b1`, replacing handles:

```gitignore
*                                  @team1-lead

# CAP-2 — institution listing
/src/screens/InstitutionList/**    @handle-a @handle-b
/src/screens/InstitutionDetail/**  @handle-a @handle-b
/src/api/institutions.ts           @handle-a @team1-lead

# CAP-3 — institute selection
/src/screens/InstitutionSelect/**  @handle-c @handle-d
/src/store/selection/**            @handle-c @team1-lead
/src/navigation/authRouting.ts     @handle-c @team1-lead   # ← flambeau handoff

# cross-cutting — always notify
/.github/workflows/**              @platform-owner
/package.json                      @platform-owner @team1-lead
```

Two files deserve extra care because they're **cross-team contracts**, not just our code. Both are now concrete rather than speculative:

- **`api/institutions.ts`** — shaped by wokay's institution schema, which is **published**. Two unauthenticated endpoints:
  - `GET /api/v1/institutions?q=&country=&page=&size=` → `{items, page, size, total}`, where an item is `{id, code, name, type, country, city, logoUrl}`. **Server-side search and server-side paging** — not a client-side filter over a fetched list.
  - `GET /api/v1/institutions/{id}` → adds `branding{logoUrl, primaryColor}`, `signIn{method:'SAML', idpHint}` and `catalogueUrl`.
  - An inactive institution returns **404, not 403** — deliberately, so its existence isn't disclosed. Don't map it to a "forbidden" state.
  - `catalogueUrl` is handed to us so **we never build wokay's URLs**. Follow hrefs.

- **`navigation/authRouting.ts`** — **no longer a branching file.** Institutional sign-in is *always* SAML; there is no `authMethod` field on wokay's institution record. This file reads `signIn.idpHint` from institution detail and passes it, with `institutionId`, into flambeau. That's it. `AUTH_TYPES` is deleted from `model/types.js`, so anything importing it needs updating in the same commit.

Add the relevant other-team lead as a reviewer on any PR touching those.

---

## Our environments

| Env | What it is | Who deploys |
|---|---|---|
| personal dev | your laptop, docker-compose | you |
| `mobile-dev-team1` | auto-deploys on merge to `b1:main` | CI |
| ~~`backend-dev-team1`~~ | **Never exists** — we own no backend module | — |
| `mobile-stage` | shared with all 4 teams, after Thursday's merge | platform |
| `mobile-prod` | live | platform, manual approval |

Right now `mobile-dev-team1` = CI builds a debug APK and attaches it to the Actions run. Install from there. Not a hosted environment.

---

## The week

| Day | Us |
|---|---|
| Mon–Wed | features → PRs into `b1:main` |
| **Thu** | **lead raises `b1:main → b:stage`.** CI must be green or we skip the week |
| **Thu** | review cross-team; flag any change to the institution API shape or the auth handoff |
| **Fri** | conflict resolution, get `stage` green |
| Sat–Sun | freeze — nothing merged upstream |

If our fork `main` is red Thursday morning, we don't raise the PR. A broken `stage` blocks all 20 people.

---

## Closed by wokay's source of truth

Five of the seven items previously listed here are answered. Don't carry them into the gate.

| Was open | Answer |
|---|---|
| **Do we own any backend module?** | **No.** *"team1 and t4targaryen have NO backend module. React Native only."* `a1` and `c1` don't exist, and neither do their environments. |
| **Institution schema fields** | Published in full. Both endpoints and every field are above, under CODEOWNERS. |
| **Auth-type routing contract** | **Dead as a question.** Sign-in is always SAML; there is no auth type. We pass `signIn.idpHint` and `institutionId` through. |
| **Search/filter for CAP-2** | **Backend, server-side, paged** — `?q=&country=&page=&size=`. Not a client-side filter. Same answer for catalogue search, which is also wokay's and lands Week 4. |
| **Individual / B2C bypass** | **RETAINED, details to come (11 Aug).** wokay recommended cutting individual accounts at their gate (decision 7); that recommendation is not being taken and they will supply the B2C details later. So `subscribe` and `Session.type` stay, and screen 03's second option is not settled. One skip-institution path we do have is the anonymous open-access feed, `GET /opds/v1/public/catalogue`, which needs no token and no institution. CAP-3 needs no "no institution" path, but the home screen does need a *browse free content* entry. |

## Open for team1

Carry these into the Week 1 Alignment & Contracts Gate:

1. **Who is team1's lead?** They fork, own the Thursday PR, own fork secrets.
2. **Selection persistence semantics** — device-local only, or synced to the account? Changes whether CAP-3 needs a backend call. (Note: since we own no backend, "synced" would mean asking flambeau for it.)
3. **Does CAP-2 own the *anonymous* entry point too?** The public feed needs no institution, so there's a path into the app that bypasses our screens entirely. Worth confirming it's ours to present.

## New, and dated

Not gate items — these have deadlines inside Week 1:

1. **wokay need four things from us by end of Week 1**, and their default if we're silent is to choose for us. On subject filters, their default deletes the Browse-by-Subject row from screen 01. Answers are in the Week 1 Foundation Spec, §P0-8. **Say yes to subject facets.**
2. **Chase wokay's Week 1 fixtures on Monday morning** — OpenAPI file, three OPDS samples, mock endpoints. They close the largest risk in our plan.
3. **The signed design spec's access matrix was wrong** on Elite and Open Access; it's corrected to v0.3 and needs leadership ratification. Bundle it with the L-2 escalation.
4. **My Library / screen 08 ownership is contested** — wokay's §03 says it's ours, our plan says t4targaryen's. Resolve by Friday or escalate.
