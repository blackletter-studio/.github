# M0 — Scaffold & Infrastructure Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Stand up Black Letter studio's infrastructure — Cloudflare DNS active on `blackletter.studio`, seven GitHub repos created with shared CI patterns and brand tokens committed, a placeholder marketing site live at `https://blackletter.studio`, and a sideloadable Word add-in skeleton running in local Word. Zero product logic; pure scaffolding that every future milestone builds on.

**Architecture:** Polyrepo under `github.com/blackletter-studio`. DNS via Cloudflare (WAF + TLS 1.3 + HSTS). Site on Netlify (Astro + TypeScript + Tailwind). Add-in on TypeScript + React + Vite + Fluent UI v9 + Office.js. Brand tokens in a private `brand` repo consumed as a git submodule. All CI uses pnpm + Node 20 LTS + GitHub Actions with OIDC to cloud providers.

**Tech Stack:** Astro 4, Tailwind 3, React 18, Vite 5, Fluent UI React v9, Office.js WordApi 1.7+, Wrangler 3, pnpm 9, Node 20 LTS, Terraform 1.6+, Fraunces + IBM Plex Sans (self-hosted woff2), Porkbun (registrar), Cloudflare (DNS + WAF + Workers), Netlify (site hosting).

**Prerequisites (confirm before starting):**

- GitHub org `blackletter-studio` exists and user can create repos in it
- Domain `blackletter.studio` registered at Porkbun, user can change nameservers
- Cloudflare account exists; user can add new zones
- Netlify account exists; user can create new sites
- Local machine has: `gh` CLI authenticated, `pnpm`, `node 20+`, `git`, Microsoft Word installed for sideload testing
- User's terminal at `~/GitRepos/` for creating local repo clones

**Repo summary (what gets created in this plan):**

| Local path                                                    | GitHub URL                                                    | Visibility | Purpose                                                                         |
| ------------------------------------------------------------- | ------------------------------------------------------------- | ---------- | ------------------------------------------------------------------------------- |
| `~/GitRepos/blackletter-studio/.github-org`                   | `github.com/blackletter-studio/.github`                       | Public     | Org-wide defaults (PR/issue templates, CODEOWNERS defaults, reusable workflows) |
| `~/GitRepos/blackletter-studio/brand`                         | `github.com/blackletter-studio/brand`                         | Private    | Color tokens, fonts, logo SVGs                                                  |
| `~/GitRepos/blackletter-studio/site`                          | `github.com/blackletter-studio/site`                          | Public     | Astro marketing site + feedback Cloudflare Worker                               |
| `~/GitRepos/blackletter-studio/fair-copy`                     | `github.com/blackletter-studio/fair-copy`                     | Public     | Word add-in (Fair Copy + Proofmark)                                             |
| `~/GitRepos/blackletter-studio/legal-augmentation-dictionary` | `github.com/blackletter-studio/legal-augmentation-dictionary` | Public     | Hunspell dictionary seed                                                        |
| `~/GitRepos/blackletter-studio/feedback-intake`               | `github.com/blackletter-studio/feedback-intake`               | Public     | Anonymous-feedback intake tracker                                               |
| `~/GitRepos/blackletter-studio/infrastructure`                | `github.com/blackletter-studio/infrastructure`                | Private    | Terraform for Cloudflare DNS + Workers routes                                   |

Note: The existing `~/GitRepos/blackletter-studio/` planning directory (containing this plan and the design spec) migrates into the `.github` repo during Task 13. Keep the planning repo on branch `docs/initial-design-spec` until migration.

---

## Task 1 — Verify prerequisites

**Files:** none (verification only)

- [ ] **Step 1: Confirm GitHub CLI is authenticated to the correct account**

```bash
gh auth status
```

Expected: logged in as `twitchyvr`; required scopes include `repo`, `admin:org`, `workflow`.

- [ ] **Step 2: Confirm org membership and repo-creation permission**

```bash
gh api orgs/blackletter-studio --jq '.login, .plan.name'
gh api user/memberships/orgs/blackletter-studio --jq '.role'
```

Expected: returns `blackletter-studio`, `free`, `admin`.

- [ ] **Step 3: Confirm local tool versions**

```bash
node --version    # v20.x or v22.x
pnpm --version    # 9.x
git --version     # 2.40+
terraform --version  # 1.6+ (install via `brew install terraform` if missing)
wrangler --version   # 3.x (install via `pnpm dlx wrangler --version` if not global)
```

- [ ] **Step 4: Confirm Word is installed and the `Word` account is signed in**

```bash
ls /Applications/Microsoft\ Word.app  # macOS
```

Open Word once manually if not run recently; confirm signed-in Microsoft account appears under File → Account.

- [ ] **Step 5: Confirm Porkbun API credentials (optional, for Terraform-managed DNS-to-registrar workflow in a later milestone)**

Log into Porkbun; verify API access is enabled for `blackletter.studio`. Note the API key + secret in 1Password under `Black Letter / Porkbun`. (We do not use them yet — only nameservers change in Task 2.)

---

## Task 2 — Add `blackletter.studio` to Cloudflare and swap nameservers at Porkbun

**Files:** none (manual UI work + `dig` verification)

- [ ] **Step 1: Add the zone to Cloudflare**

In the Cloudflare dashboard: `+ Add a site` → enter `blackletter.studio` → select Free plan.

- [ ] **Step 2: Copy the two assigned Cloudflare nameservers**

Cloudflare will display two names like `colin.ns.cloudflare.com` and `lena.ns.cloudflare.com` (the names vary per zone). Copy both.

- [ ] **Step 3: Update nameservers at Porkbun**

Porkbun dashboard → Domain Management → `blackletter.studio` → NS section → replace existing nameservers with the two Cloudflare names from Step 2 → Save.

- [ ] **Step 4: Verify propagation**

```bash
dig NS blackletter.studio +short
```

Expected: the two Cloudflare nameservers from Step 2. Propagation often takes 5–30 minutes; run this in a loop if needed.

- [ ] **Step 5: Confirm activation in Cloudflare**

Back in the Cloudflare dashboard, the zone status should switch from `Pending` to `Active`. Cloudflare sends a confirmation email on activation.

- [ ] **Step 6: Enable zone security settings**

SSL/TLS → Overview → set mode to **Full (strict)** (not Flexible).
SSL/TLS → Edge Certificates → enable **Always Use HTTPS**, **HSTS** (max-age 6 months to start; ramp to 12 months after launch), **Minimum TLS version 1.3**, **Opportunistic Encryption**, **TLS 1.3**.
Speed → Optimization → enable **Brotli**.

- [ ] **Step 7: Create scoped API token for CI**

My Profile → API Tokens → Create Token → Template "Edit zone DNS" → Zone Resources: Include → Specific zone → `blackletter.studio` → TTL: no expiry for now (rotate annually per spec). Copy token; store in 1Password under `Black Letter / Cloudflare API token`.

- [ ] **Step 8: Commit nothing (this task is infra-only)**

No git work here. Progress is recorded by running Task 3.

---

## Task 3 — Create `.github` org-defaults repo with shared templates

**Files:**

- Create: `~/GitRepos/blackletter-studio/.github-org/.github/PULL_REQUEST_TEMPLATE.md`
- Create: `~/GitRepos/blackletter-studio/.github-org/.github/ISSUE_TEMPLATE/bug_report.yml`
- Create: `~/GitRepos/blackletter-studio/.github-org/.github/ISSUE_TEMPLATE/feature_request.yml`
- Create: `~/GitRepos/blackletter-studio/.github-org/.github/ISSUE_TEMPLATE/config.yml`
- Create: `~/GitRepos/blackletter-studio/.github-org/.github/workflows/reusable-node-ci.yml`
- Create: `~/GitRepos/blackletter-studio/.github-org/CODEOWNERS.template`
- Create: `~/GitRepos/blackletter-studio/.github-org/SECURITY.md`
- Create: `~/GitRepos/blackletter-studio/.github-org/CODE_OF_CONDUCT.md`
- Create: `~/GitRepos/blackletter-studio/.github-org/profile/README.md`

- [ ] **Step 1: Create repo on GitHub and clone locally**

```bash
cd ~/GitRepos/blackletter-studio
gh repo create blackletter-studio/.github --public --description "Org-wide defaults for Black Letter Studio" --clone
mv .github .github-org   # local folder rename so it doesn't conflict with .github dirs inside other repos
cd .github-org
git checkout -b feat/initial-defaults
```

- [ ] **Step 2: Create the PR template**

File: `.github/PULL_REQUEST_TEMPLATE.md`

```markdown
## Summary

<!-- 1-3 bullets describing what this PR changes and why. -->

## Linked issue

Closes #

## Test plan

- [ ] Unit tests pass locally
- [ ] Type-check passes
- [ ] Lint passes
- [ ] Manual verification: <describe what you did>

## Risk

- [ ] No user-facing behavior change
- [ ] Destructive operations gated behind confirmation
- [ ] No new network calls added
- [ ] No new telemetry introduced

## Checklist

- [ ] Signed commits
- [ ] Conventional Commit style (feat/fix/docs/refactor/test/chore/ci/perf/build)
- [ ] Co-Authored-By line included for AI assistance
```

