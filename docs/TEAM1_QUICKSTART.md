# Team1 Quickstart — Discovery & Selection

Short version of [GITHUB_WORKFLOW_AND_CICD.md](GITHUB_WORKFLOW_AND_CICD.md), written from team1's seat. When the two disagree, the master doc wins.

**Team1 owns:** CAP-2 (institution listing) + CAP-3 (institute selection)
**People:** Akriti Khetan, Khushi S Shukla, Prayas Yadav, Moktik, Keshav Sharma — lead **TBD** (lead forks, other four clone)

---

## Our repos

| Ref | Fork we work in | Upstream | Do we own backend code here? |
|---|---|---|---|
| `b1` | `tf_reader_mobile_app` fork | `b` | **Yes** — CAP-2/CAP-3 UI is React Native |
| `a1` | `tf_reader_backend` fork | `a` | **Unresolved** — see [Open](#open-for-team1) |
| `c1` | `tf_reader_admin` fork | `c` | Probably not — admin is wokay's |

Team1's skill list is the only one of the four that doesn't name Spring Boot/MongoDB, so `b1` is the certain one. Confirm `a1`/`c1` at the Week 1 contracts gate before the lead forks all three.

---

## One-time setup

```bash
# lead only: fork b (and a/c if we own them) via GitHub UI → this is b1

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

Two files deserve extra care because they're **cross-team contracts**, not just our code:
- `api/institutions.ts` — shaped by **wokay's** institution schema
- `navigation/authRouting.ts` — hands `institutionId` into **flambeau's** sign-in flow

Add the relevant other-team lead as a reviewer on any PR touching those.

---

## Our environments

| Env | What it is | Who deploys |
|---|---|---|
| personal dev | your laptop, docker-compose | you |
| `mobile-dev-team1` | auto-deploys on merge to `b1:main` | CI |
| `backend-dev-team1` | only if we own `a1` | CI |
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

## Open for team1

Carry these into the Week 1 Alignment & Contracts Gate:

1. **Who is team1's lead?** They fork, own the Thursday PR, own fork secrets.
2. **Do we own any backend module?** Determines whether `a1` exists at all — and 6 environments with it.
3. **Institution schema fields** — need wokay's contract before `api/institutions.ts` is real.
4. **Auth-type routing contract** — what shape does flambeau expect `institutionId` in?
5. **Search/filter requirements for CAP-2** — client-side over a fetched list, or a backend search endpoint? Changes who owns the work.
6. **Selection persistence semantics** — device-local only, or synced to the account? Changes whether CAP-3 needs a backend call.
7. **Individual / B2C bypass** — can a user skip institution selection? If yes, CAP-3 needs a "no institution" path.
