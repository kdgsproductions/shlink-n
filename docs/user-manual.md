# Shlink-n user manual

> Draft outline — the new build has not been verified as implemented. This file is not yet an operating guide for its planned features.

See the [build scope and delivery rules](build-scope.md) for requirements. Sections will receive verified instructions at each completed phase; the complete manual will be checked against the finished application before final delivery.

## Installation and first setup

Pending Phase 0 verification: environment requirements, dependencies, MySQL setup, shared domain, HTTPS, installation commands, initial API key, and redirect checks.

## Accounts and access

Pending Phase 1: first super-user setup; login/logout; company creation/disabling; owner/member administration; multiple roles; company-wide versus own-record visibility; modification permissions; last-active-super-user protection. No public signup is planned.

## Creating and managing short links

Pending Phase 4: custom aliases unique across the shared domain; destination changes; dates/visit caps; titles; company-scoped organization; bulk creation; permission-aware duplicate results with reuse/create-new; trash/restoration; imports; shared-domain not-found settings.

## Creating and downloading QRs

Pending Phases 2–3: automatic link QRs; standalone text/vCard/WiFi payloads; company branding; PNG/SVG/EPS/PDF downloads; supported sizing and rendering defaults. Per-link styling overrides are deferred.

## Managing company branding

Pending Phase 3: authorized profile editing, colors, logos, upload limits, default scale/ECC/quiet zone, and output previews.

## Sharing QR files

Pending Phase 5: issuing expiring download links, revoking/reissuing tokens, expiry settings, and downloading without an account.

## Analytics and reports

Pending Phase 6: permitted link/company views, bot filtering, CSV exports, printable/PDF reports, and anonymized IP behavior. Geo-IP lookup is deferred.

## Notifications

Pending Phase 7: SMTP configuration/test, credential management, alert events, and authorized recipients.

## Security and audit history

Pending Phase 8: optional TOTP, scoped credentials, rotation/revocation, audit access, and explanations of verified access/error behavior.

## Application API and flyer/document integration

Pending Phase 9: approved authentication, actual endpoints/payloads, QR creation/retrieval, branding retrieval, permission scope, rate limits, error responses, and credential rotation/revocation.

## Troubleshooting

Pending implementation: verified symptoms and recovery steps for setup, authentication, permissions, redirects, QR output, imports, sharing, reports, SMTP, and API requests.

## Final manual acceptance

- Verify every published instruction against the finished build.
- Use actual UI labels, commands, configuration names, and API contracts.
- Cover super-user, company-owner, member, public-download, and integration workflows as applicable.
- Identify optional/deferred features and remove stale implementation placeholders.
- Record the build/version verified and any remaining documented limitations.