- [ ] **Step 3: Create the bug-report issue template**

File: `.github/ISSUE_TEMPLATE/bug_report.yml`

```yaml
name: Bug report
description: Something is broken
labels: ["bug"]
body:
  - type: textarea
    id: what-happened
    attributes:
      label: What happened?
      description: Steps to reproduce, expected vs actual.
    validations:
      required: true
  - type: input
    id: version
    attributes:
      label: Version
      description: App / add-in version string (visible in "About" panel)
  - type: input
    id: os
    attributes:
      label: OS and Word version
      placeholder: "macOS 15.2, Word for Mac 16.82"
```

- [ ] **Step 4: Create the feature-request issue template**

File: `.github/ISSUE_TEMPLATE/feature_request.yml`

```yaml
name: Feature request
description: Suggest an improvement
labels: ["feature"]
body:
  - type: textarea
    id: problem
    attributes:
      label: Problem statement
      description: What are you trying to accomplish and what's getting in the way?
    validations:
      required: true
  - type: textarea
    id: proposal
    attributes:
      label: Proposed solution
  - type: textarea
    id: alternatives
    attributes:
      label: Alternatives considered
```

- [ ] **Step 5: Disable blank issues via config**

File: `.github/ISSUE_TEMPLATE/config.yml`

```yaml
blank_issues_enabled: false
contact_links:
  - name: Anonymous feedback form (no GitHub account needed)
    url: https://blackletter.studio/feedback
    about: Prefer anonymous? Use the public form.
  - name: Security disclosure (private)
    url: mailto:security@blackletter.studio
    about: Report security issues privately.
```

- [ ] **Step 6: Create CODEOWNERS template for child repos**

File: `CODEOWNERS.template`

```
# Copy this to .github/CODEOWNERS in each repo and tune paths.
# Default reviewers for everything:
*       @twitchyvr

# Dictionary changes: require extra review (mirrors spec's two-reviewer gate)
/src/**/*.dic     @twitchyvr
/src/**/*.aff     @twitchyvr

# Security-sensitive paths
/SECURITY.md      @twitchyvr
```

- [ ] **Step 7: Create SECURITY.md**

File: `SECURITY.md`

```markdown
# Security Policy

## Reporting a vulnerability

Email **security@blackletter.studio** with details. PGP-signed email preferred; public key at https://blackletter.studio/.well-known/security.asc (published after M5).

**Scope:** any Black Letter product (Fair Copy, Proofmark, marketing site, feedback Worker).

**Response commitment:** acknowledgement within 3 business days, coordinated disclosure within 90 days. Reporters credited in release notes with consent.

**No bug bounty at launch.** A hall-of-fame page is maintained at https://blackletter.studio/trust#disclosure-credits.

## What we consider in scope

- Unauthorized access to license data
- Document content exfiltration paths (there should be none; prove us wrong)
- XSS / injection in task pane or feedback form
- Supply-chain compromise vectors in the dictionary repo

## What is not in scope

- Social engineering of maintainers
- Denial of service via sending extremely large files (Word handles these; we inherit its limits)
- Issues in Microsoft Office itself (report to Microsoft MSRC)
```

