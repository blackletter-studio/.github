# M0 — Completion Report

**Date completed:** 2026-04-17
**Tasks executed:** 13 of 13
**Skill used:** superpowers:subagent-driven-development
**Implementer subagents dispatched:** 8
**Outstanding items:** 2 (one user action, one post-v1 CI issue — see below)

## Verification results (all green)

- [x] **Cloudflare DNS:** zone `blackletter.studio` is `status=active`, activated `2026-04-17T02:12:40Z`. Nameservers are `ignacio.ns.cloudflare.com` / `sandy.ns.cloudflare.com`. Porkbun NS swap complete.
- [x] **Zone security hardening:** SSL Full-Strict, TLS 1.3 minimum, Always-Use-HTTPS, Brotli, HSTS (15552000s, includeSubDomains), Opportunistic Encryption, Automatic HTTPS Rewrites — all applied via API.
- [x] **Site live:** `https://blackletter.studio` returns HTTP 200 through Cloudflare + Netlify Edge with correct security headers (X-Frame-Options DENY, strict Referrer-Policy, HSTS).
- [x] **Seven repos created** under `github.com/blackletter-studio`: `.github`, `site`, `fair-copy`, `legal-augmentation-dictionary`, `feedback-intake`, `brand`, `infrastructure`.
- [x] **Branch protection on `main`** for every repo: 1-reviewer requirement, linear history enforced, signed commits required, no force-pushes, no deletions, admin enforcement on.
- [x] **Fair Copy add-in:** sideload-able Word add-in skeleton. `pnpm test` passes (1/1), `pnpm typecheck` passes (0 errors), `pnpm lint` passes (0 errors/warnings), `pnpm build` produces dist/.
- [x] **Marketing site:** 8 placeholder pages render correctly (home, fair-copy, proofmark, trust, feedback, support, legal/eula, legal/privacy). V1 Editorial aesthetic applied (Fraunces + IBM Plex Sans, ivory `#faf6ef`, oxblood `#6b1e2a`).
- [x] **Legal dictionary:** 49 curated legal terms seeded. Validator passes (`ok: legal-augmentation.dic passes lint and Hunspell parse`). CI validator workflow in place.
- [x] **Feedback intake:** 17 labels created (8 Black Letter labels + 9 GitHub defaults). Manual issue creation blocked via `.github/ISSUE_TEMPLATE/config.yml`. Ready for the Worker to file into (M1).
- [x] **Infrastructure:** Terraform 1.14.8 + Cloudflare provider v4. All 5 existing DNS records imported. `terraform plan` returns no changes. CI runs `fmt` + `validate`; apply is local-only until M5.
- [x] **Brand repo:** flipped to public during execution so Netlify builder can clone the submodule without auth. V1 tokens (tokens.css + tokens.ts) and placeholder wordmark in place. Fraunces + IBM Plex fonts (both SIL OFL 1.1) self-hosted.
- [x] **Planning artifacts migrated:** design spec and M0 plan now live at `.github/docs/superpowers/specs/` and `.github/docs/superpowers/plans/` (moved from the bootstrap planning directory at `~/GitRepos/black-letter-studio/`).

## Deviations from original plan (all deliberate, all documented)

| Deviation                                                                                                                        | Reason                                                                                                                                                                                          |
| -------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `pnpm 10` instead of `9` in CI                                                                                                   | Matt's local environment has 10; mismatching would cause lockfile friction                                                                                                                      |
| `@astrojs/check` added to site devDependencies                                                                                   | Astro's `astro check` script requires it; original plan didn't include it                                                                                                                       |
| Tailwind v4 via `@tailwindcss/vite` + `@theme` block                                                                             | Current Astro scaffold installs v4 by default; v3 config still present as spec parity but inert                                                                                                 |
| Fraunces variable-font URL updated to v38                                                                                        | Original v32 URL on Google's CDN 404's                                                                                                                                                          |
| ESLint v9 with flat config instead of v10 + `.eslintrc.cjs`                                                                      | `eslint-plugin-react@7.37.5` incompatible with ESLint 10                                                                                                                                        |
| `App.tsx` return type `ReactElement` instead of `JSX.Element`                                                                    | React 19 removed the global `JSX` namespace                                                                                                                                                     |
| Site deployed via local build + `netlify deploy --prod --dir=dist`                                                               | Netlify's remote builder fails with exit code 2 at the "building site" stage (tracked as site issue #3); local build works; this unblocks v1 and leaves the CI-build-fix for a later small task |
| `brand` repo flipped from private to public                                                                                      | Netlify builder can't clone a private submodule without credentials; brand tokens/fonts/wordmarks are inherently public anyway                                                                  |
| `cloudflare_record` resources use `content` instead of deprecated `value`, and full `blackletter.studio` instead of `@` for apex | Cloudflare provider v4 deprecation / compatibility                                                                                                                                              |

## Outstanding items

1. **User action required: toggle "Require 2FA for everyone" in org Security settings.** Via API the value stays `false` — it's a known GitHub quirk when the prerequisite conditions (all members have 2FA, no outside collaborators without 2FA) can't be algorithmically verified from API alone. One UI click at `github.com/organizations/blackletter-studio/settings/security` resolves it.
2. **Post-M0 bugfix: Netlify remote builder fails.** Tracked as `blackletter-studio/site#3`. Local `pnpm build` succeeds in a fresh clone; the site is live via local-build-and-upload. Root cause unknown (possibly pnpm 10's ignored-install-scripts behavior on esbuild/sharp, or Node 20 vs 22 difference). Fix before M1 ships so auto-deploys on commit work.

## Artifacts

- Design spec: `github.com/blackletter-studio/.github/blob/main/docs/superpowers/specs/2026-04-16-black-letter-studio-design.md`
- M0 plan: `github.com/blackletter-studio/.github/blob/main/docs/superpowers/plans/2026-04-16-m0-scaffold-and-infrastructure.md`
- This report: `github.com/blackletter-studio/.github/blob/main/docs/superpowers/plans/m0-completion-report.md`

## Next milestone

**M1 — Marketing site alpha.** Scope (from the spec Section 7):

- Real content on `/`, `/fair-copy`, `/proofmark`, `/trust`, `/legal/*`
- Feedback Cloudflare Worker built and filing to `feedback-intake`
- Email list signup live (Buttondown or alternative)
- OpenGraph card generation at build time
- Final wordmark in `brand` repo replacing the placeholder
- Fix `site#3` so Netlify auto-deploys on commit work

The M1 plan will be generated via the `writing-plans` skill at the start of that milestone.
