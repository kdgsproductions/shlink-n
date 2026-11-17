# Mikkle — QR Code Shortener: Full Roadmap (Consolidated)

Base repository: `kdgsproductions/shlink-n`
Application stack: PHP/Mezzio, Doctrine ORM, MySQL.
QR rendering: `chillerlan/php-qrcode`, running locally.
Build tool: Codex.

This document is the single source of truth for the build. It supersedes all prior versions. Every change from previous versions is recorded in the Amendment Log at the bottom.

---

## Project contract — rules applying to everyone working on this build

1. **Decisions are reserved to the project owner.** All approve/backlog/reject calls belong to the owner. The owner reserves the right to override any rule in this contract.
2. **No assumptions.** For any instruction with more than one reasonable reading, present the interpretations and ask before proceeding. Proceed without asking only when the reading is unambiguous.
3. **Conflict surfacing.** When a new instruction contradicts an earlier decision, flag the conflict before proceeding.
4. **Brief.** Concise responses, no filler.
5. **Factual only, externally verified, current.** Factual claims require at least one external source reflecting the current state (licenses, maintenance status, versions change — verify today, not from memory). Label inference explicitly.
6. **Opinions labeled.** Facts and recommendations are always visibly separated, even when opinions are invited.
7. **Scope boundary.** No unsolicited feature proposals mid-task; suggestions only when invited.
8. **Lists are numbered with brief descriptions.**
9. **Decision ledger.** Every approve/backlog/reject is recorded in this document; decisions must survive context loss. Nothing lives only in chat history.
10. **Change control.** Changes to this document are made as numbered amendments in the Amendment Log; each change is atomic and traceable.
11. **Strict execution.** Tasks are executed exactly as given.

---

## Guiding constraints — apply to every phase

1. **Local QR processing** — all QR generation, validation, and rendering run locally. Do not use third-party QR APIs.
2. **No public signup** — accounts are created only by super users or company owners, within their permissions.
3. **Company isolation** — company users cannot access another company's links, users, branding, or statistics. Super-user administration follows explicitly granted permissions.
4. **One shared short domain** — one domain like TinyURL; all companies use it. Companies define ownership and permissions, not separate public URL namespaces.
5. **Domain-wide alias uniqueness** — every short code/custom alias must be unique across the shared domain, regardless of company.
6. **Permission-aware visibility** — designated users can access all company links/QRs; other users can access only their own, according to assigned permissions.
7. **Permission-aware deduplication** — duplicate searches and displayed links, QRs, and statistics must stay within the requesting user's authorized visibility.
8. **Application API included** — implement the application's own API endpoints in Phase 9, after Phase 8 passes review. This does not authorize third-party QR services.
9. **Integration preparation** — reserve nullable columns/tables needed for the application API and flyer/document-app integration before implementing their endpoints.
10. **Upstream awareness** — track Shlink upstream security patches for review and merging into the fork.
11. **Local assets only** — all JS, CSS, images, and fonts are served from the application domain. No CDNs, no external font or icon services. Dependencies (e.g., Driver.js) are installed via npm/vendor files, never loaded from a CDN.
12. **No external analytics or telemetry** — no client-side tracking scripts, beacons, or third-party analytics on any page. The only analytics are the application's own server-side visit tracking (Phase 6).
13. **Local fonts** — self-hosted webfonts (woff2) stored in the repository. Audit all templates and any inherited frontend for external font/CDN links and replace them.
14. **Translatable strings from the start** — all new UI uses the application's i18n mechanism with translatable strings; no hardcoded English in templates.
15. **Timezone convention** — store all timestamps in UTC; convert for display per user/company settings.
16. **Whitelabel portability** — the application stays organization-agnostic: companies are generic entities. A new deployment for a different organization is a configuration exercise (new database, domain, branding), not a code change. No billing, no usage quotas.

---

## Build workflow — process discipline for every phase