- [ ] **Step 8: Create CODE_OF_CONDUCT.md** (Contributor Covenant 2.1 standard text — use the [official version](https://www.contributor-covenant.org/version/2/1/code_of_conduct/) verbatim, replacing the contact email with `conduct@blackletter.studio`).

- [ ] **Step 9: Create org profile README**

File: `profile/README.md`

```markdown
# Black Letter Studio

Simple, elegant desktop tools for legal professionals.

- Launch product: **Fair Copy** — strips formatting noise from DOCX files while preserving emphasis and alignment. Paired with **Proofmark**, a typo-highlighter that won't collide with your tracked changes.
- Website: https://blackletter.studio
- Feedback (no GitHub account required): https://blackletter.studio/feedback
- Security disclosure: security@blackletter.studio

Our products never send your documents anywhere. Buy once. Use forever.
```

- [ ] **Step 10: Create reusable Node CI workflow**

File: `.github/workflows/reusable-node-ci.yml`

```yaml
name: Reusable Node CI
on:
  workflow_call:
    inputs:
      node-version:
        type: string
        default: "20"
      working-directory:
        type: string
        default: "."
      run-build:
        type: boolean
        default: true
      run-test:
        type: boolean
        default: true
    secrets:
      turbo_token:
        required: false

jobs:
  node-ci:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pull-requests: write
    defaults:
      run:
        working-directory: ${{ inputs.working-directory }}
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v4
        with:
          version: 9
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ inputs.node-version }}
          cache: "pnpm"
          cache-dependency-path: ${{ inputs.working-directory }}/pnpm-lock.yaml
      - run: pnpm install --frozen-lockfile
      - run: pnpm lint
      - run: pnpm typecheck
      - if: ${{ inputs.run-test }}
        run: pnpm test
      - if: ${{ inputs.run-build }}
        run: pnpm build
```

- [ ] **Step 11: Commit + push + open PR + merge**

```bash
git add .
git commit -m "feat: seed org-wide defaults (templates, CODEOWNERS, SECURITY.md, reusable CI)

Co-Authored-By: Claude <noreply@anthropic.com>"
git push -u origin feat/initial-defaults
gh pr create --title "feat: seed org-wide defaults" --body "Initial PR templates, issue forms, SECURITY policy, CoC, org profile README, and the reusable Node CI workflow consumed by every child repo. Closes the M0 Task 3 checkbox in docs/superpowers/plans/2026-04-16-m0-scaffold-and-infrastructure.md."
gh pr merge --squash --delete-branch --admin
```

- [ ] **Step 12: Enable org-level security settings (UI)**

GitHub → org `blackletter-studio` → Settings → Code security:

- Require two-factor authentication for everyone in this organization: **Enabled**
- Dependency graph: **Enabled**
- Dependabot alerts: **Enabled** (new repos inherit)
- Dependabot security updates: **Enabled**
- Secret scanning: **Enabled**
- Push protection (secret scanning): **Enabled**

---

## Task 4 — Create `brand` private repo with color tokens, fonts, and placeholder wordmark

**Files:**

- Create: `~/GitRepos/blackletter-studio/brand/README.md`
- Create: `~/GitRepos/blackletter-studio/brand/tokens/tokens.css`
- Create: `~/GitRepos/blackletter-studio/brand/tokens/tokens.ts`
- Create: `~/GitRepos/blackletter-studio/brand/fonts/README.md`
- Create: `~/GitRepos/blackletter-studio/brand/wordmark/black-letter-wordmark-placeholder.svg`
- Create: `~/GitRepos/blackletter-studio/brand/.gitignore`

- [ ] **Step 1: Create and clone repo**

```bash
cd ~/GitRepos/blackletter-studio
gh repo create blackletter-studio/brand --private --description "Black Letter brand assets: tokens, fonts, wordmarks" --clone
cd brand
git checkout -b feat/initial-tokens
```

- [ ] **Step 2: Write `tokens/tokens.css` (the V1 Editorial palette + type scale)**

```css
/* Black Letter — V1 Editorial tokens.
 * Consumed by the `site` and `fair-copy` repos.
 * Change only via PR with two reviewers.
 */
:root {
  /* Palette */
  --color-paper: #faf6ef; /* ivory background */
  --color-ink: #1a1a1a; /* primary type */
  --color-ink-soft: #3a3a3a; /* secondary type */
  --color-oxblood: #6b1e2a; /* accent + emphasis */
  --color-rule: rgba(0, 0, 0, 0.08);

  /* Typography */
  --font-display: "Fraunces", Georgia, serif;
  --font-body: "IBM Plex Sans", system-ui, sans-serif;
  --font-mono: "IBM Plex Mono", ui-monospace, monospace;

  /* Type scale — major-second-ish, tuned for editorial feel */
  --text-xs: 12px;
  --text-sm: 14px;
  --text-base: 16px;
  --text-md: 18px;
  --text-lg: 22px;
  --text-xl: 28px;
  --text-2xl: 40px;
  --text-3xl: 60px;
  --text-4xl: 96px;

  /* Spacing scale */
  --space-1: 4px;
  --space-2: 8px;
  --space-3: 12px;
  --space-4: 16px;
  --space-6: 24px;
  --space-8: 32px;
  --space-12: 48px;
  --space-16: 64px;
  --space-24: 96px;

  /* Radius */
  --radius-sm: 6px;
  --radius-md: 10px;
  --radius-lg: 14px;

  /* Elevation */
  --shadow-sm: 0 1px 2px rgba(0, 0, 0, 0.06);
  --shadow-md: 0 8px 24px -12px rgba(0, 0, 0, 0.18);
  --shadow-lg: 0 18px 40px -18px rgba(0, 0, 0, 0.28);
}
```

- [ ] **Step 3: Write `tokens/tokens.ts` (same values as TypeScript consts for the add-in + Tailwind config)**

```typescript
/** Black Letter — V1 Editorial tokens (TypeScript).
 *  Same values as tokens.css; keep in sync.
 */
export const colors = {
  paper: "#faf6ef",
  ink: "#1a1a1a",
  inkSoft: "#3a3a3a",
  oxblood: "#6b1e2a",
  rule: "rgba(0, 0, 0, 0.08)",
} as const;

export const fonts = {
  display: '"Fraunces", Georgia, serif',
  body: '"IBM Plex Sans", system-ui, sans-serif',
  mono: '"IBM Plex Mono", ui-monospace, monospace',
} as const;

export const textScale = {
  xs: "12px",
  sm: "14px",
  base: "16px",
  md: "18px",
  lg: "22px",
  xl: "28px",
  "2xl": "40px",
  "3xl": "60px",
  "4xl": "96px",
} as const;

export const space = {
  1: "4px",
  2: "8px",
  3: "12px",
  4: "16px",
  6: "24px",
  8: "32px",
  12: "48px",
  16: "64px",
  24: "96px",
} as const;

export const radius = { sm: "6px", md: "10px", lg: "14px" } as const;

export type ColorToken = keyof typeof colors;
```

- [ ] **Step 4: Add a README explaining the font situation**

File: `fonts/README.md`

```markdown
# Fonts — Black Letter

## Licensing

- **Fraunces** — SIL OFL 1.1 (free for embedding + redistribution, including commercial). Source: Google Fonts / fonts.google.com/specimen/Fraunces.
- **IBM Plex Sans + IBM Plex Mono** — SIL OFL 1.1. Source: github.com/IBM/plex.

Both are safe to self-host and redistribute. No commercial license purchase needed.

## Files

Self-hosted `.woff2` subsets live under `fonts/woff2/`. Subsetting target: Latin + common punctuation + a few typographic extras (smart quotes, en/em dashes). Add new files via PR with two reviewers.

## Adding a font

1. Download source `.ttf` from the upstream.
2. Run `glyphhanger --subset=*.ttf --formats=woff2 --LATIN` (or equivalent subsetting tool) to produce `.woff2` files.
3. Commit the subsetted files only.
4. Update `tokens.css` / `tokens.ts` font-family strings if needed.
5. Open PR.
```

- [ ] **Step 5: Create placeholder wordmark (stub SVG; final wordmark comes during M1)**

File: `wordmark/black-letter-wordmark-placeholder.svg`

```xml
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 600 120" role="img" aria-label="Black Letter">
  <style>
    text { font-family: "Fraunces", "Georgia", serif; font-size: 72px; fill: #1a1a1a; letter-spacing: -0.02em; }
    .accent { fill: #6b1e2a; font-style: italic; }
  </style>
  <text x="20" y="85">Black <tspan class="accent">Letter.</tspan></text>
</svg>
```

- [ ] **Step 6: Create `.gitignore`**

File: `.gitignore`

```
.DS_Store
node_modules/
dist/
*.log
```

- [ ] **Step 7: Copy CODEOWNERS template from the org defaults**

```bash
curl -sSL -o .github/CODEOWNERS https://raw.githubusercontent.com/blackletter-studio/.github/main/CODEOWNERS.template
mkdir -p .github
# (If the file didn't end up in .github/, move it.)
```

- [ ] **Step 8: Commit + push + PR + merge**

```bash
git add .
git commit -m "feat: seed brand tokens, font licensing docs, placeholder wordmark

Co-Authored-By: Claude <noreply@anthropic.com>"
git push -u origin feat/initial-tokens
gh pr create --title "feat: initial brand tokens" --body "Seeds V1 Editorial tokens (CSS + TS), font licensing README (Fraunces + IBM Plex — both OFL), and a placeholder SVG wordmark. Final wordmark lands in M1. Closes M0 Task 4."
gh pr merge --squash --delete-branch --admin
```

---

## Task 5 — Scaffold `site` repo (Astro + Tailwind + placeholder pages)

**Files:**

- Create: `~/GitRepos/blackletter-studio/site/` (Astro project root)
- Create: `~/GitRepos/blackletter-studio/site/astro.config.mjs`
- Create: `~/GitRepos/blackletter-studio/site/tailwind.config.js`
- Create: `~/GitRepos/blackletter-studio/site/src/styles/global.css`
- Create: `~/GitRepos/blackletter-studio/site/src/layouts/BaseLayout.astro`
- Create: `~/GitRepos/blackletter-studio/site/src/components/Nav.astro`
- Create: `~/GitRepos/blackletter-studio/site/src/components/Footer.astro`
- Create: `~/GitRepos/blackletter-studio/site/src/pages/index.astro`
- Create: `~/GitRepos/blackletter-studio/site/src/pages/fair-copy.astro`
- Create: `~/GitRepos/blackletter-studio/site/src/pages/proofmark.astro`
- Create: `~/GitRepos/blackletter-studio/site/src/pages/trust.astro`
- Create: `~/GitRepos/blackletter-studio/site/src/pages/feedback.astro`
- Create: `~/GitRepos/blackletter-studio/site/src/pages/support.astro`
- Create: `~/GitRepos/blackletter-studio/site/src/pages/legal/eula.astro`
- Create: `~/GitRepos/blackletter-studio/site/src/pages/legal/privacy.astro`
- Create: `~/GitRepos/blackletter-studio/site/.github/workflows/ci.yml`
- Create: `~/GitRepos/blackletter-studio/site/.gitmodules` (brand submodule)

- [ ] **Step 1: Create repo and clone**

```bash
cd ~/GitRepos/blackletter-studio
gh repo create blackletter-studio/site --public --description "Black Letter Studio — marketing site and feedback Worker" --clone
cd site
git checkout -b feat/initial-scaffold
```

- [ ] **Step 2: Initialize Astro with TypeScript + Tailwind + MDX**

```bash
pnpm create astro@latest . --template minimal --typescript strict --install --no-git
pnpm astro add tailwind mdx sitemap
```

Answer prompts:

- "Add devDependencies for Tailwind": Yes
- "Generate tailwind.config.js": Yes
- "Add `@astrojs/mdx` integration": Yes
- "Add `@astrojs/sitemap`": Yes

- [ ] **Step 3: Add the brand repo as a git submodule**

```bash
git submodule add git@github.com:blackletter-studio/brand.git vendor/brand
```

Resulting: `site/vendor/brand/tokens/tokens.css` and `site/vendor/brand/tokens/tokens.ts` usable by the site.

- [ ] **Step 4: Write `tailwind.config.js` consuming brand tokens**

```js
import {
  colors,
  fonts,
  textScale,
  space,
  radius,
} from "./vendor/brand/tokens/tokens.ts";

/** @type {import('tailwindcss').Config} */
export default {
  content: ["./src/**/*.{astro,html,ts,tsx,md,mdx}"],
  theme: {
    extend: {
      colors: {
        paper: colors.paper,
        ink: colors.ink,
        "ink-soft": colors.inkSoft,
        oxblood: colors.oxblood,
      },
      fontFamily: {
        display: fonts.display,
        body: fonts.body,
        mono: fonts.mono,
      },
      fontSize: textScale,
      spacing: space,
      borderRadius: radius,
    },
  },
  plugins: [],
};
```

- [ ] **Step 5: Write `src/styles/global.css` loading tokens and fonts**

```css
@import "../../vendor/brand/tokens/tokens.css";

@tailwind base;
@tailwind components;
@tailwind utilities;

@font-face {
  font-family: "Fraunces";
  font-style: normal;
  font-weight: 300 700;
  font-display: swap;
  src: url("/fonts/Fraunces-VF-Latin.woff2") format("woff2-variations");
}

@font-face {
  font-family: "IBM Plex Sans";
  font-style: normal;
  font-weight: 300 600;
  font-display: swap;
  src: url("/fonts/IBMPlexSans-Latin.woff2") format("woff2");
}

@font-face {
  font-family: "IBM Plex Mono";
  font-style: normal;
  font-weight: 400 500;
  font-display: swap;
  src: url("/fonts/IBMPlexMono-Latin.woff2") format("woff2");
}

html {
  background: var(--color-paper);
  color: var(--color-ink);
  font-family: var(--font-body);
}
body {
  font-size: var(--text-base);
  line-height: 1.55;
}
```

- [ ] **Step 6: Download and commit subset font files**

```bash
mkdir -p public/fonts
# Temporary placeholder — real subsets ship in M1.
# For M0, copy the full woff2 from fonts.google.com and IBM Plex GitHub releases.
curl -L -o public/fonts/Fraunces-VF-Latin.woff2 https://fonts.gstatic.com/s/fraunces/v32/6NUh8FyLNQOQZAnv9ZwNjucMHVn85Ni7emAe9lKqZTnaFPjOGwP5oK7HZFYGHFjo.woff2
curl -L -o public/fonts/IBMPlexSans-Latin.woff2 https://github.com/IBM/plex/raw/master/packages/plex-sans/fonts/complete/woff2/IBMPlexSans-Regular.woff2
curl -L -o public/fonts/IBMPlexMono-Latin.woff2 https://github.com/IBM/plex/raw/master/packages/plex-mono/fonts/complete/woff2/IBMPlexMono-Regular.woff2
```

- [ ] **Step 7: Write `src/layouts/BaseLayout.astro`**

```astro
---
import "../styles/global.css";
import Nav from "../components/Nav.astro";
import Footer from "../components/Footer.astro";

interface Props {
  title: string;
  description?: string;
}
const { title, description = "Black Letter — simple, elegant desktop tools for legal professionals." } = Astro.props;
---
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width,initial-scale=1" />
    <title>{title} · Black Letter</title>
    <meta name="description" content={description} />
    <link rel="icon" href="/favicon.svg" />
  </head>
  <body class="font-body bg-paper text-ink">
    <Nav />
    <main class="max-w-4xl mx-auto px-6 py-16"><slot /></main>
    <Footer />
  </body>
</html>
```

- [ ] **Step 8: Write `src/components/Nav.astro`**

```astro
---
const links = [
  { href: "/fair-copy", label: "Fair Copy" },
  { href: "/proofmark", label: "Proofmark" },
  { href: "/trust", label: "Trust" },
  { href: "/support", label: "Support" },
];
---
<nav class="border-b border-black/10">
  <div class="max-w-4xl mx-auto px-6 h-14 flex items-center justify-between">
    <a href="/" class="font-display text-xl"><span>Black</span> <em class="not-italic text-oxblood">Letter.</em></a>
    <ul class="flex gap-6 text-sm">
      {links.map(l => <li><a href={l.href} class="hover:text-oxblood">{l.label}</a></li>)}
    </ul>
  </div>
</nav>
```

- [ ] **Step 9: Write `src/components/Footer.astro`**

```astro
<footer class="border-t border-black/10 mt-24">
  <div class="max-w-4xl mx-auto px-6 py-10 text-sm text-ink-soft flex flex-wrap gap-6 justify-between">
    <span>&copy; {new Date().getFullYear()} Black Letter Studio</span>
    <nav class="flex gap-4">
      <a href="/legal/privacy">Privacy</a>
      <a href="/legal/eula">EULA</a>
      <a href="/feedback">Feedback</a>
      <a href="mailto:security@blackletter.studio">Security</a>
    </nav>
  </div>
</footer>
```

- [ ] **Step 10: Write `src/pages/index.astro`** (placeholder home)

```astro
---
import BaseLayout from "../layouts/BaseLayout.astro";
---
<BaseLayout title="Simple tools for the considered practice">
  <h1 class="font-display text-4xl leading-none tracking-tight mb-8">
    Simple tools for the <em class="text-oxblood not-italic font-normal">considered practice.</em>
  </h1>
  <p class="text-lg text-ink-soft max-w-2xl mb-12">
    Black Letter is a new studio shipping quiet, precise utilities for legal professionals.
    Launching soon with <a href="/fair-copy" class="underline">Fair Copy</a> and
    <a href="/proofmark" class="underline">Proofmark</a> for Microsoft Word.
  </p>
  <p class="text-sm text-ink-soft">Site under construction.</p>
</BaseLayout>
```

- [ ] **Step 11: Create remaining placeholder pages — each a single `<h1>` + one paragraph**

Pages to create with the pattern below (same `BaseLayout`, one heading + one paragraph noting "Coming soon — see the design spec"):

- `src/pages/fair-copy.astro`
- `src/pages/proofmark.astro`
- `src/pages/trust.astro`
- `src/pages/feedback.astro`
- `src/pages/support.astro`
- `src/pages/legal/eula.astro`
- `src/pages/legal/privacy.astro`

Example (`fair-copy.astro`):

```astro
---
import BaseLayout from "../layouts/BaseLayout.astro";
---
<BaseLayout title="Fair Copy">
  <h1 class="font-display text-3xl mb-6">Fair <em class="text-oxblood not-italic">Copy.</em></h1>
  <p class="text-ink-soft">Coming soon — strip formatting noise from any Word document while keeping what you meant.</p>
</BaseLayout>
```

- [ ] **Step 12: Write CI workflow**

File: `.github/workflows/ci.yml`

```yaml
name: CI
on:
  pull_request:
    branches: [main, develop]
  push:
    branches: [main, develop]

jobs:
  ci:
    uses: blackletter-studio/.github/.github/workflows/reusable-node-ci.yml@main
    with:
      node-version: "20"
      working-directory: "."
      run-build: true
      run-test: false
```

Add minimal package.json scripts to satisfy the reusable workflow:

```json
{
  "scripts": {
    "lint": "astro check",
    "typecheck": "astro check",
    "build": "astro build",
    "dev": "astro dev"
  }
}
```

- [ ] **Step 13: Verify build works locally**

```bash
pnpm install
pnpm build
```

Expected: `dist/` directory populated with HTML files. No TypeScript errors.

- [ ] **Step 14: Commit + push**

```bash
git add .
git commit -m "feat: initial Astro scaffold with placeholder pages, brand tokens wired

Co-Authored-By: Claude <noreply@anthropic.com>"
git push -u origin feat/initial-scaffold
gh pr create --title "feat: initial site scaffold" --body "Astro + Tailwind + MDX skeleton. Brand tokens imported via submodule. Placeholder pages for home, fair-copy, proofmark, trust, feedback, support, legal/*. CI wired to reusable workflow. Real content lands in M1. Closes M0 Task 5."
gh pr merge --squash --delete-branch --admin
```

---

## Task 6 — Connect `site` to Netlify and deploy to `blackletter.studio`

**Files:**

- Create: `~/GitRepos/blackletter-studio/site/netlify.toml`

- [ ] **Step 1: Create Netlify site from GitHub repo**

Netlify dashboard → Add new site → Import existing project → GitHub → authorize `blackletter-studio/site`.

Build settings:

- Build command: `pnpm build`
- Publish directory: `dist`
- Node version: 20

- [ ] **Step 2: Add `netlify.toml` to pin build settings in code**

```toml
[build]
  command = "pnpm build"
  publish = "dist"

[build.environment]
  NODE_VERSION = "20"
  PNPM_VERSION = "9"

[[headers]]
  for = "/*"
  [headers.values]
    X-Frame-Options = "DENY"
    X-Content-Type-Options = "nosniff"
    Referrer-Policy = "strict-origin-when-cross-origin"
    Permissions-Policy = "interest-cohort=()"
    Strict-Transport-Security = "max-age=15552000; includeSubDomains"

[[headers]]
  for = "/*.woff2"
  [headers.values]
    Cache-Control = "public, max-age=31536000, immutable"

[build.processing]
  skip_processing = false
[build.processing.css]
  bundle = true
  minify = true
[build.processing.js]
  bundle = true
  minify = true
[build.processing.html]
  pretty_urls = true
```

- [ ] **Step 3: Commit netlify.toml on a new branch**

```bash
git checkout -b chore/netlify-config
git add netlify.toml
git commit -m "chore: pin Netlify build + security headers

Co-Authored-By: Claude <noreply@anthropic.com>"
git push -u origin chore/netlify-config
gh pr create --title "chore: pin Netlify build + security headers" --body "Commits Netlify config to repo so build settings and security headers are reviewed like any other code."
gh pr merge --squash --delete-branch --admin
```

Netlify will auto-deploy the merged change.

- [ ] **Step 4: In Netlify, add custom domain**

Netlify site → Domain management → Add domain → `blackletter.studio` → skip Netlify DNS (we manage DNS at Cloudflare).

Netlify will display the target hostname (like `mysite-abc123.netlify.app`).

- [ ] **Step 5: Add CNAME records at Cloudflare**

Cloudflare dashboard → blackletter.studio → DNS → Add record:

- Type: `CNAME`, Name: `www`, Target: `<netlify-target-from-step-4>`, Proxy status: **Proxied (orange cloud)**, TTL: Auto.
- Type: `CNAME`, Name: `@` (apex), Target: `<netlify-target>`, Proxy: **Proxied**. (Cloudflare supports CNAME flattening on the apex.)

- [ ] **Step 6: Let Netlify + Cloudflare coordinate on SSL**

Because Cloudflare proxies (orange cloud) and Netlify also provisions SSL, set Cloudflare SSL/TLS mode to **Full (strict)** and let Netlify complete its Let's Encrypt issuance. Verify the site loads:

```bash
curl -I https://blackletter.studio
```

Expected: `HTTP/2 200`, security headers present from `netlify.toml`, `strict-transport-security` header set.

- [ ] **Step 7: Verify in a browser**

Open https://blackletter.studio → see placeholder home page with V1 Editorial styling.

- [ ] **Step 8: No code commit needed for this task** — infrastructure only. Success state: a Google-searchable, HTTPS-terminated placeholder site is live.

---

## Task 7 — Scaffold `fair-copy` repo (Office add-in skeleton)

**Files:**

- Create: `~/GitRepos/blackletter-studio/fair-copy/` (project root via Office generator or manual)
- Create: `~/GitRepos/blackletter-studio/fair-copy/manifest.xml`
- Create: `~/GitRepos/blackletter-studio/fair-copy/package.json`
- Create: `~/GitRepos/blackletter-studio/fair-copy/tsconfig.json`
- Create: `~/GitRepos/blackletter-studio/fair-copy/vite.config.ts`
- Create: `~/GitRepos/blackletter-studio/fair-copy/src/taskpane/index.html`
- Create: `~/GitRepos/blackletter-studio/fair-copy/src/taskpane/main.tsx`
- Create: `~/GitRepos/blackletter-studio/fair-copy/src/taskpane/App.tsx`
- Create: `~/GitRepos/blackletter-studio/fair-copy/src/commands/commands.ts`
- Create: `~/GitRepos/blackletter-studio/fair-copy/.eslintrc.cjs`
- Create: `~/GitRepos/blackletter-studio/fair-copy/.prettierrc`
- Create: `~/GitRepos/blackletter-studio/fair-copy/.github/workflows/ci.yml`
- Create: `~/GitRepos/blackletter-studio/fair-copy/tests/setup.ts`
- Create: `~/GitRepos/blackletter-studio/fair-copy/tests/App.test.tsx`

- [ ] **Step 1: Create repo and clone**

```bash
cd ~/GitRepos/blackletter-studio
gh repo create blackletter-studio/fair-copy --public --description "Fair Copy + Proofmark — the Black Letter Word add-in" --clone
cd fair-copy
git checkout -b feat/initial-scaffold
git submodule add git@github.com:blackletter-studio/brand.git vendor/brand
```

- [ ] **Step 2: Initialize package with pnpm**

```bash
pnpm init
```

Edit `package.json` to add these fields:

```json
{
  "name": "@black-letter/fair-copy",
  "private": true,
  "version": "0.1.0",
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "tsc -b && vite build",
    "typecheck": "tsc --noEmit",
    "lint": "eslint . --ext ts,tsx --max-warnings 0",
    "format": "prettier --write .",
    "test": "vitest run",
    "test:watch": "vitest",
    "start:word": "office-addin-debugging start manifest.xml desktop"
  }
}
```

- [ ] **Step 3: Install dependencies**

```bash
pnpm add react react-dom
pnpm add @fluentui/react-components @fluentui/react-icons
pnpm add -D typescript @types/react @types/react-dom @types/node
pnpm add -D vite @vitejs/plugin-react
pnpm add -D vitest @testing-library/react @testing-library/jest-dom jsdom
pnpm add -D eslint @typescript-eslint/parser @typescript-eslint/eslint-plugin eslint-plugin-react eslint-plugin-react-hooks eslint-plugin-security eslint-plugin-jsx-a11y
pnpm add -D prettier
pnpm add -D office-addin-debugging office-addin-manifest
pnpm add -D @types/office-js
```

- [ ] **Step 4: Write `tsconfig.json`**

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "lib": ["ES2022", "DOM", "DOM.Iterable"],
    "jsx": "react-jsx",
    "strict": true,
    "noImplicitAny": true,
    "noUncheckedIndexedAccess": true,
    "noFallthroughCasesInSwitch": true,
    "noImplicitReturns": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "types": ["office-js", "vitest/globals"]
  },
  "include": ["src", "tests"]
}
```

- [ ] **Step 5: Write `vite.config.ts`**

```typescript
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";
import { resolve } from "path";

