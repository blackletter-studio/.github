# Black Letter Studio — Design Specification

> **Status**: Draft, awaiting user review
> **Date**: 2026-04-16
> **Author**: Matt Rogers (with Claude as design partner)
> **Spec type**: Studio + first product (v1.0)
> **Approval gate**: User must approve this document before any implementation planning begins

---

## Executive Summary

**Black Letter** is a tool studio shipping simple, elegant desktop utilities for legal professionals. Launch product is a single Microsoft Word add-in containing two complementary tools:

1. **Fair Copy** — strips formatting noise from a DOCX while preserving emphasis (bold / italic / underline) and paragraph alignment. Reduces an inherited, visually chaotic document to a clean, house-style-ready base in one click.
2. **Proofmark** — flags probable typographical errors with a distinct highlighter-pen treatment (amber highlight, dotted-underline fallback) that cannot be confused with Word's native tracked-changes redline markup. Runs entirely client-side via Hunspell-WASM with a curated legal-augmentation dictionary.

The business is an indie tool studio, not a venture-scale SaaS. Pricing is a single one-time purchase of **$49 USD** granting perpetual use of the v1 product family, sold through Microsoft AppSource. The add-in is **zero-telemetry**: no analytics, no error reporting with document content, no cloud spellcheck, no phone-home for anything other than one license-entitlement check against Microsoft Graph. The marketing face of the studio is `blackletter.studio`, hosted on Netlify behind Cloudflare; the GitHub org is `blackletter-studio` (free tier).

The design direction is refined-editorial — classical legal + typographic craft — built on Fraunces + IBM Plex Sans, ivory (`#faf6ef`), oxblood (`#6b1e2a`), and ink black (`#1a1a1a`). The working UI inside Word is one large "Clean" button with all granular settings hidden behind an Advanced panel: decoration lives on the site and splash screen; the tool itself is nearly brutalist in its simplicity.

Target launch: **v1.0 public release 10–14 weeks from scaffold start**, pending AppSource review cadence and one-time legal review of EULA and privacy policy.

---

## Goals & Non-Goals

### Goals

- Solve one specific lawyer-expressed frustration ("strip everything except bold, italic, underlined, and paragraph alignment") with craftsmanship appropriate to the audience.
- Establish a studio shell — brand, site, repo structure, trust posture — that future tools can be added to without rework.
- Earn a trust signal from legal professionals via zero-telemetry, client-side processing, perpetual licenses, and an auditable security posture.
- Operate at low fixed cost (no recurring servers required for v1; AppSource covers billing).
- Give lawyers a non-account-required path to submit feedback and feature requests that auto-creates tracked GitHub issues.

### Non-Goals (v1)

- Not a grammar checker, style editor, plain-language rewriter, or AI assistant. Proofmark flags; it never corrects.
- Not a subscription. No recurring revenue mechanics, no account system, no tier proliferation.
- Not multi-seat or enterprise licensing. Individual licenses only at launch; firm licensing deferred.
- Not a replacement for Microsoft's spellcheck or tracked-changes system. Coexists with both, designed explicitly not to visually collide.
- Not cross-platform beyond what Office.js gives us for free (Mac Word, Windows Word, Web Word, iPad Word). The standalone desktop app is a later milestone, not v1.
- Not AI-powered. No LLM calls, no ML model, no cloud inference. The value is precision and restraint, not intelligence.

---

## Section 1 — Studio & Product Lineup

### Studio identity

| Field                  | Value                                                                                                                                           |
| ---------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| **Studio name**        | Black Letter                                                                                                                                    |
| **Tagline direction**  | _"Simple tools for the considered practice"_ (stub — refine during M1)                                                                          |
| **Voice**              | Refined, classical, understated authority. Hoefler & Co. meets Panic. Warm but serious.                                                         |
| **Primary domain**     | `blackletter.studio` (registered via Porkbun, user-owned)                                                                                       |
| **GitHub org**         | `github.com/blackletter-studio` (free tier, owned by `twitchyvr`, member visibility: private)                                                   |
| **Primary audience**   | Solo practitioners, small-firm lawyers, paralegals, and legal assistants who live in Microsoft Word and routinely clean up inherited DOCX files |
| **Secondary audience** | Firm IT staff who evaluate and procure desktop utilities; legal-technology journalists and bar-association CLE programs                         |

### Launch product lineup

One Office Word add-in containing two named commands, marketed on the site as two products in the Black Letter lineup.

| Tool          | Primary job                                                                                                                        | Mode of use                                                                            |
| ------------- | ---------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------- |
| **Fair Copy** | Strip formatting noise from a DOCX; preserve bold, italic, underline, and alignment. Confirm before destructive operations.        | Open document → click Clean → done.                                                    |
| **Proofmark** | Flag probable typos with a highlighter-pen treatment that does not collide with tracked-changes markup. Runs entirely client-side. | Open document → click Proof → review flagged words in the task pane → decide per word. |

**Packaging**: one Office.js manifest, two ribbon commands, one install, one license. Distributed exclusively through Microsoft AppSource.

### Studio posture on future tools

Not committed in v1, tracked as plausible tool slots informed by adjacent lawyer pain points:

- Metadata scrubber (strip author history, last-edited-by, tracked changes trail before sending)
- Exhibit renamer + Bates stamper
- Citation normalizer (Bluebook / ALWD auto-formatting)
- Defined-terms auditor (terms defined but unused, or used but undefined)
- Pleading-paper converter (court-specific margins, line numbering)
- Signature-block stripper (clean up forwarded correspondence)

Decision gate for tool #2: 90 days post v1 launch, informed by feedback-form submissions. Never guess at tool #2 ahead of signal.

---

## Section 2 — Fair Copy Behavior & UI

### The one-button promise

First-time experience inside the Word task pane is one primary action. No visible toggles.

```
┌─ Black Letter · Fair Copy ─┐
│                            │
│   Preset: [ Standard  ▼ ]  │
│                            │
│       ┌───────────┐        │
│       │   Clean   │        │
│       └───────────┘        │
│                            │
│  ▸ Advanced settings       │
│                            │
│  ┌──────────────────────┐  │
│  │ Proofmark  →         │  │
│  └──────────────────────┘  │
└────────────────────────────┘
```

The 20+ granular configuration toggles exist behind the Advanced chevron. First-time lawyer clicks Clean and the Standard preset runs against the active document.

### Presets