1. **Pre-phase cleanup** — before starting a phase, remove leftover experiments, dead code, and unused dependencies from prior work; verify the working tree is clean and tests are green on the phase's base commit.
2. **Atomic, tested phase commits** — each phase's work lands as a series of commits; every commit must leave the test suite passing. Never commit red tests.
3. **Push per phase** — completed, tested phases are pushed to the remote before the next phase begins.
4. **Per-phase documentation and user-manual update** — at the close of each phase, update both this roadmap and the user manual. This roadmap receives a dated entry recording what was implemented, deviations encountered, and any decisions made during the phase (recorded as amendments). The user manual receives verified instructions for completed functionality. At the end of the entire build, verify the complete user manual against the finished application and deliver the finalized manual with the build.
5. **Rollback paths** — every schema migration added during the phase ships with a tested rollback before the phase is considered complete.

---

## Phase 0 — Foundation and fork hygiene

1. **Install and verify base**
   - Install dependencies.
   - Run `vendor/bin/shlink-installer install`.
   - Confirm MySQL connectivity.
   - Generate an initial API key.
   - Confirm redirects work through the shared short domain.

2. **Pin environment**
   - PHP 8.4+.
   - Required application extensions.
   - `ext-mbstring`.
   - GD and/or Imagick for the chosen rendering and logo-compositing implementation.
   - MySQL 8.
   - Record exact dependency versions used for the build.

3. **CI baseline**
   - Establish passing unit, API, CLI, and database tests before modification.
   - Include `phpunit.xml.dist`, `phpunit-api.xml`, `phpunit-cli.xml`, and `phpunit-db.xml`.
   - Preserve applicable formatting, static-analysis, and OpenAPI checks.

4. **Add QR library**
   - Add `chillerlan/php-qrcode` through Composer.
   - Select and record a specific version.
   - Smoke-test local PNG and SVG rendering.
   - Install `setasign/fpdf` — required by chillerlan's **PDF output module only** — and verify local QR PDF rendering.
   - Verify EPS output using chillerlan's built-in `QREps` output module (no additional dependency).
   - Install and verify any additional dependencies required for the remaining output formats.

5. **Document divergence**
   - Define the fork's branch strategy.
   - Maintain a changelog for changes added on top of Shlink.
   - Document the process for reviewing upstream security patches.

6. **Migration tooling for fork-specific schema**
   - Define how the fork's own schema changes (companies, users, branding, onboarding, etc.) ship as migrations with tested rollback paths, separate from upstream Shlink migrations.

