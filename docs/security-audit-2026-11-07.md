# Security audit, 2026-11-07

Target: `vukem-marketplace-reactjs` at branch `build/vukem-v0.39.0`.
Method: the vendored Cloudflare `security-audit` skill (`.claude/skills/security-audit`), full audit mode, profile `quick`.

Machine-readable records: [findings](security-audit-2026-11-07.findings.json), [coverage ledger](security-audit-2026-11-07.coverage.json).

## Scope and coverage statement

Partial coverage by design. Profile `quick`: one reviewer, coarse ledger units (8), one hunting pass, no follow-up waves. Source review only; no live endpoints were probed and no dependency was added. Scanners: `go tool govulncheck ./...` (0 reachable vulnerabilities) and `bun audit --prod` (none).

In scope: auth and two-factor routes, checkout, payments and payouts ownership checks, seller webhooks (SSRF), media serving and uploads, REST and GraphQL limits, the client-log endpoint, compose stack, Dockerfile and the GitHub workflows. Not reviewed in depth: chat and Q&A authorization, moderation, the Stripe and AbacatePay client code, the React pages beyond two-factor, and the vukem framework itself.

## Summary

| Severity | Confirmed | Needs validation | Rejected |
| --- | --- | --- | --- |
| Critical | 0 | - | - |
| High | 0 | - | - |
| Medium | 0 | - | - |
| Low | 3 | - | - |
| Informational | 1 | - | - |
| Unrated | - | 0 | 2 |

## Findings

### SA-001 Two-factor recovery codes regenerate with a session alone (low)

- Fingerprint: `vukem-marketplace-reactjs:auth:recovery-codes-no-password`
- CWE: CWE-306 (missing authentication for a critical function)
- Location: `routes/two_factor.go:40`, `app/controllers/account/two_factor.go:117`
- Impact: a hijacked session could mint working recovery codes (a second-factor bypass) without the password; disabling two-factor already required it.
- Fix: the handler verifies the current password, the route has the confirm throttle, and the page asks for the password. Regression: `TestTwoFactorRecoveryCodes`. Fixed in `c849439`.

### SA-002 Dev compose ports published on all interfaces with default credentials (low)

- Fingerprint: `vukem-marketplace-reactjs:infra:compose-ports-all-interfaces`
- CWE: CWE-1327 (binding to an unrestricted IP address)
- Location: `compose.yaml:48` and the other published ports
- Impact: Postgres, Redis, Mongo, RustFS and Mailpit are reachable from the local network while the dev stack runs; the credentials are the published defaults. The same mapping comes from the vukem scaffold (see `vukem-feedback.md` VE-8 in the workspace).
- Fix: every published port is prefixed with `127.0.0.1:`. Regression: `TestComposePublishesPortsOnLoopbackOnly`. Fixed in `d983a7b`.

### SA-003 Workflow checkouts persist the GitHub token (informational)

- Fingerprint: `vukem-marketplace-reactjs:ci:checkout-persists-credentials`
- CWE: CWE-522 (insufficiently protected credentials)
- Location: `.github/workflows/ci.yml:23`, `dast.yml:21`, `release.yml:16`
- Fix: `persist-credentials: false` on every checkout. Verified by parsing the workflows; no code test applies. Fixed in `b8a314a`.

### SA-004 Third-party Actions pinned by tag (low)

- Fingerprint: `vukem-marketplace-reactjs:ci:actions-pinned-by-tag`
- CWE: CWE-829 (inclusion of functionality from an untrusted control sphere)
- Location: `.github/workflows/ci.yml`, `dast.yml`, `release.yml`
- Impact: a moved upstream tag would run attacker code with the job token; `release.yml` holds `packages: write`.
- Fix: every `uses:` is pinned to a commit SHA with the tag in a comment, as in `security-audit.yml`. No code test applies. Fixed in `bf71ece`.

## Rejected after validation

- `vukem-marketplace-reactjs:webhooks:seller-endpoint-ssrf`: deliveries use `httpclient.Safe`, which refuses loopback, private and metadata addresses at connect time.
- `vukem-marketplace-reactjs:catalog:media-path-traversal`: `catalog.MediaObject` validates the path before the disk opens; responses carry `nosniff` and a sandbox CSP.

Hardening notes, not findings: `POST /reset-password` has no route throttle (the token is the secret); the per-IP limits rely on the platform setting `TRUSTED_PROXIES` correctly in production, which this audit could not check.

## Verification

Findings and ledger passed the skill validators (`validate-findings.cjs`: 6 findings; `validate-coverage-ledger.cjs`: 8 units), run on Windows Node with a preload shim for the missing POSIX no-follow constants. The skill's own CLI tests fail 7 cases on Windows for the same reason, as in `vukem-email`.
