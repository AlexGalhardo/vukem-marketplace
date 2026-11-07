<div align="center">

<h1><strong><a href="https://marketplace.vukem.dev" style="text-decoration: none" target="_blank">Vukem Marketplace</a></strong></h1>

[![CI](https://github.com/AlexGalhardo/vukem-marketplace/actions/workflows/ci.yml/badge.svg)](https://github.com/AlexGalhardo/vukem-marketplace/actions/workflows/ci.yml)
[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Go 1.27.3](https://img.shields.io/badge/go-1.27.3-00ADD8?logo=go&logoColor=white)](go.mod)
<a href="https://github.com/AlexGalhardo/vukem-marketplace/releases"><img src="https://img.shields.io/github/v/release/AlexGalhardo/vukem-marketplace?include_prereleases&label=version" alt="Version"></a>
<a href="https://scorecard.dev/viewer/?uri=github.com/AlexGalhardo/vukem-marketplace"><img src="https://api.scorecard.dev/projects/github.com/AlexGalhardo/vukem-marketplace/badge" alt="OpenSSF Scorecard"></a>

<h3><strong>Open Source Multi-Seller Marketplace with Stripe Connect and PIX</strong></h3>

</div>

## Introduction

Vukem Marketplace is an open source multi-seller marketplace. Sellers open shops and list products with images; buyers search, fill a cart, calculate freight by CEP and pay by card or PIX, and sellers receive their share through Stripe Connect. Its layout is inspired by the big Latin American e-commerce sites (not affiliated, no brand assets). It runs in production at <https://marketplace.vukem.dev> and is the flagship battle test for Vukem 1.0.

This application is built using the [Vukem Go framework](https://vukem.dev), with Inertia v3, React 19, Base UI and Tailwind CSS v4.

- **A real product, not a demo page.** Money flows, moderation, APIs and operations are complete, so every Vukem feature is tested under real constraints.
- **One domain, three API styles.** REST, GraphQL and gRPC share the same models, scopes and rate limits.
- **Secure by default.** Every item of the [OWASP Top 10:2025](PLAN.md#5-owasp-top-102025-checklist) names the test that proves it.
- **Windows is first-class.** Pure Go, no C compiler; developed on Windows 11, deployed on Linux.

## Summary

- [Introduction](#introduction)
- [Features](#features)
- [See it working](#see-it-working)
- [Quick start](#quick-start)
- [Documentation](#documentation)
- [Stability](#stability)
- [Contributing](#contributing)
- [Code of conduct](#code-of-conduct)
- [Security vulnerabilities](#security-vulnerabilities)
- [License](#license)

## Features

| Area | What you get |
| --- | --- |
| **Storefront** | Home sections, Postgres full-text and trigram search with filters, sorting and live suggestions in the navbar, category, product and shop pages, recently viewed products, EN/PT-BR and dark/light themes. |
| **Sellers** | One shop per seller, products (draft/published/archived) with S3 images and 320/640/1280 thumbnails made by a queue job, orders, balance from a ledger and payout requests. |
| **Cart and checkout** | Server-priced multi-seller cart, address book with ViaCEP autofill, freight table by weight band and region, card through Stripe Connect (Accounts v2, 10% platform fee) and PIX through AbacatePay. |
| **Community** | Reviews of paid order lines with votes, replies and moderation; product questions answered by the shop; live buyer-seller chat over WebSockets with SSE/polling fallback. |
| **Auth** | Email/password, Google sign-in, email verification, password reset, TOTP two-factor with recovery codes, account and security pages; admin area behind the admin role and confirmed 2FA. |
| **Public APIs** | REST `/api/v1` with OpenAPI 3.1 and Scalar docs, GraphQL with depth/complexity limits, gRPC/Connect, scoped API keys and signed, retried, SSRF-safe seller webhooks. |
| **Operations** | Queue worker and scheduler inside `serve`, `/health` and `/ready` probes, OpenTelemetry traces, Telegram alerts for backend errors and browser errors (telegram-logger), feature flags in `/admin/flags`. |
| **Demo data** | A deterministic English catalog with embedded WebP images: 100 shops, 2000 products, 300 buyers, ~1500 orders, ~1200 reviews and ~1200 questions. |

| Layer | Choice |
| --- | --- |
| Backend | Go 1.27, Vukem (`contrib/graphql`, `contrib/grpc`, `contrib/stripe`, `contrib/imaging`, Postgres, Redis, MongoDB, S3, WebSocket and OpenTelemetry drivers) |
| Frontend | Inertia 3.9, React 19.3, TypeScript 7, Base UI 1.9, Tailwind 4.3, Vite 8 |
| Data | PostgreSQL 18 (core), Redis 8 (sessions, cache, queues, rate limits, chat fan-out), MongoDB 8 (audit, history, telemetry), S3-compatible storage (RustFS locally) |
| Payments | Stripe Connect (card, test mode), AbacatePay PIX (sandbox) |
| Email | Mailtrap (Email Sandbox in development and CI, HTTP API in production), Mailpit offline |
| Tooling | Bun 1.4, Biome, Playwright, golangci-lint, lefthook, commitlint |
| Hosting | Railway (Docker image), images on GHCR |

## See it working

<details>
<summary><strong>Show the GIFs (6)</strong></summary>

### Create an account

Name, email and password limits with a live password checklist and a show/hide toggle, all validated by Vukem `z` on the server.

![Sign-up with live password rules](docs/media/register.gif)

### Search, suggestions and cart

Suggestions from 3 characters: the 8 best products with image, title, category and price. Keyboard friendly.

![Navbar search suggestions, then add to cart](docs/media/search.gif)

### Dark and light themes

![Theme toggle](docs/media/theme.gif)

### Checkout

Cart, delivery address and the review-and-pay step with server-side freight and totals. PIX (AbacatePay) and card (Stripe Connect) are enabled by adding the provider sandbox keys; the local stack in this recording has none, so both show as unavailable.

![Cart to the payment step](docs/media/checkout.gif)

### Live chat with the seller

WebSocket chat between a buyer and a seller in two separate sessions.

![Buyer and seller chat in real time](docs/media/chat.gif)

### Tests

![Go tests passing](docs/media/tests.gif)

All GIFs are recorded against the local stack with `bun scripts/record-readme-gifs.ts` (see [tooling](docs/tooling.md)).

</details>

## Quick start

Requirements: Docker with Compose, Bun 1.4.3 and the Vukem CLI (`go install github.com/AlexGalhardo/vukem/cmd/vukem@v0.39.0`).

1. Clone the repository and enter it.

   ```sh
   git clone https://github.com/AlexGalhardo/vukem-marketplace.git
   cd vukem-marketplace
   ```

2. Create your environment file.

   ```sh
   cp .env.example .env
   ```

3. Generate the app key (it is written to `.env`).

   ```sh
   vukem key:generate
   ```

4. Install the JS dependencies and git hooks.

   ```sh
   bun install
   ```

5. Start the whole stack (app, worker, scheduler, databases, storage, mail catcher).

   ```sh
   docker compose up -d --build
   ```

6. Check that it is ready, then open <http://localhost:3200>.

   ```sh
   curl http://localhost:3200/ready
   ```

7. Log in with a demo account: `buyer@example.com`, `seller@example.com` or `admin@example.com` (password `marketplace-demo-password`, development only).

Prefer hot reload on the host? See [setup without Docker](docs/setup-local-without-docker.md).

| Service | URL |
| --- | --- |
| App | <http://localhost:3200> |
| Mailpit | <http://localhost:8035> |
| RustFS console | <http://localhost:9201> |
| Postgres / Redis / Mongo | `localhost:5442` / `6389` / `27027` |

## Documentation

- [Architecture and system design](docs/architecture-system-design.md).
- [Local setup with Docker](docs/setup-local-with-docker.md) and [without Docker](docs/setup-local-without-docker.md).
- [Deploy on Railway](docs/setup-deploy-railway.md).
- [Tooling](docs/tooling.md) and [demo data and images](docs/demo-data-and-images.md).
- [Telegram logging](docs/telegram-logging.md) (backend with Vukem, browser with telegram-logger).
- [PLAN.md](PLAN.md): decisions, data model, money flows, routes, APIs, OWASP checklist, roadmap.
- [vukem-feedback.md](vukem-feedback.md): bugs, gaps and ideas found while building on Vukem.
- [AGENTS.md](AGENTS.md) and [CLAUDE.md](CLAUDE.md): rules for coding agents. The project is developed with AI coding agents; the path-scoped rules in `.claude/rules/` and the skills in `.claude/skills/` describe the conventions, the test-first workflow and the security contract every change follows.

## Stability

Vukem Marketplace follows [Semantic Versioning](https://semver.org) and tags releases as `vX.Y.Z` on `main`. Releases are published on [GitHub Releases](https://github.com/AlexGalhardo/vukem-marketplace/releases).

Changes are listed in [CHANGELOG.md](CHANGELOG.md), which follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/) and links every entry to its commit.

## Contributing

Thank you for considering a contribution to Vukem Marketplace.

- Write the test first. Run the specific test, lint and typecheck that cover your change.
- Accounts, payments, uploads, webhooks and API keys need a security test.
- Update `CHANGELOG.md` and the documentation in the same change when behaviour, commands or conventions change.

```sh
golangci-lint run ./... && go test -race ./...
bun run lint && bun run typecheck && bun run build
bun run test:e2e                # needs the compose stack and `bunx playwright install chromium`
```

Integration tests and the email-testing setup (Mailtrap) are described in [PLAN.md](PLAN.md#91-commands). The full guide is in [CONTRIBUTING.md](CONTRIBUTING.md).

### Pull requests

1. Create a branch from `main`.
2. Make one logical change per commit.
3. Open a pull request against `main`.

`main` accepts changes through a pull request with green checks and linear history.

### Commits

Use [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/) in English: `type(scope): description`.

- Types: `feat`, `fix`, `perf`, `refactor`, `docs`, `test`, `build`, `ci`, `chore`, `style`, `revert`.
- Scopes: `auth`, `catalog`, `cart`, `checkout`, `payments`, `orders`, `shops`, `admin`, `api`, `ui`, `infra`, `deps`, `repo`, `docs`.

`bun install` installs the lefthook git hooks: commitlint checks the header locally, and CI checks the commits and the pull request title.

## Code of conduct

To keep the community welcoming to everyone, please review and follow the [Code of Conduct](CODE_OF_CONDUCT.md).

## Security vulnerabilities

If you discover a security vulnerability, please follow the [security policy](SECURITY.md) instead of opening a public issue.

## License

Vukem Marketplace is open source software released under the [MIT license](LICENSE).
