# T&F Reader — GitHub Structure, Branching & CI/CD

**Status:** draft standard for the 8-week build
**Applies to:** all 3 repos, all 4 teams
**Owner:** platform / repo admins
**Companion doc:** [TEAM1_QUICKSTART.md](TEAM1_QUICKSTART.md)

Open items that still need a decision are collected in [§12 Open Questions](#12-open-questions). Anything marked **[ASSUMED]** is a default I picked so the doc is usable — confirm or overwrite it.

---

## 1. Clients & Repos

Two client surfaces, three repos.

| Client | Repo | Ref | Stack |
|---|---|---|---|
| Phone | `tf_reader_mobile_app` | **b** | React Native |
| Web admin | `tf_reader_admin` | **c** | Web (React) **[ASSUMED]** |
| — (serves both) | `tf_reader_backend` | **a** | Spring Boot modular monolith + MongoDB + Redis |

`a`, `b`, `c` are shorthand used throughout this doc. They are not real names.

### Upstream branches

Every upstream repo has exactly two long-lived branches:

```
tf_reader_backend    (a)  →  main, stage
tf_reader_mobile_app (b)  →  main, stage
tf_reader_admin      (c)  →  main, stage
```

- `main` = **production**. Live. Protected. Never pushed to directly.
- `stage` = **staging / UAT / pre-prod**. Protected. Integration point for all four teams.

No `develop` branch upstream. Integration happens on `stage`.

---

## 2. Fork & Clone Topology

Each team's **lead forks** the upstream repo. Everyone else on the team **clones the team fork** — nobody clones upstream directly.

```
                    tf_reader_backend (a)          ← upstream, org-owned
                    ├── main   (prod)
                    └── stage  (UAT)
                             ▲
        ┌────────────┬───────┴──────┬──────────────┐
        │            │              │              │
       a1           a2             a3             a4          ← team forks
   (team1)       (wokay)      (flambeau)   (t4targaryen)
   main = that team's dev line
        │
   ┌────┼────┬────┐
  p1   p2   p3   p4                                            ← 4 devs clone the fork
  (+ lead = 5 per team)
```

The same 4-fork pattern exists for `b` (`b1..b4`) and `c` (`c1..c4`). **12 forks total.**

> Team-number mapping (`a1` = team1, `a2` = wokay, `a3` = flambeau, `a4` = t4targaryen) is illustrative — fix the real mapping in [§12](#12-open-questions).

### What `main` means in a fork

**In a fork, `main` is the team's dev integration branch — not production.** This is the single most confusing part of this setup, so state it in every fork's README:

| Branch | In upstream `a` | In fork `a1` |
|---|---|---|
| `main` | production, live | **team dev line**, deploys to team dev env |
| `stage` | UAT | unused — do not create/track it in forks |
| `feature/*` | never | where individual devs work |

### Remotes on every dev's clone

```bash
git clone git@github.com:<org-or-lead>/tf_reader_backend.git   # this is a1
cd tf_reader_backend
git remote add upstream git@github.com:<org>/tf_reader_backend.git
git remote -v
# origin    a1  (fetch/push)   ← your team fork
# upstream  a   (fetch)        ← the real repo, read-only for you
```

Only the **team lead** ever pushes to `upstream`, and only via PR.

---

## 3. Branch Naming

```
feature/CAP-<n>-<short-slug>      feature/CAP-2-institution-list
fix/CAP-<n>-<short-slug>          fix/CAP-3-selection-persist
chore/<short-slug>                chore/ci-cache-gradle
spike/<short-slug>                spike/opds-parser
release/<yyyy-mm-dd>              release/2026-08-13   (stage → main only)
hotfix/<short-slug>               hotfix/jwt-expiry    (branches off main)
```

Lowercase, hyphens, no personal names in branch names.

---

## 4. Day-to-Day Working Rules

The loop for one feature, start to merge.

### 4.1 Start of every session — sync down

```bash
git checkout main
git pull --ff-only origin main       # team fork main
git fetch upstream
```

### 4.2 Create the feature branch

```bash
git checkout -b feature/CAP-2-institution-list
```

### 4.3 While working — rebase, never merge

Pull fork `main` **every session** and rebase your feature onto it:

```bash
git checkout main && git pull --ff-only origin main
git checkout feature/CAP-2-institution-list
git rebase main
```

**Every few days**, also pull upstream so your team fork doesn't drift from the rest of the org:

```bash
# team lead (or anyone, on the fork's main)
git checkout main
git fetch upstream
git rebase upstream/stage        # forks track upstream stage, not upstream main
git push origin main
```

Rationale: forks integrate against `stage`, because `stage` is where the other three teams' work lands. Rebasing forks onto upstream `main` would leave you a week behind.

**Rebase, don't merge.** History stays linear; the weekly upstream PR stays reviewable.

### 4.4 Push cadence

**Push your feature branch to the fork every 2–3 days minimum**, even if incomplete. This is a hackathon-speed build — a branch that lives 5 days unpushed is a conflict grenade. Prefix the PR `Draft:` if it isn't ready.

### 4.5 Pre-PR checklist

Run all of this **before** raising the PR:

- [ ] **Rebased** on latest fork `main`
- [ ] **Tests** pass locally (unit + whatever integration exists)
- [ ] **`git diff` reviewed in VS Code** — read your own diff hunk by hunk. Catch stray debug logs, commented-out blocks, committed `.env`, accidental formatting churn
- [ ] **README** for the feature, if the feature warrants one (new module, new env var, new endpoint, non-obvious setup)
- [ ] **Log file / changelog** updated if you're keeping one
- [ ] **AI pass** — optimization, regression risk, code quality
- [ ] **Security review** — in Claude Code run `/security-review` (or `/code-review` with a security focus). Do this on the diff before every PR, not just for auth/crypto work
- [ ] No secrets, no tokens, no real institution/user data in the diff

### 4.6 Raise the PR → team fork

```
feature/CAP-2-institution-list  →  a1:main
```

Not to upstream. Never to upstream from a feature branch.

PR must have: what changed, why, how to test, screenshots for UI, linked CAP ID.

### 4.7 Review & merge

- Reviewers assigned automatically via `CODEOWNERS` (see [§5](#5-code-review-setup))
- Minimum **1 approval** from someone who isn't the author
- CI green (required)
- **Squash merge** into fork `main` — keeps the weekly upstream PR readable
- Delete the feature branch after merge

---

## 5. Code Review Setup

Three files, at the root of **every repo** (upstream and forks):

### 5.1 `.github/CODEOWNERS` — the enforcement

This is the file GitHub actually reads. It auto-requests reviews and (with branch protection) blocks merge until the owner approves. `CODE_REVIEW.md` and `CONTRIBUTORS.md` are documentation; **CODEOWNERS is the mechanism.**

```gitignore
# .github/CODEOWNERS
# Syntax: <file pattern>  <@reviewers>
# Last matching pattern wins.

# Fallback — team lead reviews anything unclaimed
*                                   @team1-lead

# --- Capability ownership -------------------------------------------------
/src/main/java/**/institution/**    @team1-lead @dev-p1 @dev-p2   # CAP-2
/src/main/java/**/selection/**      @team1-lead @dev-p3           # CAP-3
/src/main/java/**/auth/**           @flambeau-lead                # CAP-6
/src/main/java/**/catalogue/**      @wokay-lead                   # CAP-5

# --- Cross-cutting: notify + require review -------------------------------
/.github/workflows/**               @platform-owner
/.ci/**                             @platform-owner
/CODEOWNERS                         @platform-owner
/build.gradle*  /pom.xml            @platform-owner @backend-leads
/package.json   /package-lock.json  @platform-owner
/src/main/resources/application*.yml @platform-owner
/**/db/migration/**                 @wokay-lead @platform-owner
/**/*.env.example                   @platform-owner
```

Rules of thumb:
- Anything touching **build, CI, config, secrets, or schema** → platform owner is a required reviewer
- Anything touching a **capability module** → that capability's owning team
- Shared contract files (API DTOs, the institution schema) → **both** producing and consuming team

### 5.2 `CODE_REVIEW.md` — the human contract

Documents the *policy* CODEOWNERS enforces:

- Which file patterns map to which reviewers, and **why** (with the CAP ID)
- Which changes require **notification** vs. **blocking approval**
- SLA: reviews turned around within **1 working day**; Thursday PRs reviewed same day
- What a reviewer is expected to check (correctness, tests, security, contract compatibility)
- Escalation path when a reviewer is unavailable

### 5.3 `CONTRIBUTORS.md` — who's who

- Every contributor: name, GitHub handle, team, capability, timezone
- Which files/modules each person authored (so CODEOWNERS stays honest as people move)
- Team leads, and who has fork-admin / upstream-push rights

Keep all three in sync. When someone takes over a module, update **CODEOWNERS and CONTRIBUTORS.md in the same PR**.

---

## 6. Weekly Cadence

Fixed rhythm, so integration pain is bounded to two days a week.

| Day | What happens | Who |
|---|---|---|
| **Mon–Wed** | Feature work. Push to fork every 2–3 days. PRs into fork `main`. | All devs |
| **Thu** | **Team lead raises PR: `a1:main → a:stage`.** One per repo, per team. Up to 12 PRs org-wide. | Team leads |
| **Thu** | Cross-team review of those PRs. Contract changes flagged loudly. | Leads + affected owners |
| **Fri** | **Conflict resolution day.** Merge the stage PRs, resolve conflicts, get `stage` green. | Leads + platform |
| **Fri EOD** | `stage` deployed and smoke-tested. Freeze. | Platform |
| **Sat–Sun** | **`stage` and `main` must be stable.** No merges upstream. Emergencies via `hotfix/*` + lead approval only. | — |

### The weekly upstream PR

```
a1:main  →  a:stage      (Thursday, one per team per repo)
```

Requirements: CI green, no conflicts with `stage` at time of raise, description lists every CAP touched and every contract change.

If a team's fork `main` is red on Thursday, **that team does not raise a PR that week.** A broken stage blocks 20 people.

### Promotion to production

`stage → main` is **not** part of the weekly loop by default. It's a deliberate release:

```
release/2026-08-13  →  a:main
```

Cut from `stage` once it has been green and smoke-tested. Requires: platform owner + 1 team lead approval, manual approval gate on the prod deploy job, and a tag `v<yyyy.mm.dd>` on merge. **[ASSUMED]** — confirm the release cadence in [§12](#12-open-questions).

---

## 7. Environments

**6 per repo × 3 repos = 18 environments.**

| # | Env | Source branch | Repo hosting it | Deploys on |
|---|---|---|---|---|
| 1 | **prod** | `main` | upstream `a` / `b` / `c` | merge to `main` + manual approval |
| 2 | **stage** (UAT / pre-prod) | `stage` | upstream `a` / `b` / `c` | merge to `stage`, automatic |
| 3 | **dev-team1** | `main` | fork `a1` / `b1` / `c1` | merge to fork `main`, automatic |
| 4 | **dev-wokay** | `main` | fork `a2` / `b2` / `c2` | merge to fork `main`, automatic |
| 5 | **dev-flambeau** | `main` | fork `a3` / `b3` / `c3` | merge to fork `main`, automatic |
| 6 | **dev-t4targaryen** | `main` | fork `a4` / `b4` / `c4` | merge to fork `main`, automatic |

Plus **personal dev** — every developer's local machine (docker-compose for Mongo + Redis + backend). Not a deployed environment, not in the 18, no pipeline.

### Naming convention

```
<repo-short>-<env>
backend-prod   backend-stage   backend-dev-team1   ...
mobile-prod    mobile-stage    mobile-dev-team1    ...
admin-prod     admin-stage     admin-dev-team1     ...
```

Use the same string as the GitHub Environment name, the deploy target, and the config namespace. One name, everywhere.

### Data isolation

Each of the 18 gets its **own MongoDB database and Redis namespace**. Never point a dev env at stage data. Seed dev envs from a checked-in fixture set (`/.ci/seed/`), not from a prod dump.

### Important: forks run their own CI

GitHub Actions in a fork run under the **fork's** Actions billing and use the **fork's** secrets — upstream secrets are never available to a fork. So each of the 12 forks needs its own secret set for its dev env. See [§10](#10-secrets--configuration).

---

## 8. CI/CD Pipeline Design

### 8.1 The deploy seam

The cloud target is not decided yet, so **the pipelines are provider-agnostic**. Every repo has one script:

```
.ci/deploy.sh
```

The workflows always call it the same way:

```bash
.ci/deploy.sh --env "$ENV_NAME" --image "$IMAGE_REF" --ref "$GITHUB_SHA"
```

The workflow contract is fixed and doesn't change when the cloud is chosen. **Only `deploy.sh` changes.** Today it can be a stub that echoes and exits 0; when AWS/Azure/whatever is picked, one file per repo gets implemented and all 18 environments start deploying.

Artifacts go to **GitHub Container Registry** (`ghcr.io`) — works everywhere, no cloud account needed, and any cloud can pull from it later.

```
ghcr.io/<org>/tf-reader-backend:<sha>
ghcr.io/<org>/tf-reader-backend:<env>     # moving tag per environment
```

### 8.2 Workflows per repo

| File | Trigger | Runs |
|---|---|---|
| `ci.yml` | PR to any branch; push to `main`/`stage` | lint → test → build → security |
| `deploy-dev.yml` | push to `main` **in a fork** | build image → deploy `dev-<team>` |
| `deploy-stage.yml` | push to `stage` **upstream** | build image → deploy `stage` → smoke |
| `deploy-prod.yml` | push to `main` **upstream**; `workflow_dispatch` | build image → **manual approval** → deploy `prod` → smoke → tag |
| `_build-deploy.yml` | `workflow_call` | reusable: the actual build + deploy body |
| `pr-hygiene.yml` | PR opened/synced | conflict check, PR title/format, size warning |

The three deploy workflows are thin wrappers over `_build-deploy.yml` so the logic exists once.

### 8.3 `ci.yml` — backend (Spring Boot)

```yaml
# .github/workflows/ci.yml
name: CI

on:
  pull_request:
  push:
    branches: [main, stage]

concurrency:
  group: ci-${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

permissions:
  contents: read
  security-events: write

jobs:
  build-test:
    runs-on: ubuntu-latest
    services:
      mongodb:
        image: mongo:7
        ports: ['27017:27017']
      redis:
        image: redis:7
        ports: ['6379:6379']
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-java@v4
        with:
          java-version: '21'
          distribution: temurin
          cache: gradle          # switch to `maven` if the repo uses Maven

      - name: Lint / format check
        run: ./gradlew spotlessCheck

      - name: Unit + integration tests
        run: ./gradlew test
        env:
          SPRING_DATA_MONGODB_URI: mongodb://localhost:27017/tfreader_ci
          SPRING_DATA_REDIS_HOST: localhost

      - name: Build
        run: ./gradlew bootJar -x test

      - name: Publish test report
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: test-results
          path: build/reports/tests/
          retention-days: 7

  security:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      security-events: write
    steps:
      - uses: actions/checkout@v4
        with: { fetch-depth: 0 }

      - name: Secret scan
        uses: gitleaks/gitleaks-action@v2
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

      - name: Dependency review
        if: github.event_name == 'pull_request'
        uses: actions/dependency-review-action@v4
        with:
          fail-on-severity: high

      - name: CodeQL init
        uses: github/codeql-action/init@v3
        with: { languages: java }
      - name: CodeQL autobuild
        uses: github/codeql-action/autobuild@v3
      - name: CodeQL analyze
        uses: github/codeql-action/analyze@v3
```

For **admin (`c`)** the same file swaps the Java block for:

```yaml
      - uses: actions/setup-node@v4
        with: { node-version: '20', cache: npm }
      - run: npm ci
      - run: npm run lint
      - run: npm run test -- --coverage
      - run: npm run build
```

…and CodeQL `languages: javascript-typescript`. No service containers needed.

### 8.4 `_build-deploy.yml` — reusable, provider-agnostic

```yaml
# .github/workflows/_build-deploy.yml
name: Build & Deploy

on:
  workflow_call:
    inputs:
      env_name:
        required: true
        type: string          # backend-prod | backend-stage | backend-dev-team1 | ...
      gh_environment:
        required: true
        type: string          # GitHub Environment (carries approval rules + secrets)
      smoke:
        type: boolean
        default: true

permissions:
  contents: read
  packages: write

jobs:
  build:
    runs-on: ubuntu-latest
    outputs:
      image: ${{ steps.meta.outputs.image }}
    steps:
      - uses: actions/checkout@v4

      - id: meta
        run: |
          IMG="ghcr.io/${GITHUB_REPOSITORY,,}:${GITHUB_SHA::12}"
          echo "image=$IMG" >> "$GITHUB_OUTPUT"

      - uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - uses: docker/build-push-action@v6
        with:
          push: true
          tags: |
            ${{ steps.meta.outputs.image }}
            ghcr.io/${{ github.repository }}:${{ inputs.env_name }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: ${{ inputs.gh_environment }}     # approval gate + env secrets live here
      url: ${{ steps.deploy.outputs.url }}
    concurrency:
      group: deploy-${{ inputs.env_name }}   # never two deploys to one env at once
      cancel-in-progress: false
    steps:
      - uses: actions/checkout@v4

      - id: deploy
        name: Deploy
        run: .ci/deploy.sh --env "${{ inputs.env_name }}" \
                           --image "${{ needs.build.outputs.image }}" \
                           --ref  "$GITHUB_SHA"
        env:
          DEPLOY_TOKEN:  ${{ secrets.DEPLOY_TOKEN }}
          MONGODB_URI:   ${{ secrets.MONGODB_URI }}
          REDIS_URL:     ${{ secrets.REDIS_URL }}

      - name: Smoke test
        if: inputs.smoke
        run: .ci/smoke.sh --url "${{ steps.deploy.outputs.url }}"
```

### 8.5 The three thin wrappers

```yaml
# .github/workflows/deploy-dev.yml   — lives in FORKS
name: Deploy dev
on:
  push:
    branches: [main]
jobs:
  dev:
    # guard: never runs if this file ends up in upstream
    if: github.repository != '<org>/tf_reader_backend'
    uses: ./.github/workflows/_build-deploy.yml
    with:
      env_name: backend-dev-${{ vars.TEAM_SLUG }}   # repo variable: team1 | wokay | ...
      gh_environment: dev
    secrets: inherit
```

```yaml
# .github/workflows/deploy-stage.yml — lives UPSTREAM
name: Deploy stage
on:
  push:
    branches: [stage]
jobs:
  stage:
    if: github.repository == '<org>/tf_reader_backend'
    uses: ./.github/workflows/_build-deploy.yml
    with:
      env_name: backend-stage
      gh_environment: stage
    secrets: inherit
```

```yaml
# .github/workflows/deploy-prod.yml  — lives UPSTREAM
name: Deploy prod
on:
  push:
    branches: [main]
  workflow_dispatch:
jobs:
  prod:
    if: github.repository == '<org>/tf_reader_backend'
    uses: ./.github/workflows/_build-deploy.yml
    with:
      env_name: backend-prod
      gh_environment: production      # ← required reviewers configured here
    secrets: inherit

  tag:
    needs: prod
    runs-on: ubuntu-latest
    permissions: { contents: write }
    steps:
      - uses: actions/checkout@v4
      - run: |
          TAG="v$(date +%Y.%m.%d)-${GITHUB_SHA::7}"
          git tag "$TAG" && git push origin "$TAG"
```

`TEAM_SLUG` is a **repository variable** set once in each fork (Settings → Variables → Actions). It's the only per-fork difference — the workflow files stay identical across all 12 forks, so they merge cleanly.

### 8.6 Mobile app (`b`) — deploy means something different

**[ASSUMED — needs a decision, see [§12](#12-open-questions)]** You said "no clue" on this, so here's the lowest-friction default for an 8-week build:

| Env | What "deploy" means |
|---|---|
| `mobile-dev-<team>` | CI builds a **debug APK**, uploads as a workflow artifact (7-day retention). Team installs from the Actions run page. |
| `mobile-stage` | CI builds a **release APK + iOS simulator build**, uploads as artifact, 30-day retention. QA/UAT installs manually. |
| `mobile-prod` | CI builds a **signed release APK/AAB**, attaches it to a **GitHub Release** on the version tag. No app-store submission during the build phase. |

This needs zero signing certs for dev/stage and zero cloud/store accounts. When store distribution is needed, `deploy.sh` for `b` gains a Firebase App Distribution or TestFlight step — the workflow doesn't change.

```yaml
# .github/workflows/ci.yml (mobile) — build job
  android:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '20', cache: npm }
      - run: npm ci
      - run: npm run lint && npm test -- --ci
      - uses: actions/setup-java@v4
        with: { java-version: '17', distribution: temurin }
      - run: ./gradlew assembleRelease
        working-directory: android
      - uses: actions/upload-artifact@v4
        with:
          name: app-${{ github.sha }}.apk
          path: android/app/build/outputs/apk/release/*.apk
          retention-days: 30

  ios:
    runs-on: macos-latest        # ⚠ 10× minute cost — see §12
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '20', cache: npm }
      - run: npm ci
      - run: cd ios && pod install
      - run: xcodebuild -workspace ios/TFReader.xcworkspace \
                        -scheme TFReader -sdk iphonesimulator build
```

> **Cost warning:** macOS runners bill at 10× Linux minutes. Run the `ios` job only on PRs to `stage`/`main` and on a nightly schedule, not on every fork push, or four teams will burn the org's Actions quota in week 2.

### 8.7 `pr-hygiene.yml`

```yaml
name: PR hygiene
on:
  pull_request:
    types: [opened, synchronize, reopened, edited]

jobs:
  checks:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with: { fetch-depth: 0 }

      - name: PR title must reference a CAP
        run: |
          echo "${{ github.event.pull_request.title }}" \
            | grep -Eiq '(CAP-[0-9]+|chore|hotfix)' \
            || { echo "::error::PR title must reference CAP-<n>, chore, or hotfix"; exit 1; }

      - name: Warn on large PRs
        run: |
          N=$(git diff --shortstat origin/${{ github.base_ref }}...HEAD | awk '{print $4+$6}')
          [ "${N:-0}" -lt 800 ] || echo "::warning::${N} lines changed — consider splitting"

      - name: Block merge commits (rebase policy)
        run: |
          git log --merges origin/${{ github.base_ref }}..HEAD --oneline | grep . \
            && { echo "::error::Merge commits found — rebase instead"; exit 1; } || true
```

---

## 9. Branch Protection

### Upstream `a` / `b` / `c`

**`main`:**
- No direct pushes, no force-push, no deletion
- PR required; source must be `stage` or `hotfix/*`
- **2 approvals**, one must be a CODEOWNER
- Required status checks: `build-test`, `security`, `pr-hygiene`
- Dismiss stale approvals on new commits
- Require branches up to date before merge
- Require linear history
- Include administrators

**`stage`:**
- No direct pushes, no force-push
- PR required
- **1 approval** from a CODEOWNER
- Same required checks
- Require linear history

### Team forks `a1..a4` etc.

**`main`:**
- No direct pushes — PR required even for the lead
- **1 approval**, not the author
- Required checks: `build-test`
- Require linear history (enforces the rebase policy)

Loose enough not to block a 5-person team, strict enough that fork `main` stays deployable.

---

## 10. Secrets & Configuration

### Never in git

`.env`, keystores, `google-services.json`, `GoogleService-Info.plist`, signing keys, connection strings. Every repo commits a `.env.example` with **keys only, no values**, and gitleaks in CI catches slips.

### Where secrets live

| Scope | Where | Contains |
|---|---|---|
| Org | GitHub org secrets | shared registry creds |
| Upstream repo → `stage` | GitHub Environment `stage` | stage Mongo/Redis URIs, deploy token |
| Upstream repo → `production` | GitHub Environment `production` | prod creds + **required reviewers** |
| Each fork → `dev` | GitHub Environment `dev` **in that fork** | that team's dev Mongo/Redis, deploy token |

**Each of the 12 forks configures its own `dev` environment secrets.** Upstream secrets are not inherited by forks — this is a GitHub security boundary, not a misconfiguration. Team leads own this setup in week 1.

Prod secrets are known to platform owners only. No team lead needs them.

### Config precedence

`defaults in repo` → `env-specific config file` → `environment variables from CI secrets`. Nothing environment-specific gets hardcoded; if it differs between the 18 environments, it's an env var.

---

## 11. Hotfixes

Only path that bypasses the weekly rhythm.

```bash
git fetch upstream
git checkout -b hotfix/jwt-expiry upstream/main
# minimal fix + test
```

1. PR `hotfix/*` → `a:main`. Requires platform owner + 1 lead. CI must be green.
2. Deploy to prod via the manual approval gate.
3. **Immediately** cherry-pick or re-PR the same fix into `stage`, then let it flow to forks. A hotfix that only exists on `main` gets reverted by the next release.

Weekend hotfixes: lead approval + notify the channel. Otherwise the weekend freeze holds.

---

## 12. Open Questions

Needs a decision before this doc is final. Ordered by how much they block.

| # | Question | Blocks | Suggested default |
|---|---|---|---|
| 1 | **Cloud target?** AWS / Azure / self-hosted / docker-compose on a shared box | All 18 deploy jobs — `.ci/deploy.sh` is a stub until this lands | Pick in week 1; pipelines already work around it |
| 2 | **Mobile "deploy" definition** — is artifact-only ([§8.6](#86-mobile-app-b--deploy-means-something-different)) acceptable, or do you need Firebase App Distribution / TestFlight? | 6 of the 18 envs | Artifact-only through week 5; revisit for UAT |
| 3 | **Who is the platform / CI owner?** Someone must own `.github/workflows`, secrets, and the 18 environments across 3 repos | Everything | One named person, not a committee |
| 4 | **Team → fork-number mapping.** Which team is `a1`, `a2`, `a3`, `a4`? | CODEOWNERS, `TEAM_SLUG` vars | — |
| 5 | **Team1's lead** (who forks and owns the Thursday PR)? Team1 is Akriti, Khushi, Prayas, Moktik, Keshav — you said 4 clone + lead forks = 5, which fits | Fork creation | — |
| 6 | **`stage → main` promotion cadence.** Weekly? End of each 2-week block? Only at week 8? I assumed a deliberate release PR, not automatic | Prod pipeline trigger | Manual release PR |
| 7 | **Build tool for backend** — Gradle or Maven? CI is written for Gradle | `ci.yml` for `a` | — |
| 8 | **Does team1 own any backend module?** Prior notes flag this as unresolved. If team1 is frontend-only, team1 may not need `a1` at all — that's 6 fewer environments to run | Whether a1/c1 exist for team1 | — |
| 9 | **Admin client stack** — React? Something else? | `ci.yml` for `c` | — |
| 10 | **Actions minutes budget.** 4 teams × 3 repos × every push, plus macOS runners, adds up fast. Is there a quota, or self-hosted runners? | `ios` job frequency | Restrict macOS to `stage`/`main` PRs |
| 11 | **Are dev envs really 4 per repo?** 12 always-on dev deployments is a lot of infra for an 8-week build. Mobile especially — a "mobile dev environment" is arguably just the APK artifact | Infra cost | Consider backend-only dev envs |
| 12 | **Shared API contract** — where does the OpenAPI spec / DTO package live so mobile and admin don't drift from backend? Not covered by this doc | Cross-team integration | Publish spec from `a` on every `stage` deploy |

---

## Appendix A — Repo File Checklist

Every repo, upstream and fork:

```
.github/
  workflows/
    ci.yml
    pr-hygiene.yml
    _build-deploy.yml
    deploy-dev.yml        (forks)
    deploy-stage.yml      (upstream)
    deploy-prod.yml       (upstream)
  CODEOWNERS
  PULL_REQUEST_TEMPLATE.md
.ci/
  deploy.sh               ← the provider seam
  smoke.sh
  seed/                   ← dev fixtures
CODE_REVIEW.md
CONTRIBUTORS.md
README.md                 ← must state: "in this fork, main = team dev line"
.env.example
.gitignore
```

## Appendix B — Command Cheatsheet

```bash
# start of session
git checkout main && git pull --ff-only origin main

# new feature
git checkout -b feature/CAP-2-institution-list

# keep current (run daily)
git checkout main && git pull --ff-only origin main
git checkout - && git rebase main

# lead: sync fork with upstream (every 2-3 days)
git checkout main && git fetch upstream && git rebase upstream/stage && git push origin main

# review your own diff before PR
git diff main...HEAD           # then read it in VS Code

# push feature
git push -u origin feature/CAP-2-institution-list

# lead: Thursday upstream PR
gh pr create --repo <org>/tf_reader_backend \
             --base stage --head <team-org>:main \
             --title "team1 → stage: CAP-2, CAP-3 (week N)"
```