7. **i18n baseline**
   - Confirm the i18n mechanism (Shlink's existing translation setup) works for new UI; establish the pattern all new templates must follow.

8. **Staging environment**
   - Provision a second deployment for rehearsing migrations and releases before production.

9. **Driver.js vendoring**
   - Add Driver.js (MIT) as a local, vendored asset (npm/vendor files only — no CDN) for later use in Phase 5.5.

## Phase 1 — Company, user, and permission layer

1. **Schema**
   - Add `companies`, `users`, `company_memberships`, `roles`, and `audit_log`.
   - Support roles: `super_user`, `company_owner`, and `member`.
   - Support multiple roles per user.
   - Associate company roles and visibility permissions with the appropriate company membership.
   - Add company and creator ownership to links and other company-owned records.

2. **Super-user bootstrap**
   - Setup creates the first super user.
   - Block deletion, disabling, or demotion of the last active super user.

3. **No public signup**
   - Do not provide a public registration route.
   - Users are created through authorized administration UI/CLI workflows.

4. **Super-user administration**
   - Create and disable companies.
   - Create company owners.
   - Promote and demote super users.
   - View the audit log.

5. **Company-owner administration**
   - Create and remove member users within the owner's company.
   - Assign company visibility and management permissions.
   - Prevent company owners from granting super-user privileges.

6. **Company visibility**
   - Designated users can see all links/QRs in their company.
   - Restricted users can see only their own links/QRs.
   - Management actions require the corresponding permission; visibility alone does not grant modification rights.

7. **Shared-domain ownership and routing**
   - Use one shared short domain for every company.
   - Enforce domain-wide uniqueness of short codes and custom aliases.
   - Apply company/user permissions to management and analytics access.
   - Public redirects resolve through the shared domain and short code.

8. **Authentication**
   - Password login using `password_hash` with bcrypt or Argon2.
   - Harden sessions with HttpOnly, SameSite, and session-ID regeneration.
   - Add login throttling.

## Phase 2 — Local QR engine

1. **New QR subsystem**
   - Implement `QrRenderService` using `chillerlan/php-qrcode`.
   - Generate, validate, and render QR codes locally.

2. **Automatic QR availability**
   - Every short URL automatically has a QR pointing to its full short URL.
   - Editing the destination must not change the short code or invalidate the printed QR.

3. **Standalone QR creation**
   - Support arbitrary content without requiring URL shortening.
   - Include text, vCard, WiFi, and other supported payload types.
   - Apply company ownership and user permissions to standalone QRs.

4. **Output formats**
   - PNG.
   - SVG.
   - EPS (chillerlan built-in `QREps` module).
   - PDF (via `setasign/fpdf`, per Phase 0.4).

5. **QR endpoint contract, routes, and identifier scheme**
   - **Separate routes per QR type:** shortened-URL QRs are addressed by `GET /qr/short/{code}?format=png|svg|eps|pdf&size=N`; standalone QRs are addressed by `GET /qr/standalone/{id}?format=png|svg|eps|pdf&size=N`. The two identifier spaces never overlap.
   - Standalone QRs use a separate identifier namespace — IDs of the form `qr-{random-slug}`, stored in a `qr_codes` table with their own uniqueness pool and company/user ownership columns. Standalone QRs are addressed only through authenticated UI/API and Phase 5 share tokens, never through the public redirect path.
   - **Size is an exact-output contract:** `size` is the final square image dimension (pixels for raster, points/viewport units for vector), and the delivered file MUST be exactly that dimension — not approximately. Since module scale is an integer, `floor(size / matrix_modules)` leaves a remainder; the renderer MUST pad the remainder (center the QR on the exact-size canvas) rather than round the dimension. Malformed values are rejected with 422; out-of-range values are clamped to the 128–4096 bounds.
   - **Undersized-output rejection:** after applying the 128–4096 bounds, reject the request with HTTP 422 if the encoded matrix plus the required quiet zone cannot fit at the minimum supported module size. Never render at zero module scale, shrink the required quiet zone to force a fit, or silently enlarge the output beyond the effective requested size.
   - Quiet zone is included in the declared size (proportional, not fixed).
   - Apply company branding from Phase 3.
   - Require authorized access for ordinary downloads.
   - Add account-free downloads through Phase 5 share tokens.

6. **Rendering options**
   - Module colors.
   - Background color.
   - Scale.
   - Quiet zone.
   - Circular modules.
   - ECC level.
   - Define mappings for supported former Shlink QR query parameters.
   - Apply these through company branding defaults; per-link styling overrides remain backlogged.

## Phase 3 — Company branding

1. **Branding profile**
   - Store per-company colors, logo, default scale, ECC preference, and quiet-zone default.

2. **Logo embedding**
   - Reserve logo space in the QR matrix.
   - Composite company logos locally using GD/Imagick where applicable.
   - Implement and verify branded output for every supported export format.

3. **Logo upload hardening**
   - Enforce a MIME/extension whitelist.
   - Enforce file-size limits.
   - Re-encode accepted images server-side to strip embedded payloads.

4. **Automatic branding**
   - Apply the company profile to all company QRs.
   - Do not implement per-link styling overrides in this build.
   - Reserve the nullable field needed for a future override.

5. **Internal branding retrieval**
   - Provide internal profile access for application features.
   - Prepare for flyer/document-app integration.
   - Defer integration API endpoints until Phase 9.

## Phase 4 — Link management

1. **Custom aliases**
   - Support custom aliases.
   - Enforce uniqueness across the entire shared short domain.

2. **Destination editing**
   - Allow authorized users to change destinations.
   - Preserve the short code and full short URL.
   - Verify that previously printed QRs continue to use the updated destination.

3. **Expiration and visit caps**
   - Support validity date ranges and maximum visits.
   - Interpret validity dates in UTC (constraint 15).

4. **Tags/folders**
   - Support company-scoped tags and organization/filtering.
   - Apply user visibility restrictions to results.
   - Keep company tag management isolated.

5. **Bulk creation**
   - Support batch paste and file upload.
   - Apply ownership, permissions, alias uniqueness, and duplicate checks to each item.

6. **Titles/metadata**
   - Support titles and metadata.
   - Preserve company/user access restrictions.

7. **Duplicate-URL surfacing — Feature 51**
   - On submission, search for an exact destination-string match.
   - Do not normalize URLs in v1.
   - Search only records the requesting user is authorized to access.
   - Users with company-wide access can receive company-wide duplicate results.
   - Restricted users receive only results within their own authorized visibility.
   - Display the existing short link, QR, and statistics only when the user has permission to access them.
   - Let the user choose reuse or create-new.
   - Reuse does not grant additional ownership or modification rights.

8. **Soft delete / trash**
   - Implement recoverable link deletion.
   - Apply company/user permissions to trash and restoration.

9. **Domain not-found redirects**
   - Configure the shared domain's not-found redirect behavior.
   - Restrict changes to authorized application administration.

10. **Import**
    - Support Bitly/YOURLS migrations through the Shlink importer.
    - Assign imported links to a company and creator.
    - Enforce shared-domain alias uniqueness.
    - Apply permission-aware duplicate handling.

## Phase 5 — Shareable QR files

1. **Public share tokens**
   - Provide signed, expiring URLs for account-free QR downloads.
   - Scope each token to the intended QR download.

2. **Share management**
   - Allow authorized creators to revoke and reissue tokens.
   - Make the default expiry configurable.

3. **Public download route**
   - Rate-limit downloads.
   - Log downloads.
   - Prevent token enumeration.
   - Do not expose company management data or analytics through download tokens.

## Phase 5.5 — In-app onboarding and user training (UX concepts inspired by laudspeaker, implemented with Driver.js)

1. **First-login guided checklist**
   - A dismissible checklist of core tasks (create a link, create a standalone QR, view analytics, set branding for company owners) with completion progress per user, built on Driver.js's checklist module.

2. **Contextual step tours**
   - Tooltip walkthroughs attached to key UI flows (link creation, QR download, share tokens); triggered by role and prior actions.

3. **Role-targeted content**
   - Onboarding variants for member, company owner, and super user; super users additionally get an administration walkthrough.

4. **Event-triggered hints**
   - In-app prompts fired by application events (e.g., after a user's first link, suggest QR download). In-app notices only — no external channels.

5. **User controls**
   - Per-user on/off toggle, dismiss on any step, and a resume/reset-onboarding action; state persisted per user in MySQL (`onboarding_progress` table: user, checklist item, completed/dismissed timestamps).

6. **Admin visibility**
   - Completion status visible to company owners and super users for their authorized scope (extends Phase 1 visibility permissions).

7. **Content management**
   - Super users edit onboarding content (steps, text, targets) from the admin UI; content stored locally in the database, no external services.

8. **Library constraint**
   - All tours/checklists render with locally vendored Driver.js (Phase 0.9), MIT-licensed; no CDN loading (constraint 11).

## Phase 6 — Analytics and reporting

1. **Visit tracking**
   - Track timestamps, referrers, user agents, and derived device information.
   - Timestamps stored in UTC; reports converted to the reporting user's/company's timezone.

2. **Bot filtering**
   - Support a human-only analytics view.

3. **Company rollups**
   - Aggregate company link statistics.
   - Apply owner, super-user, and assigned visibility permissions.
   - Restrict other users' reports to their authorized records.

4. **PDF/print reports**
   - Generate reports locally on the server using **Dompdf** (renders printable HTML views to PDF, fully local).
   - Provide printable views for the implemented analytics.
   - Do not use external report-rendering services. Legacy TCPDF is explicitly excluded.

5. **CSV export**
   - Export links and statistics.
   - Restrict exported data to the requesting user's authorized scope.

6. **IP anonymization**
   - Keep IP anonymization enabled.

## Phase 7 — Notifications through SMTP

1. **SMTP configuration**
   - Provide administration settings for SMTP credentials.
   - Include a connection test.
   - Encrypt SMTP credentials at rest with libsodium (`sodium_crypto_secretbox` or equivalent) before database storage.

2. **Alert events**
   - Link nearing its visit cap.
   - Link expired.
   - Share-link downloads.
   - User created or removed.
   - Failed-login spikes.
   - Welcome/setup email sent to users created by super users or company owners (credentials-setup instructions).
   - Synthetic-monitoring failure alerts (see Phase 10.3).

3. **Credential encryption**
   - All SMTP and notification-related secrets encrypted at rest; keys documented in Phase 8.11 key-management notes.

4. **Notification permissions**
   - Restrict recipients and notification content to the appropriate authorized scope.

## Phase 8 — Security verification and release gate

Security controls are implemented alongside their related features. This phase verifies the complete build before Phase 9 API exposure.

1. **API-key scoping**
   - Verify company/user authorization for existing application API access.
   - Define authored-link and permitted-domain restrictions where applicable.
   - Do not allow permission-role editing on existing keys.
   - Provide key rotation and revocation workflows.

2. **Rate limiting**
   - Authentication.
   - API access.
   - Any public forms.
   - Share-download routes.

3. **CSRF protection**
   - Require CSRF tokens on state-changing administration forms.

4. **HTTPS/HSTS**
   - Enforce HTTPS and configure HSTS.

5. **Destination handling**
   - Allow only HTTP/HTTPS destination schemes.
   - Support a block-list for known-malicious domains.

6. **Two-factor authentication**
   - Optional per-user TOTP.
   - Optional WebAuthn authentication (security keys such as YubiKey, passkeys from password managers such as Bitwarden, and platform authenticators) — provider-neutral: the server treats any FIDO2-compliant authenticator identically. Candidate libraries: `lbuchs/WebAuthn` (MIT, standalone PHP) or `webauthn-framework` (Symfony); select and record one. Requires HTTPS and exact origin matching.

7. **Audit-log review UI**
   - Show who created, edited, or deleted users, links, branding, and settings.
   - Apply permissions to audit-log access.

8. **Database and output handling**
   - Verify parameterized database operations.
   - Verify appropriate output escaping in new UI/code paths.

9. **Isolation verification**
   - Verify company isolation across management, search, analytics, exports, imports, branding, trash, notifications, and onboarding.
   - Verify restricted-user visibility and modification permissions.
   - Verify permission-aware duplicate detection.
   - Verify share tokens expose only their authorized downloads.

10. **Encryption verification**
    - Enable InnoDB tablespace encryption at rest (MySQL Community keyring component; include redo/undo log encryption; document master-key rotation via `ALTER INSTANCE ROTATE INNODB MASTER KEY`).
    - Verify application-level libsodium encryption for: SMTP credentials, TOTP secrets, and API keys (API keys additionally stored hashed).
    - Verify these fields are intentionally NOT encrypted (documented rationale): destination URLs, short codes, aliases (required plaintext for redirect lookups and dedupe matching); analytics data; company names, tags, titles, branding profiles, onboarding state. Passwords use `password_hash()` (hashing, not encryption).

11. **Key management documentation**
    - Document where the libsodium key lives (app config, out of the repository), the threat model it addresses (database-level compromise), and backup/rotation procedure.

12. **CSP and asset-origin audit**
    - Deploy a strict Content-Security-Policy header (`default-src 'self'`).
    - Audit all templates and build output for external URLs (fonts, scripts, beacons); none may remain.

13. **UI permission-hiding tests**
    - Explicit tests that restricted users never see admin links/QR menus in the UI (not just blocked server-side).

14. **Accessibility pass**
    - Keyboard navigation and screen-reader basics on all new admin UI (hold our own UI to the standard that disqualified Intro.js).

15. **Gate**
    - Complete the security review before exposing Phase 9 integration endpoints.

## Phase 9 — Application API and integration endpoints

This phase implements the application's own API. It does not use third-party QR generation APIs.

1. **Activate reserved schema**
   - Implement per-user API credentials.
   - Implement branding-retrieval credentials.
   - Prepare the flyer-job queue table for integration.
   - Bind credentials and integration records to the appropriate company/user scope.

2. **Implement endpoints**
   - Create QRs for authorized users, including flyer use cases.
   - Retrieve authorized QR output.
   - Retrieve company branding for the flyer/document application.

3. **Authorization**
   - Enforce company isolation.
   - Enforce user visibility and management permissions.
   - Apply the same permission-aware duplicate rules as the UI.
   - Prevent callers from selecting an unauthorized company or user.

4. **Credential lifecycle**
   - Issue, rotate, and revoke scoped API credentials.
   - Do not allow permission-role editing on existing keys.

5. **API controls**
   - Rate-limit endpoints.
   - Validate requests.
   - Record relevant audit events.
   - Document request/response contracts and errors.
   - Test authorized and unauthorized access.

6. **Local processing**
   - Keep QR generation, validation, rendering, and branding local.
   - Do not introduce third-party QR services.

## Phase 10 — Operations and governance

1. **Backup strategy**
   - Automated MySQL dumps on a defined frequency with defined retention and an off-site copy.
   - Encrypt backups (TDE-protected dumps remain readable once extracted).
   - Documented and periodically rehearsed restore procedure.

2. **Data retention and deletion policy**
   - Define how long visit records are retained.
   - Define the data lifecycle when a company is disabled or deleted (links, analytics, branding, memberships).

3. **Synthetic monitoring**
   - Scheduled local check (cron) hitting a known test short link through the public redirect path; on failure, alert via Phase 7 SMTP. No external monitoring services.

4. **Abuse reporting channel**
   - A contact address or `/abuse` page for outsiders to report malicious links found on the shared domain; reports feed the Phase 8.5 block-list (manual review only).

5. **Domain reputation vigilance**
   - Because all companies share one domain, a flagged link can damage the domain for everyone. Manual monitoring stance only; automated blocklist-checking services are external APIs and are banned. Scoped to internal use (small number of trusted companies), so risk is accepted and documented.

6. **Legal/compliance documents**
   - Privacy policy and terms of use scoped to internal deployment, covering public-redirect visitor data (visit analytics with anonymized IPs).

## Backlog — explicitly outside this build

1. **Link-level QR styling overrides** — reserve a nullable field; defer per-link overrides beyond company branding.
2. **Geo-IP analytics lookup** — defer local IP-database integration; keep IP anonymization enabled.
3. **Normalized duplicate matching** — defer trailing-slash normalization, UTM stripping, and other non-exact matching.
4. **Gradient/custom-eye QR styling** — defer custom matrix-level rendering work.
5. **Laudspeaker-derived heavy features** — visual journey builder, A/B testing, liquid templating, SMS/push channels, webhooks, data-warehouse sync.
6. **Onboarding email nudges** — email reminders for incomplete checklists (deferred to avoid coupling onboarding with Phase 7 prematurely).

## Explicitly rejected — do not implement

1. Per-company usage quotas or link caps.
2. Billing of any kind.
3. Public signup.
4. Third-party QR APIs, CDNs, external analytics/telemetry, external monitoring or blocklist services.
5. Copying code from AGPL-licensed projects (Laudspeaker concepts only; no code).

## Credits and acknowledgments (retain LICENSE files; add acknowledgments section)

1. **Shlink** (shlink.io) — base of the fork, MIT.
2. **chillerlan/php-qrcode** — QR rendering incl. built-in EPS output (`QREps`), MIT/Apache-2.0.
3. **setasign/fpdf** — QR PDF output (PDF module only; not used for EPS).
4. **Dompdf** — analytics report PDFs.
5. **Driver.js** — onboarding tours/checklists, MIT.
6. **Laudspeaker** (github.com/laudspeaker/laudspeaker) — onboarding UX inspiration only; AGPL-3.0, features reimplemented, no code copied, project acknowledged as unmaintained.
7. **lbuchs/WebAuthn** or **webauthn-framework** — whichever is selected in Phase 8.6.

---

## Build order

1. Phase 0 → Phase 1: foundation, companies, users, ownership, permissions, and shared-domain rules.
2. Phase 2 → Phase 3: local QR engine, then company branding.
3. Phase 4 → Phase 5 → Phase 5.5: link management, permission-aware deduplication, sharing, then onboarding.
4. Phase 6 → Phase 7: analytics/reporting, then SMTP notifications.
5. Phase 8: security verification and release gate.
6. Phase 9: implement and expose the application's integration API after the gate passes.
7. Phase 10 items are introduced alongside their dependent phases (backups/staging in Phase 0, abuse channel with Phase 8.5, monitoring alerts with Phase 7) and verified as a whole at the gate.

The Build Workflow section applies to every phase throughout this order.

## Completion criteria for every phase

- Relevant automated tests pass.
- Applicable audit-log events are wired.
- New MySQL migrations have tested rollback paths.
- Company/user permissions are tested wherever data is accessed or changed.
- New UI uses translatable strings and locally hosted assets only.
- Pre-phase cleanup performed; work committed atomically with green tests and pushed.
- Implemented behavior and configuration are documented, with a dated per-phase entry in this document and verified user-manual instructions for the completed phase. The complete user manual is verified against the finished application and finalized at build completion.

---

## Amendment log (relative to the previous Codex-version roadmap)

1. Domain model clarified per owner: one shared short domain for all companies, TinyURL-style (Constraint 4 restated; Phase 1.7 updated).
2. Phase 2.5 expanded: standalone QR identifier scheme (`qr-{slug}` namespace, separate uniqueness pool, no public-redirect access).
3. QR endpoint size contract defined: pixels, 128–4096 bounds, 422 on malformed input, proportional quiet zone.
4. Phase 0.4 made explicit: `setasign/fpdf` for QR PDF/EPS output.
5. Phase 6.4 made explicit: Dompdf for report PDFs; legacy TCPDF excluded.
6. Phase 5.5 added: in-app onboarding (laudspeaker-inspired UX, Driver.js implementation), with related backlog items.
7. Guiding Constraints 11–14 added: local assets only, no external analytics/telemetry, local fonts, translatable strings from the start.
8. Phase 8 additions: CSP + asset-origin audit (8.12), UI permission-hiding tests (8.13), accessibility pass (8.14), WebAuthn 2FA extension with provider-neutral wording (8.6).
9. Encryption decisions added: Phase 8.10 (TDE baseline, libsodium columns, documented not-encrypted list), Phase 8.11 (key management documentation), Phase 7.1 libsodium for SMTP credentials, Phase 10.1 encrypted backups.
10. Phase 10 added: backups, retention policy, synthetic monitoring, abuse channel, domain-reputation stance, legal documents.
11. Phase 0 additions: fork-schema migration tooling (0.6), i18n baseline (0.7), staging environment (0.8), Driver.js vendoring (0.9).
12. Phase 7 additions: welcome/setup email on user creation; synthetic-monitoring alerts.
13. Guiding Constraint 15 (UTC storage/display convention) added; applied in Phases 4.3 and 6.1.
14. Guiding Constraint 16 (organization-agnostic/whitelabel portability, no billing/quotas) added; quotas explicitly rejected.
15. Project contract added at top for Codex, including decision-ledger and change-control rules.
16. Credits section consolidated (Shlink, chillerlan, setasign/fpdf, Dompdf, Driver.js, laudspeaker, WebAuthn library).
17. "Explicitly rejected" section added so nothing rejected re-enters scope.
18. QR routes corrected per owner: separate endpoints `/qr/short/{code}` and `/qr/standalone/{id}` replace the combined `/qr/{code}` route (Phase 2.5). Decision originated on the Codex side; recorded here per decision-ledger rule.
19. Exact-dimension contract corrected (Phase 2.5): the `ceil(size / matrix_modules)` approximation removed; output must be exactly the requested size, achieved by rendering at `floor(size / matrix_modules)` and padding the remainder, since module scale is integer.
20. EPS attribution corrected: chillerlan produces EPS via its built-in `QREps` output module with no additional dependency; `setasign/fpdf` is required for the PDF output module only (Phase 0.4, Phase 2.4, credits updated; supersedes amendment 4's EPS wording).
21. Build Workflow section added: pre-phase cleanup, atomic tested commits, push per phase, per-phase dated updates, tested rollback paths — cross-referenced in completion criteria.

22. User-manual lifecycle restored per owner: Build Workflow item 4 and phase completion criteria now explicitly require updates to both the roadmap and the user manual after each phase, plus verification of the complete manual against the finished application and delivery at build completion.
23. Undersized-output rejection restored per owner (Phase 2.5): after bounds are applied, return HTTP 422 when the encoded matrix and required quiet zone cannot fit at the minimum supported module size; prohibit zero-scale rendering, quiet-zone reduction to force a fit, and silently enlarged output.
24. Brand identity approved by owner (2026-11-17): product name Mikkle (M-I-K-K-L-E), tagline "Long Story Short", and the green/gold circular seal. The later uploaded reference image controls the logo artwork; see amendment 27. Retain upstream Shlink attribution and repository identity; see docs/brand-standards.md.
25. Typography approved by owner (2026-11-17): self-host Montserrat weights 400, 600, and 700 under public/assets/fonts/ with OFL.txt; declare each face once and use Montserrat, system-ui, sans-serif as the app default. Any later approved editable logo text must use Montserrat only; preserve the uploaded reference image unchanged. This resolves the brand package's font-TBD note.
26. Font conversion approved by owner (2026-11-17): download Montserrat static TTF files from the Google Fonts GitHub source repository and convert to WOFF2 locally because the inspected source directories do not supply WOFF2. No CDN/font API requests at runtime.
27. Logo reference correction by owner (2026-11-17): the uploaded image supersedes the earlier brand-package SVG and generated preview, which the owner rejected. Preserve the uploaded design, including its diagonal browser edge and larger QR pattern. Exclude the rejected assets. Obtain the original image file for an exact repository copy; do not substitute a reconstruction.
28. Installation scope confirmed by owner (2026-11-17): this build targets a fresh installation; migration of an existing Shlink installation's data is not required.
29. Hosting constraints reported by owner (2026-11-17): intended hosting is a Hepsia platform through webkitlive.com, with no SSH access. Docker availability is unconfirmed and the owner will test it. Record these as deployment constraints; do not assume hosting compatibility or provision a different production platform without an owner decision.
30. Approved logo file received from owner on GitHub (2026-11-17): public/assets/brand/mikkle-logo.png is the authoritative uploaded logo asset and is preserved unchanged. The file has been visually checked against the owner's reference; this completes the pending-file requirement in amendment 27. No replacement SVG or generated preview is approved.
31. Admin UI approach approved by owner (2026-11-17): use server-rendered Twig templates integrated with PHP/Mezzio, enhanced with locally hosted HTMX for interactive updates. Apply the existing requirements for company/user authorization, session security, CSRF protection, translatable strings, accessibility, Montserrat typography, and local assets. This records the implementation approach; it does not mark UI implementation or a build phase complete.