export default defineConfig({
  plugins: [react()],
  server: { port: 3000, https: true },
  build: {
    rollupOptions: {
      input: {
        taskpane: resolve(__dirname, "src/taskpane/index.html"),
        commands: resolve(__dirname, "src/commands/commands.html"),
      },
    },
  },
  test: {
    environment: "jsdom",
    globals: true,
    setupFiles: ["./tests/setup.ts"],
  },
});
```

- [ ] **Step 6: Write the Office add-in manifest**

File: `manifest.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<OfficeApp xmlns="http://schemas.microsoft.com/office/appforoffice/1.1"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xmlns:bt="http://schemas.microsoft.com/office/officeappbasictypes/1.0"
           xmlns:ov="http://schemas.microsoft.com/office/taskpaneappversionoverrides"
           xsi:type="TaskPaneApp">
  <Id>00000000-0000-0000-0000-000000000001</Id>
  <Version>0.1.0.0</Version>
  <ProviderName>Black Letter Studio</ProviderName>
  <DefaultLocale>en-US</DefaultLocale>
  <DisplayName DefaultValue="Fair Copy (dev)" />
  <Description DefaultValue="Strip formatting noise from your Word document. Dev build." />
  <IconUrl DefaultValue="https://localhost:3000/assets/icon-80.png" />
  <HighResolutionIconUrl DefaultValue="https://localhost:3000/assets/icon-128.png" />
  <SupportUrl DefaultValue="https://blackletter.studio/support" />
  <AppDomains>
    <AppDomain>https://blackletter.studio</AppDomain>
  </AppDomains>
  <Hosts>
    <Host Name="Document" />
  </Hosts>
  <Requirements>
    <Sets DefaultMinVersion="1.7">
      <Set Name="WordApi" MinVersion="1.7" />
    </Sets>
  </Requirements>
  <DefaultSettings>
    <SourceLocation DefaultValue="https://localhost:3000/src/taskpane/index.html" />
  </DefaultSettings>
  <Permissions>ReadWriteDocument</Permissions>