| Preset                 | Stance                          | Summary                                                                                                                                                                                                                                                                                                                                                                                                                         |
| ---------------------- | ------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Standard** (default) | Clean but careful               | Strips font faces, font sizes, font colors, highlighting, comments. Normalizes indents / tab stops, bullet and numbered list styles, line spacing, paragraph spacing. **Keeps** bold, italic, underline, paragraph alignment, table structure, footnotes, hyperlinks (as links, formatting stripped), section breaks, images (with warning), headers and footers, strikethrough. **Rejects tracked changes with confirmation**. |
| **Conservative**       | Lightest touch                  | Normalizes only font faces and font colors. Leaves everything else intact. For docs the user wants minimally processed.                                                                                                                                                                                                                                                                                                         |
| **Aggressive**         | Maximum flatten                 | Strips everything except bold / italic / underline and paragraph alignment. Flattens lists to plain paragraphs, removes images (with warning), inlines footnotes, strips section breaks. Still confirms on tracked changes.                                                                                                                                                                                                     |
| **Custom**             | Any user override from a preset | Saved automatically when the user changes any Advanced setting. Exportable as JSON for sharing across devices (future feature).                                                                                                                                                                                                                                                                                                 |

### Per-rule settings (Advanced panel)

Each rule exposes three-to-four user options with a committed default. Defaults reflect the "do no harm" philosophy.

| Rule                  | Options                                                                                 | Default                                                                                                                                                                                                             |
| --------------------- | --------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Font face             | Keep as-is / Change all body text to one configurable font                              | Change to one body font                                                                                                                                                                                             |
| Font size             | Keep as-is / Set specific / Normalize / One body size with heading preservation         | One body size, preserve heading & title styles                                                                                                                                                                      |
| Colored text          | Keep as-is / Convert all to black                                                       | Convert to black                                                                                                                                                                                                    |
| Highlighting          | Remove existing / Change color to avoid conflict with Proofmark / Keep as-is            | Remove existing (unless user enables "coexist with document highlights")                                                                                                                                            |
| **Bold**              | Keep / Strip (individually toggleable)                                                  | **Keep**                                                                                                                                                                                                            |
| **Italic**            | Keep / Strip (individually toggleable)                                                  | **Keep**                                                                                                                                                                                                            |
| **Underline**         | Keep / Strip (individually toggleable)                                                  | **Keep**                                                                                                                                                                                                            |
| Strikethrough         | Keep / Strip                                                                            | Keep                                                                                                                                                                                                                |
| Paragraph alignment   | Keep as-is / Set all to specific alignment                                              | **Keep as-is**                                                                                                                                                                                                      |
| Line spacing          | Keep as-is / Single / 1.15 / 1.5 / Double                                               | 1.15                                                                                                                                                                                                                |
| Paragraph spacing     | Keep as-is / Consistent / Set specific                                                  | Consistent                                                                                                                                                                                                          |
| Indents & tab stops   | Keep as-is / Normalize / Convert to tabs / Convert to spaces                            | Normalize (structural indents preserved; arbitrary tab stops normalized — heuristic described below)                                                                                                                |
| Bullet lists          | Keep as-is / Normalize bullet style / Strip to paragraphs                               | Normalize bullet style                                                                                                                                                                                              |
| Numbered lists        | Keep as-is / Normalize style / Strip to paragraphs                                      | Normalize style                                                                                                                                                                                                     |
| Tables                | Keep as-is / Normalize borders and shading / Convert to plain text (content-preserving) | Normalize borders and shading                                                                                                                                                                                       |
| Images                | Keep all / Remove all / **Warn and list**                                               | **Warn and list** (user chose "Remove" originally; override to "Warn and list" because silent removal is destructive — user confirmed A+D philosophy favors transparency; setting flagged in review for final call) |
| Headers / footers     | Keep as-is / Normalize / Strip                                                          | Keep                                                                                                                                                                                                                |
| Footnotes / endnotes  | Keep as-is / Inline into body text / Strip                                              | Keep                                                                                                                                                                                                                |
| Comments              | Keep / Strip                                                                            | Strip                                                                                                                                                                                                               |
| Tracked changes       | Accept all / Reject all / Leave as-is                                                   | Reject all **with forced confirmation** (never silent)                                                                                                                                                              |
| Hyperlinks            | Keep link, strip formatting / Strip entirely                                            | Keep link, strip formatting                                                                                                                                                                                         |
| Section / page breaks | Keep / Strip                                                                            | Keep                                                                                                                                                                                                                |

#### Note on the image default — flagged for user confirmation in review

The user's original answer specified "Remove" as the image default. During design, a pushback suggested "Warn and list" as safer (lawyers' DOCX files routinely contain signature-block images, seals, letterheads, exhibit thumbnails — silent removal of these is high-regret). The spec currently reflects the safer default but is **subject to user reversion** during spec review. If the user reverts to "Remove," the rule still triggers a one-time confirmation dialog on first-ever use of that preset, after which it runs silently.

#### Heuristic: structural vs arbitrary indents

The "Normalize" action on indents distinguishes between:

- **Structural indents** — block quotes, numbered sub-paragraphs, list continuation indents, hanging indents on Bluebook-style citations. **Preserved**.
- **Arbitrary tab stops** — custom tab positions used to visually align homemade pseudo-tables, center decorative whitespace, or emulate columns. **Normalized** to the body's default.

This heuristic lives in a dedicated rule module and ships with a **unit-test corpus** of representative legal-document samples. If classification fails on a real document, the user can toggle the direct setting in Advanced.

### Safe-by-default destructive-operation confirmations

Three operations always surface a confirmation modal, regardless of preset:

1. **Tracked changes present** → _"This document has {N} tracked changes. Reject them all before cleaning? [Review first] [Reject & clean] [Leave as-is]."_
2. **Images present** → _"This document contains {N} images ({list of page + detected type: signature / letterhead / exhibit / unknown}). Keep all / Remove all / Choose individually."_ Pre-selected to preset preference.
3. **Document marked Final, password-protected, or has active comments** → Warning dialog listing what is present.

These confirmations are intentional friction. They cannot be hidden behind a "don't show again" checkbox. Lawyers gain trust when the tool refuses to silently do destructive things.

### Proofmark — highlight system

- **Scan scope**: document body by default. Advanced toggle extends to headers, footers, comments, footnotes.
- **Engine**: Hunspell compiled to WebAssembly. Custom build recipe published publicly for audit (part of the `legal-augmentation-dictionary` repo).
- **Dictionary stack** (loaded in order; later dictionaries augment earlier):
  1. `en_US` standard Hunspell dictionary (open-source, ~50k entries, ships bundled)
  2. `legal-augmentation.dic` — curated legal-domain dictionary (~5,000 seed terms at launch: Latin maxims, US federal + state court names, Bluebook abbreviations, Bates tokens, common legal terminology, signing-formula words). Maintained publicly on GitHub; accepts community PRs with a two-reviewer gate.
  3. User's personal dictionary — "Add to my dictionary" persists to `Office.context.roamingSettings` (syncs across the user's devices via their Microsoft 365 account).
