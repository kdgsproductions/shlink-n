# Shlink-n build scope and delivery rules

## Status and authority

This document records the roadmap and clarifications agreed before implementation. Features below are planned requirements, not a claim of completed functionality. The owner's forthcoming updated roadmap must be reconciled here before implementation begins; do not fill unresolved gaps by guessing.

The roadmap is the source of truth. Existing Shlink behavior is the starting point and may be changed to meet it.

## Constraints and clarified behavior

- Generate, validate, and render QRs locally with chillerlan/php-qrcode; do not use third-party QR APIs.
- Use one shared short domain. Companies define ownership and access, not public namespaces.
- Enforce short-code/custom-alias uniqueness across the shared domain.
- No public signup. Super users and company owners create accounts within their permissions.
- Isolate company data. Permit company-wide or own-record visibility according to assigned permissions.
- Distinguish visibility from permission to modify records.
- Duplicate searches and displayed links, QRs, and statistics must respect the caller's authorized visibility. Company-wide duplicate handling is available only to users with the appropriate company permissions.
- Implement the application's own integration API in Phase 9 after the security gate passes. Reserve required schema earlier.
- Keep IP anonymization enabled.
- Track upstream security patches and document fork divergence.

## Planned phases

### Phase 0 — Foundation

Install dependencies and run the installer; verify MySQL connectivity, initial API-key creation, and redirects. Target PHP 8.4+, MySQL 8, required extensions, mbstring, and GD and/or Imagick as needed. Record dependency versions. Establish passing unit, database, API, CLI, formatting, static-analysis, and OpenAPI checks. Add the QR library and smoke-test PNG/SVG; verify additional EPS/PDF dependencies. Document branch strategy, changelog, and upstream patch handling.

### Phase 1 — Companies, users, permissions, and authentication

Add companies, users, company memberships, multi-role assignments, and audit logs. Support super_user, company_owner, and member roles. Bootstrap the first super user and protect the last active super user from deletion, disabling, or demotion.

Provide super-user company/owner administration, company-owner member administration, visibility assignments, and audit access. Associate links and other company records with company/creator ownership. Enforce the shared-domain rules. Implement password hashing with bcrypt/Argon2, hardened sessions, regeneration, and login throttling. Do not add public registration.

### Phase 2 — Local QR engine

Implement QrRenderService and automatic QR availability for each short URL. Encode the full short URL so destination edits preserve printed QR usefulness. Support standalone content such as text, vCard, and WiFi with company/user ownership.

Provide PNG, SVG, EPS, and PDF output. Define GET /qr/{code}?format=png|svg|eps|pdf&size=N, standalone identifiers, size units/bounds, and access rules. Support company-default colors, background, scale, quiet zone, circular modules, and ECC; document supported mappings from former QR parameters. Branding comes from Phase 3. Per-link overrides remain deferred.

### Phase 3 — Company branding

Store company colors, logo, default scale, ECC, and quiet zone. Reserve logo space and composite locally; verify branding in all output formats. Enforce upload type/size restrictions and server-side image re-encoding.

Apply branding automatically to company QRs. Provide internal branding retrieval for later flyer/document integration; integration endpoints wait until Phase 9. Reserve a nullable future per-link styling field.

### Phase 4 — Link management

Support domain-wide unique custom aliases, destination editing without changing short codes, validity ranges, visit caps, company-scoped tags/organization, bulk paste/file creation, titles/metadata, and recoverable trash/restoration.

Feature 51: exact destination-string duplicate search only, restricted to the user's authorized records. Surface permitted existing links, QRs, and stats; offer reuse or create-new. Reuse must not grant additional permissions. Defer URL normalization.

Provide shared-domain not-found redirects under authorized administration and Bitly/YOURLS imports with company/creator ownership, alias collision checks, and permission-aware duplicate handling.

### Phase 5 — QR sharing

Provide signed, expiring account-free QR download tokens. Allow authorized creators to revoke/reissue them and configure default expiry. Rate-limit/log downloads and prevent enumeration. Share tokens grant the intended download, not company management or analytics access.

### Phase 6 — Analytics and reporting

Provide visit timestamps, referrers, user agents/device information, bot filtering/human-only views, and permission-scoped company rollups. Generate local printable/PDF reports and CSV exports. Select the local PDF library during implementation. Keep IP anonymization enabled; defer Geo-IP lookup.

### Phase 7 — SMTP notifications

Provide administration SMTP configuration and connection testing. Encrypt credentials at rest. Support alerts for approaching visit caps, expiry, share downloads, user creation/removal, and failed-login spikes. Scope recipients/content to appropriate permissions.

### Phase 8 — Security gate

Implement feature-specific security during development, then verify it here: scoped API access, rotation/revocation without editing existing key roles, rate limits, CSRF protection, HTTPS/HSTS, HTTP/HTTPS destination allow-list, malicious-domain block-list, optional TOTP, and audit-review UI.

Verify parameterized database operations and context-appropriate output escaping separately. Test company isolation, restricted-user permissions, deduplication, imports, exports, trash, branding, notifications, and share tokens. Pass the review before Phase 9 integration endpoint exposure.

### Phase 9 — Application integration API

Implement scoped per-user API credentials and branding-retrieval credentials, with the reserved flyer-job queue schema. Implement endpoints for creating QRs for authorized users/flyer use, retrieving authorized QR output, and retrieving company branding.

Enforce company/user permissions and permission-aware duplicate behavior. Include issuance, rotation, revocation, rate limits, validation, audit events, request/response/error documentation, and access tests. Keep QR processing local.

Exact endpoint paths and payloads must be documented from the approved implementation contract; do not invent them in user instructions.

## Deferred features

- Per-link QR styling overrides beyond company branding; reserve a nullable field.
- Geo-IP lookup using a local IP database.
- Normalized duplicate matching such as trailing-slash handling or UTM stripping.
- Gradients/custom-eye styling requiring custom rendering.

Phase 9 application API endpoints are included, not backlogged.

## Cleanup and phase delivery rules

1. Before each phase, review for dead code and record replacement/removal candidates.
2. Distinguish verified unused code from live code scheduled for replacement.
3. Check references, configuration, routes, dependency-injection wiring, runtime discovery, and applicable compatibility/migration requirements before removal.
4. Remove only verified unused code that preserves required behavior. Retain live implementations until tested replacements are ready.
5. Run relevant regression checks after cleanup and the required phase checks after implementation.
6. Add applicable audit events and test new MySQL migration rollback paths.
7. Update README, feature status, changed configuration/install instructions, API documentation where applicable, and the user manual at each phase end.
8. Record checks run, results, cleanup, and known limitations. Do not claim unrun checks passed or that tests prove the absence of every defect.
9. The owner authorizes committing and pushing each completed phase once required checks pass and no known regressions remain. If required verification is blocked, report it before pushing the phase.
10. Keep phase changes and cleanup reviewable in the commit history.

## Documentation lifecycle

The [user manual](user-manual.md) begins as a planned outline. Add verified, usable instructions as each phase is completed. Mark unimplemented sections pending; do not invent UI labels, commands, or endpoint examples.

At build completion, walk through the documented workflows against the finished application, finalize role-specific instructions, installation/configuration, reports, integrations, and troubleshooting, and deliver the completed manual with the build.

Keep upstream documentation identifiable as upstream guidance. Do not present upstream badges, packages, or instructions as evidence that this fork's new features have been built or tested.