</OfficeApp>
```

- [ ] **Step 7: Create task pane HTML shell**

File: `src/taskpane/index.html`

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <title>Fair Copy</title>
    <meta
      http-equiv="Content-Security-Policy"
      content="default-src 'none'; connect-src 'self' https://graph.microsoft.com https://updates.blackletter.studio; script-src 'self' 'wasm-unsafe-eval'; style-src 'self' 'unsafe-inline'; img-src 'self' data:;"
    />
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="./main.tsx"></script>
  </body>
</html>
```

- [ ] **Step 8: Bootstrap React**

File: `src/taskpane/main.tsx`

```tsx
import React from "react";
import { createRoot } from "react-dom/client";
import { FluentProvider, webLightTheme } from "@fluentui/react-components";
import { App } from "./App";

Office.onReady(() => {
  const root = createRoot(document.getElementById("root")!);
  root.render(
    <React.StrictMode>
      <FluentProvider theme={webLightTheme}>
        <App />
      </FluentProvider>
    </React.StrictMode>,
  );
});
```

- [ ] **Step 9: Placeholder App component**

File: `src/taskpane/App.tsx`

```tsx
import { Button, Text } from "@fluentui/react-components";

export function App(): JSX.Element {
  return (
    <main style={{ padding: 24, fontFamily: "Fraunces, Georgia, serif" }}>
      <Text as="h1" size={600} weight="regular" style={{ marginBottom: 16 }}>
        Fair Copy{" "}
        <span style={{ color: "#6b1e2a", fontStyle: "italic" }}>·</span> dev
      </Text>
      <Text as="p" size={300} style={{ marginBottom: 24, color: "#3a3a3a" }}>
        M0 scaffold. Clean button here in M2.
      </Text>
      <Button appearance="primary" disabled>
        Clean (coming in M2)
      </Button>
    </main>
  );
}
```

- [ ] **Step 10: Placeholder commands handler**

File: `src/commands/commands.ts`

```typescript
/* global Office */
Office.onReady(() => {
  // Ribbon command handlers will be registered here starting in M2.
});
```

Plus `src/commands/commands.html`:

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <title>Commands</title>
  </head>
  <body>
    <script type="module" src="./commands.ts"></script>
  </body>
</html>
```

- [ ] **Step 11: Write `.eslintrc.cjs`**

```javascript
module.exports = {
  root: true,
  parser: "@typescript-eslint/parser",
  parserOptions: {
    ecmaVersion: 2022,
    sourceType: "module",
    project: "./tsconfig.json",
    ecmaFeatures: { jsx: true },
  },
  plugins: [
    "@typescript-eslint",
    "react",
    "react-hooks",
    "security",
    "jsx-a11y",
  ],
  extends: [
    "eslint:recommended",
    "plugin:@typescript-eslint/recommended-type-checked",
    "plugin:react/recommended",
    "plugin:react-hooks/recommended",
    "plugin:jsx-a11y/recommended",
    "plugin:security/recommended-legacy",
  ],
  settings: { react: { version: "detect" } },
  rules: {
    "@typescript-eslint/no-explicit-any": "error",
    "@typescript-eslint/no-unused-vars": ["error", { argsIgnorePattern: "^_" }],
    "react/react-in-jsx-scope": "off",
    "react/prop-types": "off",
  },
  ignorePatterns: ["dist", "node_modules", "vendor"],
};
```

- [ ] **Step 12: Write `.prettierrc`**

```json
{
  "semi": true,
  "singleQuote": false,
  "trailingComma": "all",
  "printWidth": 100,
  "tabWidth": 2
}
```

- [ ] **Step 13: Write Vitest setup and a placeholder test**

File: `tests/setup.ts`

```typescript
import "@testing-library/jest-dom";