- **Visual treatment**: soft amber highlight applied to the `Word.Range` of each detected potential typo. Amber chosen to not collide with Word's red squiggle (native spellcheck), red tracked-changes markup, or typical document highlights. **Conflict detection**: if amber is already in use in the document, Proofmark automatically falls back to a **dotted underline** treatment. User can override the color in Advanced settings.
- **Task pane UI**: detected items appear as a list. Each row shows the word in surrounding context (~8 words) and four quick actions:
  - _Add to my dictionary_
  - _Ignore once_ (clears the mark for this single occurrence)
  - _Ignore in this doc_ (clears all occurrences in the current document; persists to `Office.context.document.settings`)
  - _Go to word_ (scrolls the document to that location)
- **Explicit non-feature**: No "Fix all" button. No auto-correct suggestions. Proofmark flags; the lawyer writes. This is a deliberate trust decision.

### Trial counter and preview mode

- **Counter card** — pinned above the Clean button while the user is in trial: _"3 of 5 free cleans remaining"_ with a small progress arc.
- **Counter semantics**: cumulative, not time-boxed. A single Clean invocation increments the counter by one, regardless of document size. "Cancel" inside a confirmation dialog does not increment. A run that produces no changes (document was already clean) does not increment.
- **Counter storage**: `Office.context.roamingSettings`, signed with an HMAC-SHA256 key derived from the user's Microsoft 365 tenant ID plus a per-install random seed generated on first launch. Tamper detection = signature verification fails → fail-safe to "trial exhausted" state.
- **After 5 full cleans — Preview mode** becomes the default:
  - Counter card replaced with: _"Preview mode — see what would change. License to save."_
  - "Clean" button becomes _"Preview diff"_ — renders what would change as in-document highlights (inserted-text markers on what would be added, strikethrough on what would be removed). **Document is not modified; output is not saved.**
  - Clear **"Unlock — $49 lifetime"** call-to-action in the same visual position the Clean button occupied.
  - **Proofmark remains at full fidelity** in Preview mode (zero marginal cost since it's all client-side; helps users experience the quality permanently).
- License purchase flow: AppSource checkout opens in the user's Office-account context. On success, the add-in revalidates entitlement via Microsoft Graph, caches a signed JWT locally, task pane swaps to the licensed layout.

### First-open onboarding

Task pane shows a three-card carousel on first launch:

1. _"**Fair Copy** strips formatting noise. Keeps what you meant — bold, italic, underline, alignment."_
2. _"**Proofmark** finds likely typos with a highlighter that won't collide with your tracked changes."_
3. _"Your first 5 cleans are on the house. No account, no sign-up, no telemetry."_

CTA: _"Open my first document."_ Target time-to-first-Clean: under 10 seconds.

### Explicit non-behaviors (pinned)

- Never auto-corrects spelling.
- Never accepts tracked changes (only rejects with confirmation, or leaves alone).
- Never phones home. No analytics, no error reporting containing document content.
- Never touches the document's revision history (Word's built-in versioning).
- Never processes `.doc` (legacy binary format). Legacy files produce a clear error message pointing at Word's "Save As → .docx" conversion.
- Never modifies a document in Preview mode.

---

## Section 3 — Licensing, Privacy & Longevity

### License terms

- **Single one-time purchase: $49 USD.** No subscription. No auto-renewal.
- **Lifetime of the v1 product family.** Works forever on the version line purchased, including all minor and patch updates. A future v2 with major architectural changes would be a separate AppSource offer; v1 buyers keep v1 indefinitely.
- **Sold through Microsoft AppSource** for the add-in (v1). Microsoft handles billing, trials, enterprise / firm procurement workflows, global tax, refunds. Platform takes 15%; this cost is accepted in exchange for zero backend and trusted enterprise-procurement paths.
- **Future: Stripe direct-purchase** will ship with the standalone desktop app (post-v1). A single purchase from either storefront unlocks both the add-in and the future app via a shared offline-tolerant signed JWT license token.
- **Licensing model**: one license = one person. Unlimited devices for that person (matches how Microsoft 365 itself is licensed, matches how a solo lawyer actually works).
- **No multi-seat at launch.** Firm / enterprise licensing is tracked for post-v1; the customer request data from the feedback form will shape its design.

### Network-traffic inventory (complete)

All possible network calls the Black Letter add-in can make, by design:

| Call                               | When                                                       | Initiator             | Payload                                                                                                                                    |
| ---------------------------------- | ---------------------------------------------------------- | --------------------- | ------------------------------------------------------------------------------------------------------------------------------------------ |
| Add-in manifest load               | Office startup; periodic refresh                           | Word runtime (not us) | Manifest XML (Microsoft-signed, served from Microsoft CDN)                                                                                 |
| Manifest update delivery           | When a new version is published                            | Word runtime          | New add-in bundle (Microsoft CDN)                                                                                                          |
| **Entitlement check**              | First open after purchase; on-demand via "Refresh license" | Add-in                | Microsoft Graph API call; returns license state                                                                                            |
| **Entitlement re-verification**    | Once per 30 days, passive                                  | Add-in                | Same Graph endpoint; uses cached JWT if unreachable (up to 90-day grace)                                                                   |
| **"New version available" notice** | Startup check                                              | Add-in                | Signed JSON GET to `updates.blackletter.studio/addin/latest.json` — returns `{ version, releaseNotes, url }`; response signed with Ed25519 |
| **Feedback form submit**           | User explicitly clicks Submit                              | User-initiated        | POST to `feedback.blackletter.studio/api/submit` — only the form fields the user typed                                                     |

**Zero other network traffic. Documented. Auditable. Published on `/trust`.**

What does NOT cross the network:

- Document content. Not text, not headers, not paragraphs, not titles, not filenames.
- Spellcheck requests. Hunspell runs in WebAssembly inside the add-in sandbox.
- Settings, preferences, preset overrides, personal dictionary. All in `Office.context.roamingSettings` which syncs through the user's own Microsoft 365 account, never ours.
- Telemetry. No Segment, Mixpanel, Google Analytics, Sentry-with-PII, PostHog, or any analytics library of any kind.
- Error reports. If crash reporting is ever added (post-v1), it captures stack traces and add-in state only — never document content — and is opt-in, off by default.

### Offline-tolerant license tokens

Architectural answer to the "still works in 2033" requirement:

- On successful entitlement check, the add-in receives a JSON payload from Microsoft Graph and signs a local **Ed25519 JWT** with the user's license info, expiration set to `never`, and embedded licensee-hash.
- The add-in **verifies the JWT locally on every launch** using the Ed25519 public key bundled in the add-in code. No server call required after first activation.
- **Grace-period logic**: if entitlement reverification ever fails (Microsoft API unreachable, user offline, Microsoft Graph outage), the cached JWT remains valid for **90 days** from last successful verify. After 90 days of continuous failure, the add-in shows a warning banner and degrades to preview-only mode — never hard-locks the user out.
- **If Black Letter ever goes dark**, existing licensed installs continue working indefinitely. The Ed25519 verification is self-contained. No kill switch. This is documented as a public promise in the privacy policy.

### Storage surface

