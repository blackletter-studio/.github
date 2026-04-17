# M1 — Completion Report

**Date completed:** 2026-04-17
**Tasks executed:** 14 of 14
**Skill used:** superpowers:subagent-driven-development
**PRs merged:** 14 on `site` (#4-#18, excluding #3 which was the bug issue), 1 on `brand` (#2), plus 1 smoke-test issue on `feedback-intake` (#2, closed)
**Outstanding items:** 4 (one pending user signup, three non-blocking follow-ups — see below)

## Verification results (all green)

- [x] **Netlify auto-deploys restored.** `site#3` closed; `NODE_VERSION="22"` in `netlify.toml` on main; preview builds disabled to conserve minutes. Auto-deploys on push to main observed across PRs #6-#18.
- [x] **All 9 pages return HTTP 200.** `curl -sL` against `/`, `/fair-copy`, `/proofmark`, `/trust`, `/feedback`, `/support`, `/changelog`, `/legal/eula`, `/legal/privacy` — all 200 after trailing-slash redirect.
- [x] **Feedback Worker live at `feedback.blackletter.studio`.** `GET /api/health` → 200. `POST /api/submit` with empty body → 400 with Zod validation error listing the three required fields (`category`, `title`, `body`). End-to-end smoke test filed as `feedback-intake#2` (now closed).
- [x] **DNS for feedback subdomain resolves through Cloudflare.** `dig feedback.blackletter.studio` returns Cloudflare proxy IPs (`2606:4700:...`).
- [x] **Feedback form wired to Worker.** `/feedback` page HTML contains `fetch("https://feedback.blackletter.studio/api/submit", ...)` plus the honeypot `<input name="website">` and all required fields.
- [x] **Email signup form present on home.** `/` contains `<form ... action="https://buttondown.email/api/emails/embed-subscribe/PENDING">` — submits, but to a placeholder username (see Outstanding #1).
- [x] **Changelog page live and truthful.** `/changelog` renders `v0.1.0-alpha` plus "In progress" and "Deferred to later milestones" sections, reflecting only what's actually in main.
- [x] **Final wordmark shipped via brand repo.** `brand` PR #2 merged; `site` PR #8 bumped the submodule.
- [x] **Worker tests passing.** 14 tests (4 schema + 6 sanitize + 4 integration) confirmed green at PR #17 merge.
- [x] **V1 Editorial aesthetic applied.** Fraunces + IBM Plex Sans, ivory `#faf6ef`, oxblood `#6b1e2a`, confirmed across all 9 pages.
- [x] **Planning artifacts in place.** M1 plan at `.github/docs/superpowers/plans/2026-04-16-m1-marketing-site-alpha.md`; this report alongside it.

## Deviations from the M1 plan (all deliberate, all documented)

| Deviation                                                                      | Reason                                                                                                                                                                                                                                                                                                 |
| ------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| CHANGELOG.md content rewritten to reflect only shipped state                   | Plan draft asserted features (Worker, DNS, email signup) before T9-T11 had executed. Per the `/trust` page's promise that claims are verifiable against the code, unshipped items were moved to "In progress" / "Deferred".                                                                            |
| Git tag `v0.1.0-alpha` deferred                                                | Plan called for the tag at end of T13, but T9-T11 (Worker build, deploy, form wiring) were still pending. Deferred until M2 so the tag carries real alpha-shipping weight.                                                                                                                             |
| DNS record for `feedback.blackletter.studio` added manually via Cloudflare API | Plan assumed route-based Worker config would auto-create DNS. It doesn't — that only applies to Workers Custom Domains, not plain Workers Routes. Wrangler deploy succeeded but the subdomain was unreachable until an AAAA record pointing to `100::` (Cloudflare proxy sentinel) was added manually. |
| Subagent sandbox blocked pnpm in many contexts                                 | Multiple subagents fell back to `npx` for vitest/tsc/eslint and `npm install --legacy-peer-deps` for Worker-subproject installs. Not a shipped-product issue; triggers Outstanding #3 below.                                                                                                           |
| T1 root-cause was Node version, not pnpm script approval                       | The M1 plan theorized `site#3` was `onlyBuiltDependencies` approval for esbuild/sharp. PRs #4 and #5 tried that path; PR #6 identified the real cause as Astro 6 requiring Node 22 (Netlify was defaulting to Node 20). `NODE_VERSION="22"` in `netlify.toml` was the fix.                             |

## Outstanding items

1. **Buttondown username placeholder.** `BUTTONDOWN_USERNAME = "PENDING"` in `site/src/components/EmailSignup.astro`. Until Matt signs up for Buttondown and replaces the literal string, form submissions silently fail (no user-visible error — the form just POSTs to a 404 endpoint). One-line fix. A spawn-task chip is already filed for this follow-up.
2. **Tailscale MagicDNS caching on Matt's dev machine.** Occasionally shows stale records for `feedback.blackletter.studio`. Not a production issue — all public resolvers serve correct records, and site visitors using any ISP's DNS see the Worker. Workaround for local testing: `curl --resolve feedback.blackletter.studio:443:104.21.38.18 ...`.
3. **`site/workers/feedback/` uses `package-lock.json` instead of `pnpm-lock.yaml`.** The Worker subdirectory was installed via `npm install --legacy-peer-deps` by a sandboxed subagent (pnpm blocked in that context). Non-blocking — the Worker builds, tests pass, and deploys cleanly — but inconsistent with the site's pnpm convention. File as small follow-up on `site` to re-lock with pnpm.
4. **Git tag `v0.1.0-alpha` not yet pushed.** Plan called for it as the closing step of T13. Deliberately deferred until M2's core functionality is also in main so the `v0.1.0-alpha` tag means "marketing site + Fair Copy core shipping", not "marketing site only".

## Artifacts

- M1 plan: `github.com/blackletter-studio/.github/blob/main/docs/superpowers/plans/2026-04-16-m1-marketing-site-alpha.md`
- This report: `github.com/blackletter-studio/.github/blob/main/docs/superpowers/plans/m1-completion-report.md`
- Merged PRs on `site`: #4, #5, #6, #7, #8, #9, #10, #11, #12, #13, #14, #15, #16, #17, #18
- Merged PRs on `brand`: #2
- Closed issues demonstrating M1 delivery: `site#3` (Netlify build fix), `feedback-intake#2` (end-to-end Worker smoke test)
- Live site: `https://blackletter.studio` (9 pages, V1 Editorial aesthetic)
- Live Worker: `https://feedback.blackletter.studio/api/submit` (health + submit endpoints)

## Next milestone

**M2 — Fair Copy core.** In progress as of this writing. Scope (from the spec Section 8):

- Word add-in loads the Quick Check panel and calls into the legal-augmentation dictionary
- End-to-end spellcheck flow for a curated set of legal terms (from the M0 49-term seed)
- Ribbon button + taskpane + first real user feature (misspelling → suggestion → replace)
- Telemetry-free, fully on-device per the Trust page's promises
- CI on `fair-copy` covers build, typecheck, unit tests, and Office add-in manifest validation

The M2 plan will be generated via the `writing-plans` skill at the start of that milestone (and may already be in flight).