// Minimal Office.js stub so components under test don't crash on `Office.onReady`.
(globalThis as any).Office = {
  onReady: (cb: () => void) => {
    cb();
    return Promise.resolve();
  },
};
```

File: `tests/App.test.tsx`

```tsx
import { describe, it, expect } from "vitest";
import { render, screen } from "@testing-library/react";
import { FluentProvider, webLightTheme } from "@fluentui/react-components";
import { App } from "../src/taskpane/App";

describe("App (scaffold)", () => {
  it("renders the Fair Copy header and a disabled Clean button", () => {
    render(
      <FluentProvider theme={webLightTheme}>
        <App />
      </FluentProvider>,
    );
    expect(screen.getByText(/Fair Copy/i)).toBeInTheDocument();
    expect(screen.getByRole("button", { name: /Clean/i })).toBeDisabled();
  });
});
```

- [ ] **Step 14: Run it — expect one passing test**

```bash
pnpm test
```

Expected: `1 passed`.

- [ ] **Step 15: Run typecheck and lint — expect clean**

```bash
pnpm typecheck
pnpm lint
```

Expected: no errors.

- [ ] **Step 16: Sideload the add-in into local Word and confirm it renders**

```bash
pnpm start:word
```

Office-addin-debugging will open Word, sideload the manifest, and open the task pane. Expected: the Fair Copy placeholder (header + disabled Clean button) appears in the task pane.

- [ ] **Step 17: Write CI workflow**

File: `.github/workflows/ci.yml`

```yaml
name: CI
on:
  pull_request:
    branches: [main, develop]
  push:
    branches: [main, develop]
jobs:
  ci:
    uses: blackletter-studio/.github/.github/workflows/reusable-node-ci.yml@main
    with:
      node-version: "20"
      run-build: true
      run-test: true
```

- [ ] **Step 18: Commit + push + PR + merge**

```bash
git add .
git commit -m "feat: initial Office add-in scaffold with React + Fluent UI + Vite + strict TS

- Task pane renders placeholder Fair Copy UI in local Word
- CI wired to org reusable workflow
- Strict ESLint + Prettier
- Vitest smoke test passing

Co-Authored-By: Claude <noreply@anthropic.com>"
git push -u origin feat/initial-scaffold
gh pr create --title "feat: initial add-in scaffold" --body "Sideload-able add-in skeleton. No product logic — Clean button is disabled. M2 builds the real engine. Closes M0 Task 7."
gh pr merge --squash --delete-branch --admin
```

---

## Task 8 — Scaffold `legal-augmentation-dictionary` repo (data-only)

**Files:**

- Create: `~/GitRepos/blackletter-studio/legal-augmentation-dictionary/README.md`
- Create: `~/GitRepos/blackletter-studio/legal-augmentation-dictionary/src/en_US/legal-augmentation.dic`
- Create: `~/GitRepos/blackletter-studio/legal-augmentation-dictionary/src/en_US/legal-augmentation.aff`
- Create: `~/GitRepos/blackletter-studio/legal-augmentation-dictionary/scripts/validate.py`
- Create: `~/GitRepos/blackletter-studio/legal-augmentation-dictionary/.github/workflows/ci.yml`
- Create: `~/GitRepos/blackletter-studio/legal-augmentation-dictionary/.github/CODEOWNERS`
- Create: `~/GitRepos/blackletter-studio/legal-augmentation-dictionary/LICENSE`

- [ ] **Step 1: Create repo + clone**

```bash
cd ~/GitRepos/blackletter-studio
gh repo create blackletter-studio/legal-augmentation-dictionary --public --description "Hunspell dictionary augmentation for US legal terminology" --clone
cd legal-augmentation-dictionary
git checkout -b feat/initial-seed
```

- [ ] **Step 2: Write the README**

```markdown
# legal-augmentation-dictionary

A curated Hunspell dictionary extension that teaches Proofmark (the Black Letter typo-highlighter) the specialized vocabulary of US legal practice — Latin maxims, court names, Bluebook abbreviations, Bates-style tokens, common legal terminology, and signing formulas.

## What's here

- `src/en_US/legal-augmentation.dic` — word list, one term per line, with optional affix flags. **Hand-curated.** 5,000-term target by v1.0 launch.
- `src/en_US/legal-augmentation.aff` — affix rules (rare; most entries need none).
- `scripts/validate.py` — CI validator: parses the files via `hunspell`, runs the dictionary against a canary corpus, and reports false-positive / false-negative deltas.

## Adding a term