| Data                                                          | Location                           | Purpose                                                   |
| ------------------------------------------------------------- | ---------------------------------- | --------------------------------------------------------- |
| Free-trial counter (0–5)                                      | `Office.context.roamingSettings`   | Syncs across the user's devices; reinstall does not reset |
| Preset choice (Standard / Conservative / Aggressive / Custom) | `Office.context.roamingSettings`   | User preference                                           |
| Preset overrides (Advanced panel)                             | `Office.context.roamingSettings`   | Power-user config                                         |
| Personal dictionary (Proofmark "Add to my dictionary")        | `Office.context.roamingSettings`   | Per-user data, synced                                     |
| License JWT                                                   | `Office.context.roamingSettings`   | Licensed on all user's devices via sync                   |
| HMAC seed for counter tamper-detection                        | `Office.context.roamingSettings`   | Generated once on first install                           |
| Per-document "Ignore once" / "Ignore in this doc" (Proofmark) | `Office.context.document.settings` | Scoped to the document                                    |
| Recent Advanced-panel state                                   | `Office.context.roamingSettings`   | UI convenience                                            |

**No local filesystem writes. No temp files. No caches on disk outside the Office add-in sandbox.**

### Privacy-policy promises (pinned to marketing, welcome screen, EULA)

1. _"Your documents never leave your computer."_ Spellcheck runs locally, cleaning runs locally.
2. _"No analytics, no telemetry, no tracking."_ We don't know how you use the tool and we prefer it that way.
3. _"Buy once. Use forever."_ No subscription, no seats, no renewal. Works offline. Works if we disappear.
4. _"No account required"_ beyond Microsoft 365, which our users already have.
5. _"Open source for the parts that matter."_ The legal-augmentation dictionary and the formatting-rules engine are public on GitHub; lawyers can inspect them, propose additions, and audit the behavior.

### EULA and privacy-policy stubs

Both documents live at `blackletter.studio/legal/` and in the add-in's "About" panel. Final wording drafted by a human lawyer (one-time legal expense, approx. $500–$1500) before v1 ships.