1. Open a PR adding the term to `legal-augmentation.dic`, alphabetically.
2. Include a short reference (case law, Black's Law Dictionary, Bluebook rule number, etc.) in the PR description.
3. Two reviewers required (one maintainer + one lawyer). CODEOWNERS enforces this.
4. CI must pass (parse check + canary-corpus delta check).

## License

SIL Open Font License 1.1 compatible. See [LICENSE](LICENSE).

The dictionary content itself is released under CC0 (public domain dedication): free to use, modify, redistribute, commercially or otherwise. Attribution appreciated but not required.
```

- [ ] **Step 3: Seed ~50 terms in `legal-augmentation.dic`**

File: `src/en_US/legal-augmentation.dic`

```
50
ab
ad
affidavit
a
fortiori
amicus
animo
ante
bona
fide
certiorari
contra
corpus
curia
de
facto
jure
novo
delicto
dicta
dictum
en
banc
ergo
et
al
seq
ibid
id
idem
in
camera
forma
pauperis
limine
re
rem
infra
ipso
facto
jure
mandamus
nolo
contendere
obiter
pari
passu
passu
per
curiam
se
pro
```

(The first line, `50`, tells Hunspell how many entries follow.)

- [ ] **Step 4: Create a minimal affix file**

File: `src/en_US/legal-augmentation.aff`

```
SET UTF-8
TRY esiarntolcdugmpfhbywkvxzjq
```

- [ ] **Step 5: Write the Python validator**

File: `scripts/validate.py`

```python
#!/usr/bin/env python3
"""Validate the legal-augmentation dictionary parses via hunspell and lint-checks entries.

Exit codes:
  0 = all good
  1 = parse error
  2 = lint error (duplicates, empty lines, etc.)
"""
import subprocess
import sys
from pathlib import Path

ROOT = Path(__file__).parent.parent
DIC = ROOT / "src" / "en_US" / "legal-augmentation.dic"
AFF = ROOT / "src" / "en_US" / "legal-augmentation.aff"

def lint_dic() -> list[str]:
    errors: list[str] = []
    lines = DIC.read_text(encoding="utf-8").splitlines()
    if not lines:
        errors.append("dic file is empty")
        return errors
    try:
        declared_count = int(lines[0])
    except ValueError:
        errors.append(f"first line must be an integer count, got: {lines[0]!r}")
        return errors
    entries = lines[1:]
    if len(entries) != declared_count:
        errors.append(f"declared {declared_count} entries, found {len(entries)}")
    seen: set[str] = set()
    for idx, raw in enumerate(entries, start=2):
        word = raw.strip()
        if not word:
            errors.append(f"line {idx}: empty entry")
        key = word.lower().split("/")[0]
        if key in seen:
            errors.append(f"line {idx}: duplicate entry {key!r}")
        seen.add(key)
    return errors

def parse_via_hunspell() -> list[str]:
    errors: list[str] = []
    try:
        result = subprocess.run(
            ["hunspell", "-d", str(DIC.with_suffix("")), "-a"],
            input="test\n",
            capture_output=True,
            text=True,
            check=False,
            timeout=15,
        )
        if result.returncode != 0:
            errors.append(f"hunspell parse failed: {result.stderr.strip()}")
    except FileNotFoundError:
        errors.append("hunspell binary not found in PATH")
    except subprocess.TimeoutExpired:
        errors.append("hunspell took >15s to parse; suspicious")
    return errors

def main() -> int:
    dic_errors = lint_dic()
    if dic_errors:
        print("Lint errors:")
        for e in dic_errors:
            print(f"  - {e}")
        return 2
    parse_errors = parse_via_hunspell()
    if parse_errors:
        print("Parse errors:")
        for e in parse_errors:
            print(f"  - {e}")
        return 1
    print(f"ok: {DIC.name} passes lint and Hunspell parse")
    return 0

if __name__ == "__main__":
    sys.exit(main())
```

- [ ] **Step 6: Run the validator locally**

```bash
brew install hunspell    # if not installed
python3 scripts/validate.py
```

Expected: `ok: legal-augmentation.dic passes lint and Hunspell parse`.

- [ ] **Step 7: Write CI workflow**

File: `.github/workflows/ci.yml`

```yaml
name: CI
on:
  pull_request:
    branches: [main, develop]
  push:
    branches: [main, develop]
jobs:
  validate-dictionary:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Install hunspell
        run: sudo apt-get update && sudo apt-get install -y hunspell
      - uses: actions/setup-python@v5
        with:
          python-version: "3.12"
      - name: Run validator
        run: python3 scripts/validate.py
```

- [ ] **Step 8: Add CODEOWNERS with the two-reviewer rule**

File: `.github/CODEOWNERS`

```
# Every dictionary change requires one maintainer reviewer.
# A second reviewer (lawyer) is required by branch protection.
*    @twitchyvr
/src/**/*.dic @twitchyvr
/src/**/*.aff @twitchyvr
```

- [ ] **Step 9: Write LICENSE**

File: `LICENSE` — CC0 1.0 Universal full text (use [official](https://creativecommons.org/publicdomain/zero/1.0/legalcode.txt)).

- [ ] **Step 10: Commit + push + PR + merge**

```bash
git add .
git commit -m "feat: seed legal-augmentation dictionary with 50 starter terms and validator

Co-Authored-By: Claude <noreply@anthropic.com>"
git push -u origin feat/initial-seed
gh pr create --title "feat: seed dictionary with 50 starter terms" --body "Initial seed — Latin maxims and common legal shorthand. Full curation (5,000 terms) happens during M3. Validator runs in CI. CODEOWNERS enforces 2-reviewer policy. Closes M0 Task 8."
gh pr merge --squash --delete-branch --admin
```

---

## Task 9 — Create `feedback-intake` repo (issues-only)

**Files:**

- Create: `~/GitRepos/blackletter-studio/feedback-intake/README.md`
- Create: `~/GitRepos/blackletter-studio/feedback-intake/.github/ISSUE_TEMPLATE/config.yml` (block manual issues — only the Worker files here)

- [ ] **Step 1: Create + clone**

```bash
cd ~/GitRepos/blackletter-studio
gh repo create blackletter-studio/feedback-intake --public --description "Anonymous feedback intake (populated by the feedback Cloudflare Worker)" --clone
cd feedback-intake
git checkout -b feat/initial
```

- [ ] **Step 2: Write README**

```markdown
# feedback-intake

Public issue tracker populated automatically by the feedback Cloudflare Worker at https://feedback.blackletter.studio/api/submit.

**This is not a place for humans to file issues directly.** Use:

- [https://blackletter.studio/feedback](https://blackletter.studio/feedback) — public form, no GitHub account needed
- Product-specific repos (`fair-copy`, `legal-augmentation-dictionary`) — for technical issues

Issues that arrive here are triaged by maintainers and migrated to the relevant product repo.
```

- [ ] **Step 3: Disable manual issue creation**

File: `.github/ISSUE_TEMPLATE/config.yml`

```yaml
blank_issues_enabled: false
contact_links:
  - name: Feedback form (no GitHub account needed)
    url: https://blackletter.studio/feedback
    about: The right place for feature requests, bug reports, and questions.
  - name: Fair Copy bugs (technical)
    url: https://github.com/blackletter-studio/fair-copy/issues
    about: If you have a GitHub account and your bug is technical.
```

- [ ] **Step 4: Create labels used by the Worker**

```bash
gh label create "from-form" --description "Filed by the public feedback form" --color "fbca04"
gh label create "category: bug-fair-copy" --color "d73a4a"
gh label create "category: bug-proofmark" --color "d73a4a"
gh label create "category: feature" --color "0075ca"
gh label create "category: licensing" --color "7057ff"
gh label create "category: dictionary" --color "e4e669"
gh label create "category: other" --color "cfd3d7"
gh label create "needs-triage" --color "fbca04"
```

- [ ] **Step 5: Commit + push + PR + merge**

```bash
git add .
git commit -m "feat: seed feedback-intake repo with Worker-only issue policy

Co-Authored-By: Claude <noreply@anthropic.com>"
git push -u origin feat/initial
gh pr create --title "feat: initial feedback-intake seed" --body "Public intake tracker; manual issues disabled; labels matched to Worker's category taxonomy. Closes M0 Task 9."
gh pr merge --squash --delete-branch --admin
```

---

## Task 10 — Scaffold `infrastructure` (private) repo with Terraform + Cloudflare zone import

**Files:**

- Create: `~/GitRepos/blackletter-studio/infrastructure/README.md`
- Create: `~/GitRepos/blackletter-studio/infrastructure/terraform/main.tf`
- Create: `~/GitRepos/blackletter-studio/infrastructure/terraform/dns.tf`
- Create: `~/GitRepos/blackletter-studio/infrastructure/terraform/variables.tf`
- Create: `~/GitRepos/blackletter-studio/infrastructure/terraform/outputs.tf`
- Create: `~/GitRepos/blackletter-studio/infrastructure/terraform/.terraform-version`
- Create: `~/GitRepos/blackletter-studio/infrastructure/.github/workflows/tf-plan.yml`
- Create: `~/GitRepos/blackletter-studio/infrastructure/.gitignore`

- [ ] **Step 1: Create + clone**

```bash
cd ~/GitRepos/blackletter-studio
gh repo create blackletter-studio/infrastructure --private --description "Cloudflare + DNS-as-code for Black Letter" --clone
cd infrastructure
git checkout -b feat/initial-terraform
```

- [ ] **Step 2: Pin Terraform version**

File: `terraform/.terraform-version`

```
1.7.4
```

- [ ] **Step 3: Write `main.tf`**

```hcl
terraform {
  required_version = ">= 1.7"
  required_providers {
    cloudflare = { source = "cloudflare/cloudflare", version = "~> 4.0" }
  }
  backend "remote" {
    # Configure Terraform Cloud workspace separately via CLI (`terraform login` + `terraform init`).
    # Until then, local backend is used.
  }
}

provider "cloudflare" {
  # API token injected via TF_VAR_cloudflare_api_token environment variable
  api_token = var.cloudflare_api_token
}
```

- [ ] **Step 4: Write `variables.tf`**

```hcl
variable "cloudflare_api_token" {
  type        = string
  sensitive   = true
  description = "Scoped to blackletter.studio zone:edit permissions"
}

variable "cloudflare_zone_id" {
  type        = string
  description = "Zone ID for blackletter.studio (visible in Cloudflare dashboard → zone overview)"
}

variable "netlify_target" {
  type        = string
  description = "Netlify auto-assigned hostname for the site (e.g. friendly-curie-abc123.netlify.app)"
}
```

- [ ] **Step 5: Write `dns.tf`** (describes the records you already set manually; Terraform imports them in Step 8)

```hcl
resource "cloudflare_record" "apex" {
  zone_id = var.cloudflare_zone_id
  name    = "@"
  type    = "CNAME"
  value   = var.netlify_target
  proxied = true
  ttl     = 1
  comment = "Apex CNAME to Netlify (Cloudflare CNAME flattening enabled)"
}

resource "cloudflare_record" "www" {
  zone_id = var.cloudflare_zone_id
  name    = "www"
  type    = "CNAME"
  value   = var.netlify_target
  proxied = true
  ttl     = 1
}

# Reserved for future feedback Worker route
# resource "cloudflare_record" "feedback" {
#   zone_id = var.cloudflare_zone_id
#   name    = "feedback"
#   type    = "CNAME"
#   value   = "workers.dev"
#   proxied = true
#   ttl     = 1
# }
```

- [ ] **Step 6: Write `outputs.tf`**

```hcl
output "zone_id" {
  value = var.cloudflare_zone_id
}
```

- [ ] **Step 7: Write `.gitignore`**

```
.terraform/
*.tfstate
*.tfstate.backup
*.tfplan
terraform.tfvars
.env
```

- [ ] **Step 8: Import existing DNS records into Terraform state**

```bash
cd terraform
export TF_VAR_cloudflare_api_token="<token from Task 2 Step 7>"
export TF_VAR_cloudflare_zone_id="<zone id from Cloudflare dashboard>"
export TF_VAR_netlify_target="<netlify target from Task 6 Step 4>"
terraform init
terraform import cloudflare_record.apex "$TF_VAR_cloudflare_zone_id/<apex-record-id>"
terraform import cloudflare_record.www  "$TF_VAR_cloudflare_zone_id/<www-record-id>"
terraform plan
```

Expected: `terraform plan` shows `No changes. Your infrastructure matches the configuration.`

Record IDs are visible via `curl` to Cloudflare API:

```bash
curl -H "Authorization: Bearer $TF_VAR_cloudflare_api_token" \
     "https://api.cloudflare.com/client/v4/zones/$TF_VAR_cloudflare_zone_id/dns_records" | jq '.result[] | {name, id}'
```

- [ ] **Step 9: Write the plan-only CI workflow** (no apply from CI until M5 hardens OIDC)

File: `.github/workflows/tf-plan.yml`

```yaml
name: Terraform plan
on:
  pull_request:
    paths: ["terraform/**"]

jobs:
  plan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: hashicorp/setup-terraform@v3
        with: { terraform_version: "1.7.4" }
      - name: terraform fmt check
        working-directory: terraform
        run: terraform fmt -check -recursive
      - name: terraform init
        working-directory: terraform
        run: terraform init -backend=false
      - name: terraform validate
        working-directory: terraform
        run: terraform validate
```

(The plan step requires credentials; for M0 the workflow only validates syntax. Full apply-via-OIDC lands in M5 Task.)

- [ ] **Step 10: Write README**

```markdown
# infrastructure

Terraform-managed Cloudflare DNS + Workers routes for Black Letter Studio.

## Local use
```

cd terraform
export TF_VAR_cloudflare_api_token=...
export TF_VAR_cloudflare_zone_id=...
export TF_VAR_netlify_target=...
terraform init
terraform plan

```

## Making a change

1. Branch.
2. Edit `terraform/*.tf`.
3. Run `terraform fmt` and `terraform plan` locally.
4. Open PR — CI validates syntax.
5. Merge, then run `terraform apply` locally (CI-driven apply lands in M5).
```

- [ ] **Step 11: Commit + push + PR + merge**

```bash
git add .
git commit -m "feat: initial Terraform scaffold with Cloudflare DNS imported

Co-Authored-By: Claude <noreply@anthropic.com>"
git push -u origin feat/initial-terraform
gh pr create --title "feat: Terraform scaffold + DNS import" --body "Cloudflare zone + apex/www records imported into state. CI runs fmt + validate only — apply is local-only until M5 adds OIDC. Closes M0 Task 10."
gh pr merge --squash --delete-branch --admin
```

---

## Task 11 — Migrate planning-repo contents into `.github` repo

**Context:** The original planning directory at `~/GitRepos/blackletter-studio/` (containing this plan and the design spec on branch `docs/initial-design-spec`) needs a permanent home. The cleanest target is the `.github` org-defaults repo, under `docs/superpowers/`. This also makes the planning artifacts visible on the org page.

- [ ] **Step 1: Move files from planning repo to `.github-org` working copy**

```bash
cd ~/GitRepos/blackletter-studio/.github-org
git checkout -b docs/migrate-planning-artifacts
mkdir -p docs/superpowers
cp -R ../docs/superpowers/specs docs/superpowers/
cp -R ../docs/superpowers/plans docs/superpowers/
```

- [ ] **Step 2: Commit + push + PR + merge**

```bash
git add docs
git commit -m "docs: migrate planning specs and plans from planning repo

Co-Authored-By: Claude <noreply@anthropic.com>"
git push -u origin docs/migrate-planning-artifacts
gh pr create --title "docs: migrate planning artifacts" --body "Moves the design spec and M0 plan from the temporary planning directory at ~/GitRepos/blackletter-studio/ into the .github org-defaults repo where they're visible org-wide. Closes M0 Task 11."
gh pr merge --squash --delete-branch --admin
```

- [ ] **Step 3: Archive the planning directory**

```bash
mv ~/GitRepos/blackletter-studio/.git ~/GitRepos/blackletter-studio/.git.archived-m0
```

(Keeps the old history recoverable; the `.github` repo is now canonical. Remove `.git.archived-m0` after user confirms the migration.)

---

## Task 12 — Enable branch protection on `main` across all new repos

**Files:** none (GitHub API calls only)

- [ ] **Step 1: For each public repo, enable branch protection via `gh api`**

```bash
for REPO in site fair-copy legal-augmentation-dictionary feedback-intake .github; do
  gh api --method PUT "repos/blackletter-studio/$REPO/branches/main/protection" \
    --field required_status_checks[strict]=true \
    --field required_status_checks[contexts][]="ci" \
    --field enforce_admins=true \
    --field required_pull_request_reviews[required_approving_review_count]=1 \
    --field required_pull_request_reviews[require_code_owner_reviews]=true \
    --field restrictions=null \
    --field required_linear_history=true \
    --field allow_force_pushes=false \
    --field allow_deletions=false \
    --field required_signatures=true 2>/dev/null || echo "skipped $REPO (no main yet)"
done
```

- [ ] **Step 2: For private repos (`brand`, `infrastructure`), same commands**

```bash
for REPO in brand infrastructure; do
  gh api --method PUT "repos/blackletter-studio/$REPO/branches/main/protection" \
    --field required_status_checks[strict]=true \
    --field enforce_admins=true \
    --field required_pull_request_reviews[required_approving_review_count]=1 \
    --field required_linear_history=true \
    --field allow_force_pushes=false \
    --field allow_deletions=false \
    --field required_signatures=true 2>/dev/null || echo "skipped $REPO"
done
```

- [ ] **Step 3: Verify by attempting a direct push to main** (should fail)

```bash
cd ~/GitRepos/blackletter-studio/site
git checkout main
git commit --allow-empty -S -m "test: should fail"
git push origin main
```

Expected: `remote: Protected branch: ... not permitted to push`. That's the pass condition.

Revert the empty commit: `git reset --hard HEAD~1 && git push` (which will also be blocked). This is fine — the commit doesn't escape locally for long.

---

## Task 13 — Final verification and M0 completion status

**Files:**

- Create: `~/GitRepos/blackletter-studio/.github-org/docs/superpowers/plans/m0-completion-report.md` (written after all tasks green)

- [ ] **Step 1: Run the M0 completion checklist**

Verify each of these end-to-end, recording result below:

- [ ] `dig NS blackletter.studio` returns Cloudflare nameservers.
- [ ] `curl -I https://blackletter.studio` returns 200 with HSTS, X-Frame-Options, X-Content-Type-Options headers.
- [ ] Browser-open https://blackletter.studio → placeholder home page renders with Fraunces display font, IBM Plex Sans body, oxblood accent, ivory background.
- [ ] Browser-open https://blackletter.studio/fair-copy, /proofmark, /trust, /feedback, /support, /legal/eula, /legal/privacy → all render without 404.
- [ ] `gh repo list blackletter-studio` shows: `.github`, `brand`, `site`, `fair-copy`, `legal-augmentation-dictionary`, `feedback-intake`, `infrastructure` (7 repos).
- [ ] Branch protection verified on `main` of all seven repos (direct push fails).
- [ ] `pnpm start:word` in `fair-copy` sideloads the add-in and the task pane shows the Fair Copy placeholder.
- [ ] `pnpm test && pnpm typecheck && pnpm lint` in `fair-copy` all pass.
- [ ] `pnpm build && pnpm typecheck` in `site` all pass.
- [ ] `python3 scripts/validate.py` in `legal-augmentation-dictionary` prints `ok: ...`.
- [ ] `terraform plan` in `infrastructure/terraform` shows `No changes`.

- [ ] **Step 2: Write the completion report**

File: `.github-org/docs/superpowers/plans/m0-completion-report.md`

```markdown
# M0 — Completion Report

**Date completed:** <fill in>
**Tasks executed:** 13 of 13
**Outstanding issues:** <none | list>

## Verification results

- [x] Cloudflare DNS live for blackletter.studio
- [x] Placeholder site deployed on Netlify, HTTPS + security headers confirmed
- [x] Seven repos created under blackletter-studio org
- [x] Branch protection on main across all seven repos
- [x] Fair Copy add-in sideloads into Word and renders the placeholder
- [x] Legal dictionary seeded with 50 terms and validator passing
- [x] Infrastructure Terraform state imported; plan is clean
- [x] Planning artifacts migrated to .github repo; planning directory archived

## Artifacts

- Design spec: `.github/docs/superpowers/specs/2026-04-16-blackletter-studio-design.md`
- M0 plan: `.github/docs/superpowers/plans/2026-04-16-m0-scaffold-and-infrastructure.md`

## Next milestone

**M1 — Marketing site alpha** — writing-plans skill will produce the next plan file at `docs/superpowers/plans/<date>-m1-marketing-site-alpha.md`.
```

- [ ] **Step 3: Commit the completion report**

```bash
cd ~/GitRepos/blackletter-studio/.github-org
git checkout -b docs/m0-completion
git add docs/superpowers/plans/m0-completion-report.md
git commit -m "docs: M0 completion report

Co-Authored-By: Claude <noreply@anthropic.com>"
git push -u origin docs/m0-completion
gh pr create --title "docs: M0 completion report" --body "Milestone M0 complete. See report for verification details."
gh pr merge --squash --delete-branch --admin
```

---

## Summary

After Task 13, the studio is scaffolded:

- `blackletter.studio` serves a placeholder site via Netlify, behind Cloudflare (HTTPS, HSTS, security headers).
- All seven repos live under `github.com/blackletter-studio`, with shared CI, CODEOWNERS, and branch protection.
- The Fair Copy add-in sideloads into local Word and renders a placeholder task pane.
- The legal dictionary has a seed vocabulary and a passing validator.
- Cloudflare DNS is managed as code via Terraform.
- Planning artifacts live in the org-defaults `.github` repo.

**Zero product logic** was written. That's M2's job. **M1 is next: real content on the marketing pages, the feedback Worker filing real issues, and the email signup live.**