- **EULA**: perpetual non-exclusive non-transferable license; reverse-engineering allowed for interoperability per applicable law; warranty disclaimer; limitation of liability; governing law (to be finalized with user's preference).
- **Privacy policy**: reflects the network-traffic inventory above. Specifically: what we collect (nothing about documents); what we store (the storage-surface table); what crosses the wire (the six calls listed); how to delete everything (uninstall the add-in — everything lives in the user's M365 roaming settings, which they control).

---

## Section 4 — Security Posture

### Threat model

| #       | Threat                                                               | Surface                                                                 | Mitigation                                                                                                                                                                                                                                                                                                                                                                                                                   |
| ------- | -------------------------------------------------------------------- | ----------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **T1**  | Malicious DOCX input (XXE, zip bomb, billion-laughs, crafted XML)    | User-opened documents                                                   | Office.js is the parser — Word's hardened sandbox. Future standalone app will use a hardened DOCX library with XXE disabled, size limits, fuzzed-test corpus.                                                                                                                                                                                                                                                                |
| **T2**  | Supply-chain attack via dependency                                   | npm / Cargo dependencies, Hunspell-WASM, dictionary files               | (a) Minimize — MVP ships < 20 direct npm deps. (b) Pin exact versions in lockfile. (c) Dependabot + manual review on every bump. (d) Socket.dev or Snyk or GitHub Advanced Security in CI. (e) Dictionary PRs require two human reviewers (dictionary poisoning is real).                                                                                                                                                    |
| **T3**  | Supply-chain attack via distribution channel                         | Manifest, bundled JS, WASM binaries                                     | AppSource-only distribution for the add-in. We never offer direct sideload downloads. Manifest signed by Microsoft on publish. Bundle served from Microsoft's CDN.                                                                                                                                                                                                                                                           |
| **T4**  | MitM on our network calls                                            | The six network calls above                                             | All HTTPS with certificate pinning on `updates.blackletter.studio` and `feedback.blackletter.studio`. HSTS headers everywhere. Update-manifest response is Ed25519-signed and verified before trust.                                                                                                                                                                                                                         |
| **T5**  | XSS in the task pane                                                 | Document content rendered in task pane (detected typos, document title) | Every string from document is rendered via text nodes or escaped templating — never `innerHTML`. Dual-layer CSP: Office runtime + our own strict CSP (`default-src 'none'; connect-src 'self' https://graph.microsoft.com https://updates.blackletter.studio; script-src 'self' 'wasm-unsafe-eval'; style-src 'self'; img-src 'self' data:;`). No inline scripts. No remote CDNs. All deps bundled.                          |
| **T6**  | License-token forgery                                                | The JWT unlocking licensed UI                                           | Ed25519 signature. Public key bundled with add-in. **Private key lives in Azure Key Vault (HSM tier).** CI accesses it via GitHub OIDC → Azure federated identity — no long-lived Azure credentials in GitHub secrets. Local dev uses a separate short-lived Ed25519 key (30-day expiry) used only for dev builds. Rotation every 2 years with 6-month overlap (both old and new public keys bundled during overlap window). |
| **T7**  | DoS on the feedback Worker                                           | Public form endpoint                                                    | Cloudflare rate limits (3 submissions per IP per hour), honeypot field, hCaptcha available if abuse materializes. Worker is stateless.                                                                                                                                                                                                                                                                                       |
| **T8**  | Attacker with local machine access steals stored settings/dictionary | User's M365 roaming storage                                             | Data encrypted at rest by Microsoft (AES-256), synced through user's own account. We never see it. If attacker has machine access, the user has larger problems than Fair Copy.                                                                                                                                                                                                                                              |
| **T9**  | Counter-tampering to get more free docs                              | Local storage of the 0–5 counter                                        | Counter signed with HMAC-SHA256 keyed by per-user secret (M365 tenant ID + random seed generated on first install, stored in roamingSettings). Keeps the Ed25519 signing key out of per-document paths. Tamper → HMAC fails → fail-safe to "trial exhausted" (paid state, never free).                                                                                                                                       |
| **T10** | Reverse-engineer the license check                                   | Bundled JS + WASM                                                       | Accept that determined attackers will bypass. Minified, source-maps not shipped. Tampering detected at startup → revoke cached JWT and force Graph re-check. Most lawyers will not crack a $49 tool; those who would were not paying anyway.                                                                                                                                                                                 |
| **T11** | Malicious PR to the public legal dictionary                          | GitHub repo                                                             | CODEOWNERS requires 2 Black Letter reviewers for any `.dic` change. Signed commits. Dictionary diff review is part of CI. CI validates `.dic` / `.aff` parse cleanly via Hunspell before merge. CI runs new dictionary against a canary corpus of 200 legal documents (anonymized, private submodule) — sudden >10% change in false-positive or false-negative rate fails CI.                                                |
| **T12** | Compromise of the Black Letter GitHub org                            | All of GitHub                                                           | Required 2FA (hardware key preferred) for maintainers. Signed commits on `main`. Branch protection. Dependabot alerts. Secret-scanning. No admin tokens in workflows — OIDC where possible.                                                                                                                                                                                                                                  |
| **T13** | Supply-chain attack via release artifacts                            | Future standalone app binaries                                          | Authenticode-signed on Windows. Developer ID-signed and notarized on macOS. SHA-256 checksums published alongside. GitHub Actions run with minimum required `permissions:` per job. Release artifacts built only from tagged commits on `main`.                                                                                                                                                                              |
| **T14** | Crash or malformed output on edge-case documents                     | Formatting-transform engine                                             | Fuzz-testing harness in CI (cargo-fuzz-style for the TypeScript engine). Generates malformed XML fragments, nested table pathologies, extreme paragraph-style chains, zero-width-space abuse, control characters in hyperlinks. Any crash, hang > 5s, or output-validation failure fails CI. Runs nightly on main and on every PR touching transform code.                                                                   |
| **T15** | Delayed disclosure of security issue to users                        | Installed instances                                                     | In-app banner at next launch (fetched via signed update endpoint). Security advisory pinned on Trust page. Signed RSS feed at `blackletter.studio/security.xml` for enterprise-IT subscription.                                                                                                                                                                                                                              |

### Secrets discipline

- **Private signing key (Ed25519)**: Azure Key Vault, HSM tier. Never exported. CI accesses via GitHub OIDC → Azure federated identity.
- **AppSource publisher credentials**: stored in 1Password; accessed manually only.
- **Cloudflare API token**: scoped to minimum permissions (zone:edit on `blackletter.studio` only). Stored as GitHub Actions secret. Rotated annually.
- **GitHub fine-grained PAT for the feedback Worker**: scoped to `issues:write` on a single repo (`feedback-intake`). Expires in 90 days. Auto-rotated via a scheduled Workflow.
- **No long-lived credentials in the add-in itself.** Public keys only. User's M365 auth token is handled by Office.js and never visible to our code.

### Supply-chain hygiene

- `pnpm ci` everywhere (never `pnpm install` in CI). Exact versions pinned. Lockfile reviewed in every PR.
- Socket.dev / Snyk / GitHub Advanced Security in CI — zero tolerance for known-vulnerable packages shipping to production.
- Dictionary changes require two human reviewers plus one lawyer reviewer (CODEOWNERS).
- Quarterly dependency audit: every direct dep reviewed, unused ones removed.
- SBOM (Software Bill of Materials) published with every release for enterprise-IT buyers.

### Disclosure and response

- **SECURITY.md** in every repo with a disclosure channel: `security@blackletter.studio` (PGP-signed email preferred) and a private advisory process via GitHub Security Advisories.
- Responsible-disclosure commitment: 90-day window, credit the reporter, ship fixes before public disclosure.
- No bug bounty at launch (too early). Hall-of-fame page for good-faith reports.

### Compliance posture

- No GDPR / CCPA data-subject-rights complexity because we hold no user data. No opt-outs needed for data we never collect.
- No SOC 2 / ISO 27001 pressure at v1 since there is no service to audit. Revisit if enterprise firms request.
- `/trust` page at `blackletter.studio/trust` with the network-traffic table, public-version threat-model summary, SECURITY.md link, and signed-release verification steps.

### Marketing trust line (pinned, site + EULA + welcome screen)

> **Your documents never leave your computer. We collect nothing, store nothing, phone home for nothing but the one check that keeps your license alive. Buy once, use forever — even if we disappear tomorrow.**

---

## Section 5 — Marketing Site, Feedback Flow & Support

### Site stack

| Layer           | Choice                                                              | Why                                                                                                                                                                              |
| --------------- | ------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Framework       | Astro 4+                                                            | Static-first (zero client JS by default), component islands for interactive bits like the feedback form, MDX support for post-v1 blog, excellent Netlify integration.            |
| Styling         | Tailwind + custom theme tokens                                      | V1 Editorial palette locked in `tailwind.config.js`. Self-hosted fonts (Fraunces + IBM Plex Sans subsetted to Latin + common punctuation). No Google Fonts runtime call.         |
| Deploy          | Netlify                                                             | Preview deploys on every PR, auto-deploy on merge to `main`, SSL auto-managed, built-in forms as backup. User already pays for Netlify.                                          |
| DNS / CDN / WAF | Cloudflare (orange-cloud proxied)                                   | DDoS, WAF, bot protection, rate limits, TLS 1.3, HSTS. Nameservers at Porkbun swap to Cloudflare's.                                                                              |
| Forms bridge    | Cloudflare Workers                                                  | Stateless `feedback.blackletter.studio/api/submit` — receives form POSTs, files a GitHub Issue. Free tier covers us.                                                             |
| Analytics       | Netlify Analytics (server-side only) — subject to user confirmation | Zero client JS, no cookies, no consent banner. $9/mo on top of existing Netlify plan. Alternative: no analytics at all (purest but we learn nothing). Decision deferred to user. |

### Site map at launch

```
/                     Home. Hero (V1 Editorial). Two-tool lineup. Primary CTA: "Add to Word."
/fair-copy            Product detail. What it does. Before/after visualization. Pricing card. CTA.
/proofmark            Product detail. Highlighter system explained. Privacy emphasis (client-side, no cloud).
/trust                Network-traffic table. Threat-model summary. SECURITY.md link. Signed-release verification.
/changelog            Release notes timeline. Mirrors repo CHANGELOG.md at build time.
/changelog.xml        Atom feed for changelog subscribers.
/security.xml         Signed advisory RSS feed (for enterprise IT).
/feedback             Feature-request form: category, title, description, optional name/email, honeypot.
/support              FAQ, contact info, premium-support path for licensees.
/legal/eula           End User License Agreement (drafted by real lawyer before v1 ships).
/legal/privacy        Privacy policy (reflects the network-traffic inventory).
/legal/dpa            Data Processing Addendum ("n/a, we process no data").
```

Post-v1 slots (deferred): `/blog`, `/for-firms`, `/tools/<next-tool>`.

### Feedback flow — end to end

**Public path (no login required):**

1. User visits `/feedback`. Form fields:
   - **Category** (dropdown): Bug in Fair Copy / Bug in Proofmark / Feature request / Question about licensing / Dictionary addition / Other
   - **Title** (required, 120 char max)
   - **Description** (required, Markdown-enabled, 5000 char max)
   - **Your name** (optional — "stay anonymous" default)
   - **Your email** (optional — only if user wants a reply)
   - **Your role** (optional: Lawyer / Paralegal / Legal assistant / Firm IT / Other — triage only)
   - **Honeypot** (hidden `<input name="website">` — silently drop if filled)
2. Client-side validation; submits via `fetch` to the Worker.
3. **Cloudflare Worker** (`feedback.blackletter.studio/api/submit`):
   - Cloudflare rate limit: 3 submissions per IP per hour. 429 with polite message.
   - Honeypot check.
   - Content-length and minimum-body-length validation.
   - Input sanitization: strip HTML tags, normalize Unicode, escape Markdown-breaking chars.
   - POST to GitHub REST API with a fine-grained PAT (scoped to `issues:write` on one repo: `feedback-intake`; expires 90 days; auto-rotated).
   - Files issue with template-formatted body, category as label, redacted metadata in collapsed `<details>` block, unique `feedback-<uuid>` label.
   - If user ticked "I want a reply" with email: Worker sends confirmation via Cloudflare Email Routing.
4. **`feedback-intake` repo** is intake-only; maintainer triages and moves genuine items to product repos. Dictionary requests auto-routed to `legal-augmentation-dictionary`.

### Premium support (licensed users only)

1. In task pane overflow menu: **"Contact support"** button.
2. Opens user's default email client via `mailto:` with pre-filled subject and body:
   - To: `premium@blackletter.studio`
   - Subject: `[Premium] Fair Copy {version} — {user's subject}`
   - Body includes: License ID (anonymous hash), app version, Word version, OS. User edits or extends with their message.
3. Email goes to a small shared inbox.
4. **SLA**: first human response within 2 business days. Published on `/support`.
5. No tickets, no helpdesk SaaS at launch. Humans replying from the inbox.
6. Inbox provider decision: Fastmail (recommended — $5/mo, privacy-respecting) vs Google Workspace ($6/user/mo, familiar). Decision deferred to M4.

### Email list (opt-in)

- Single-field signup on home page: _"Get notified when the next Black Letter tool launches."_
- Clear double opt-in (confirmation email required).
- Storage: Buttondown ($9/mo) — indie-run, privacy-respecting. Alternative: static list managed via GitHub Actions (less ergonomic).
- Use: occasional (quarterly max) emails when a tool ships or a major update lands. Not promotional.
- One-click unsubscribe on every email.
- Mentioned in privacy policy.

### Release / changelog discipline

- Every release produces a CHANGELOG.md entry ([Keep a Changelog](https://keepachangelog.com/) format).
- `/changelog` page generated at build time from CHANGELOG.md — single source of truth.
- Atom feed at `/changelog.xml`.
- Release notes include: changes, migration notes (if any), known issues, signed checksums of future binary artifacts.

### SEO and discoverability

- OpenGraph cards in V1 aesthetic (server-rendered with studio wordmark + tool name + tagline).
- Structured data (Schema.org `SoftwareApplication` + `Product` + pricing).
- Sitemap.xml + robots.txt.
- Target search intents (hierarchy, not stuffing): "clean up Word document formatting," "strip formatting DOCX legal," "Word add-in for lawyers," "alternative to WordRake / Litera."
- Post-v1 blog content is craft-focused, not keyword-stuffed.

---

## Section 6 — Technical Stack, Repo Structure & CI/CD

### GitHub organization

`github.com/blackletter-studio` — free tier, owned by `twitchyvr`, member visibility set to private. Org-level defaults live in `blackletter-studio/.github`.

**Cost**: $0. **Actions quota**: org receives fresh 2000 min/mo on private repos, unlimited on public repos.

### Repo layout (polyrepo)

| Repo                            | Contents                                                                                                                                   | Visibility                                                                                                                                                   |
| ------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `fair-copy`                     | Word add-in. TypeScript + React + Vite. Contains Fair Copy and Proofmark commands in one Office.js manifest.                               | Public                                                                                                                                                       |
| `site`                          | `blackletter.studio` Astro site. Also houses `/workers/feedback/` (Cloudflare Worker) as a sibling directory with its own Wrangler config. | Public                                                                                                                                                       |
| `legal-augmentation-dictionary` | Hunspell `.dic` / `.aff` files for legal terminology. Validation scripts. Accepts PRs.                                                     | Public                                                                                                                                                       |
| `feedback-intake`               | Issues-only. CF Worker files public submissions here before triage.                                                                        | Public                                                                                                                                                       |
| `brand`                         | Logo SVGs, self-hosted font files (OFL-licensed: Fraunces, IBM Plex), Tailwind color tokens, press-kit PDFs.                               | Public (flipped from private during M0 — brand assets are inherently public-facing and submodule must be clonable without credentials from Netlify/other CI) |
| `fair-copy-app`                 | Future — standalone Mac + Windows desktop app. Tauri.                                                                                      | Public when created                                                                                                                                          |
| `infrastructure`                | Terraform / Pulumi for Cloudflare zone, DNS records, Workers routes.                                                                       | Private                                                                                                                                                      |
| `.github`                       | Org defaults: PR template, issue templates, CODE_OF_CONDUCT, reusable workflows.                                                           | Public                                                                                                                                                       |

### Per-repo tech

**`fair-copy`**:

- TypeScript, `strict: true`, no `any` (ESLint-enforced).
- React 18 + Fluent UI React v9 (Office-native component library).
- Vite with official Office Add-in plugin.
- Office.js: WordApi 1.7+ requirement set (Word 2019 / Mac Word 16.50+ / Web Word).
- Hunspell compiled to WebAssembly (custom build, public build recipe). Lazy-loaded — Fair Copy's Clean action does not pay the Hunspell download cost.
- DOCX diff rendering for Preview mode: `docx-preview` or minimal custom renderer (deferred to plan stage).
- Crypto: Web Crypto API (browser built-in) for JWT verification. No third-party crypto libraries.
- Testing: Vitest (unit), Playwright + `office-addin-test-tool` (E2E against real Word instance).
- Linting: ESLint + `@typescript-eslint` strict + `eslint-plugin-security`.
- Formatting: Prettier.

**`site`**:

- Astro 4+ with TypeScript, Tailwind, MDX.
- Self-hosted fonts subsetted to Latin + common punctuation.
- `/workers/feedback/`: TypeScript on Cloudflare Workers. Wrangler for deploy. Vitest for tests. OIDC from GitHub → Cloudflare for deploy auth.

**`legal-augmentation-dictionary`**:

- `.dic` + `.aff` in standard Hunspell format.
- Python validation script in CI: parses, runs canary corpus, reports false-positive / false-negative deltas.
- CODEOWNERS: 2 maintainers + 1 lawyer reviewer.

**`feedback-intake`**: no code. GitHub Actions for auto-labeling.

**`brand`**: SVG sources, OTF fonts (commercially licensed), `tokens.css` + `tokens.ts` consumed by `site` and `fair-copy`.

### CI / CD patterns

All workflows use GitHub OIDC for cloud authentication. No long-lived cloud tokens in repository secrets.

**PR workflow** (every PR):

- `pnpm ci`
- Lint (ESLint + Prettier check)
- Type-check (`tsc --noEmit`)
- Unit tests (`vitest run --coverage`)
- Dependency audit (`pnpm audit` + Socket.dev or Snyk)
- Secret scan (Gitleaks or GitHub Advanced Security)
- Bundle-size check against declared budgets — PR comment if over
- E2E smoke tests where applicable

**Merge-to-main**:

- PR workflow steps, plus
- Deploy preview (Netlify for site, staging AppSource for add-in) via OIDC
- Notify on deploy

**Release** (triggered on `v*` git tag):

- Production artifact build
- Signing (Ed25519 via Azure Key Vault for license JWTs; Authenticode / Developer ID for future app binaries)
- Publish (AppSource manifest update, Netlify deploy, GitHub Release with SBOM + checksums)
- CHANGELOG update via `release-please` or similar

**Branch protection on `main`**: signed commits required, 1 reviewer, passing CI, linear history, no force-pushes.

**Branch strategy** (CLAUDE.md-aligned): `feat/*` → `develop` → `main` via release PRs.

### Versioning

SemVer everywhere. `release-please` for changelog + tag automation. Git tags `v1.0.0`, `v1.1.0`, etc. AppSource manifest version tracks the same SemVer.

### Bundle budgets (CI-enforced)

| Surface                        | Budget (gzipped)                  |
| ------------------------------ | --------------------------------- |
| Fair Copy task-pane initial JS | < 300 KB                          |
| Proofmark chunk (lazy-loaded)  | < 500 KB (excluding dictionaries) |
| Hunspell WASM binary           | < 600 KB                          |
| `en_US` dictionary             | < 2 MB                            |
| Legal dictionary               | < 200 KB                          |
| Marketing site home page       | < 80 KB (critical path)           |
| Any blog post page             | < 100 KB                          |

### Dev environment

- Node 20+ LTS (matches Cloudflare Workers runtime, Netlify build image default).
- pnpm.
- VSCode / Cursor with committed `.vscode/extensions.json` recommending: ESLint, Prettier, Tailwind IntelliSense, Astro, TypeScript Next.
- `office-addin-debugging` for local add-in sideloading into Word.
- Docker not used (Office add-in dev is Word-host-specific; containerization doesn't fit). Documented as an intentional exception to CLAUDE.md's containerization preference.
- `direnv` + `.envrc.example` for local secrets (gitignored `.envrc`).

### Dependency governance

- Quarterly audit: every direct dep reviewed for maintenance status and necessity.
- Dependabot weekly for security updates; auto-merge patches after CI passes.
- Socket.dev or GitHub Advanced Security in CI.
- SBOM generation on every release (`syft` or `cyclonedx-node-npm`). Published as release artifact.
- Hard cap: add-in ships with **< 20 direct npm dependencies**. More requires explicit PR justification.

---

## Section 7 — Roadmap, Milestones & Open Questions

### Launch milestones

| Milestone                              | Scope                                                                                                                                                                                             | Rough effort               |
| -------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------- |
| **M0 — Scaffold & infra**              | Org created (done). DNS at Cloudflare (nameserver swap at Porkbun). `site` + `fair-copy` skeletons. CI workflows. Brand tokens committed.                                                         | 2–4 days                   |
| **M1 — Marketing site alpha**          | Home + `/fair-copy` + `/proofmark` + `/trust` + `/legal/*` placeholders. Feedback Worker live. Email list signup live. No download link yet. Site live at `blackletter.studio`.                   | 1–1.5 weeks                |
| **M2 — Fair Copy core**                | Formatting-strip engine. Three presets. Advanced settings panel. Confirmations for tracked changes + images. Counter card. No Proofmark yet. Unit + E2E tests.                                    | 2–3 weeks                  |
| **M3 — Proofmark**                     | Hunspell-WASM integration. `en_US` + initial `legal-augmentation.dic` (~500 seed terms). Amber-highlight + dotted-underline fallback. "Add to my dictionary" persistence. Task pane list UI.      | 1.5–2 weeks                |
| **M4 — Licensing & trial**             | AppSource SaaS offer set up. Graph entitlement check. Local JWT caching. 5-free-cleans counter integrated. Preview mode graceful degradation. EULA + privacy policy final-drafted by real lawyer. | 1–2 weeks                  |
| **M5 — Security hardening**            | Azure Key Vault for signing key. OIDC from GHA. Dictionary canary corpus in CI. Fuzz-testing harness. SBOM generation in release. Trust page populated.                                           | 1 week                     |
| **M6 — AppSource submission + review** | Manifest validation. Listing copy. Screenshots. Pricing tier locked. Submit. Microsoft review cycle (typically 3–7 business days).                                                                | 1–2 weeks                  |
| **🚀 v1.0 launch**                     | Public at `blackletter.studio`. Add-in live on AppSource. Trust + changelog + RSS + feedback form live. Newsletter sent.                                                                          | End of Week ~11–13 from M0 |

**Total realistic timeline: 10–14 weeks** solo.

### Deferred to post-v1

- Standalone desktop app (Mac + Windows, Tauri). Same license unlocks both via shared JWT.
- Stripe direct-purchase flow (paired with the standalone app).
- Firm / multi-seat licensing.
- "Suggest dictionary addition" in-add-in flow.
- Tool #2 (decision gate: 90 days post-launch, informed by feedback data).
- Blog / craft posts.
- hCaptcha on feedback form (if abuse materializes).
- Opt-in crash reporting (no document content).

### Success metrics

Calibrated to indie-studio survival, not growth-stage SaaS.

| Metric                             | 90-day target | 1-year target |
| ---------------------------------- | ------------- | ------------- |
| Unique AppSource listing visits    | 500           | 5,000         |
| Free-trial installs                | 150           | 1,500         |
| Paid conversions                   | 30 (20%)      | 300 (20%)     |
| Gross revenue (post-Microsoft 15%) | ~$1,250       | ~$12,500      |
| Feedback-form submissions (real)   | 20            | 200           |
| Dictionary PRs accepted            | 5             | 50            |
| Email list subscribers             | 200           | 1,500         |
| Support SLA adherence              | 100%          | 100%          |

### Open questions (deferred to the plan stage)

1. Exact Hunspell build recipe — existing `hunspell-wasm` package (audit needed) vs. custom Emscripten build from Hunspell C++ source. Build recipe must be public for audit.
2. Initial `legal-augmentation.dic` seed list curation — draft from Black's Law Dictionary + Bluebook + federal courts list; needs one pass from someone with real legal-editing experience before v1 ships.
3. EULA + privacy policy final wording — real lawyer review during M4. Approx. $500–$1500 one-time cost.
4. Pricing confirmation — $49 is the stake; could go $29–$79 depending on positioning. Revisit before M6 submission.
5. Email list tool — Buttondown (recommended) vs ConvertKit vs self-host.
6. OpenGraph image generation — Vercel `@vercel/og` on Netlify Functions vs build-time generator in Astro. Decide during M1.
7. Support inbox provider — Fastmail (recommended) vs Google Workspace. Decide during M4.
8. ~~Whether to publish `fair-copy` source publicly on GitHub~~ — **decided: public** (see repo table, Section 6). Moat is brand, dictionary, and customer relationships — not code. User may veto during spec review if preferred.
9. Tool #2 decision — 90-day gate after v1 launch.
10. Image-default final call — current spec uses "Warn and list" for safety; user's original answer was "Remove." User to reconfirm during spec review.
11. Analytics final call — Netlify Analytics (server-side only, $9/mo) vs zero analytics. User to confirm during spec review.
12. License governing-law jurisdiction for EULA — user preference.

### Risk register

| Risk                                                           | Likelihood | Impact       | Mitigation                                                                                                                  |
| -------------------------------------------------------------- | ---------- | ------------ | --------------------------------------------------------------------------------------------------------------------------- |
| AppSource rejects v1 on policy grounds                         | Medium     | High         | Read Office Add-in Requirements Policy early; validate manifest; have non-AppSource sideload fallback for early adopters    |
| Hunspell false-positive rate too high despite legal dictionary | Medium     | Medium       | Canary corpus testing; private beta with lawyers; "Add to my dictionary" as pressure-release valve                          |
| Lawyers don't buy (adoption stalls)                            | Medium     | High         | Direct outreach to known lawyers, not pure AppSource organic; iterate landing page copy                                     |
| Competitor ships similar tool during build                     | Low        | Medium       | Moat is brand + dictionary + trust, not code. Ship anyway.                                                                  |
| Security incident (signing-key compromise)                     | Low        | Catastrophic | Key Vault + hardware-backed signing + rotation (Section 4). Documented incident response.                                   |
| Netlify / Cloudflare / AppSource outage                        | Low        | Medium       | Static site survives CDN outage via cache. AppSource outage is Microsoft's problem; existing licenses keep working offline. |

---

## Appendix A — Glossary

- **Add-in**: a Microsoft Office extension running in Word via Office.js. Distributed via AppSource, sideload, or organization-wide deployment.
- **AppSource**: Microsoft's marketplace for Office add-ins. Handles billing, trials, entitlement.
- **Bates stamp**: sequential numbering stamped on pages of a legal document (typically an exhibit in discovery).
- **Black letter law**: foundational, well-established legal principles (distinguishes from disputed or unsettled law).
- **Blackletter (typography)**: a class of historical display typefaces with strong vertical strokes (Fraktur, Textura, etc.). The studio name plays on both meanings.
- **Bluebook**: the dominant US legal-citation style manual.
- **DOCX**: Office Open XML document format. A zip archive of XML parts. Default modern Word format.
- **Engrossed copy / engrossment**: historically, the final fair-copy version of a legal document prepared for signing.
- **Fair copy**: the clean, final version of a document after drafts. Traditional scribal term.
- **Hunspell**: open-source spellcheck engine used by LibreOffice, Firefox, macOS, and others. Ships `.dic` (word list) + `.aff` (affix rules) files.
- **Office.js**: Microsoft's JavaScript library for building Office add-ins.
- **OIDC (OpenID Connect)**: federated identity protocol. Used here to let GitHub Actions authenticate to Azure / Cloudflare without long-lived secrets.
- **Proofmark**: Black Letter's typo-flagging tool. Uses a highlighter-pen visual distinct from tracked-changes markup.
- **Redline**: in legal usage, a version of a document showing tracked insertions/deletions from a prior version. Visually indicated with strikethrough and colored text in Word.
- **Tracked changes**: Microsoft Word's revision-recording feature.

---

## Appendix B — References

- Hunspell: https://hunspell.github.io/
- Office Add-in Requirements: https://learn.microsoft.com/en-us/office/dev/add-ins/concepts/
- Keep a Changelog: https://keepachangelog.com/
- WCAG AA contrast: https://www.w3.org/WAI/WCAG21/quickref/
- Ed25519 (RFC 8032): https://datatracker.ietf.org/doc/html/rfc8032
- Bluebook (citation style): https://www.legalbluebook.com/
- Nielsen Heuristics (UX): https://www.nngroup.com/articles/ten-usability-heuristics/

---

## Appendix C — Decision log (from brainstorming)

| #   | Decision                                                                                          | Locked | Rationale                                                                                       |
| --- | ------------------------------------------------------------------------------------------------- | ------ | ----------------------------------------------------------------------------------------------- |
| Q1  | Studio-first, one tool at launch (path B)                                                         | ✅     | Studio shell future-proofs for tool #2+ without premature monorepo complexity                   |
| Q2  | Both app AND Word add-in; ship add-in first                                                       | ✅     | Add-in covers Mac + Win + Web + iPad from one codebase; no signing setup needed for MVP         |
| Q3  | V1 Editorial aesthetic (Fraunces + IBM Plex Sans; ivory/oxblood/ink)                              | ✅     | Matches lawyer audience's classical visual culture; distinctive, not trendy                     |
| Q4  | One add-in, two marketed tools (path C)                                                           | ✅     | Same install, stronger "studio with two tools" story                                            |
| Q5  | Studio name = Black Letter                                                                        | ✅     | Dual meaning (law + typography). Foolscap rejected (fool's-cap etymology too self-deprecating). |
| Q6  | Count-based trial (5 free) + Preview-mode degradation. One-time $49 lifetime.                     | ✅     | Lawyer-friendly freemium; graceful degradation; "buy once, use forever"                         |
| Q7  | Tool names: Fair Copy + Proofmark                                                                 | ✅     | Both legal/literate; intuitive; distinctive                                                     |
| Q8  | All formatting rules user-configurable with safe defaults; 3 presets + Advanced                   | ✅     | Lawyer gets one button; power users get full control                                            |
| Q9  | Cloudflare Worker → GitHub Issues for anonymous feedback; premium support via email for licensees | ✅     | GitHub-native backlog; zero login friction for lawyer; cheap infra                              |
| —   | MVP licensing = AppSource SaaS offer; Stripe added later for app                                  | ✅     | Zero backend for $49 one-time; Microsoft handles billing/trials/IT                              |
| —   | Domain: `blackletter.studio` via Porkbun; Cloudflare for DNS/WAF/Workers; Netlify for site        | ✅     | Uses user's existing paid infra; Cloudflare security at edge                                    |
| —   | GitHub org `blackletter-studio` (free tier), separate from `twitchyvr`                            | ✅     | Brand separation; fresh Actions quota; zero cost                                                |

---

## Approval gate

This spec is complete and internally consistent. Before implementation planning begins, the user must:

1. Read this document end-to-end.
2. Confirm or revise the two items explicitly flagged for reconfirmation:
   - **Image-default setting**: current spec uses "Warn and list" for safety (user's original answer was "Remove").
   - **Analytics setting**: current spec uses Netlify Analytics (server-side only) as the middle path; alternative is zero analytics.
3. Confirm any other adjustments after re-reading.
4. Approve.

Once approved, the implementation planning phase uses the `superpowers:writing-plans` skill to produce a step-by-step plan across the 7 milestones.

---

_End of design specification._
