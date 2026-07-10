# autolycos

**Fetch web pages that fight back, without your code caring how.**

`autolycos` is an anti-bot fetching subsystem: it resolves access to
bot-protected pages by escalating through increasingly capable (and costly)
fetcher tiers, behind a single stable port, with fail-closed SSRF guards. It is
domain-agnostic by design -- the caller injects a `DomainPolicy`, so the library
carries no hardcoded allowlist and can be reused across projects.

> **Pre-release (`0.1.0a1`).** The public API is not frozen yet and may change
> before the first stable release. Published to reserve the name and validate
> the release pipeline. Not recommended for production use yet.

## The problem it solves

Fetching a page from a modern e-commerce or content site is rarely a plain HTTP
GET anymore. Sites sit behind bot managers (Cloudflare, Akamai, DataDome,
Kasada) that inspect TLS fingerprints, browser automation signals, and
behavioural signatures. The same URL might return a clean `200` one minute and a
challenge page or `429` the next.

Handling this well means owning a messy escalation ladder: try a cheap HTTP call,
fall back to TLS impersonation, then to a real (undetected) browser, and only as
a last resort to a full undetected Chrome driver. Each rung is more capable but
slower, heavier, and more expensive. Doing this inline, in every project that
needs a page, spreads that complexity everywhere and makes it easy to leak
requests to internal addresses (SSRF) or to confuse a transient block with a real
result.

`autolycos` packages that ladder once, correctly, behind a clean boundary.

## What it does

It escalates through fetcher tiers by increasing cost, and stops at the first
success:

1. `http` -- plain HTTP with a pinned IP (SSRF-safe). Cheapest, works on open sites.
2. `tls` -- TLS fingerprint impersonation (`curl_cffi`). Beats TLS-signature filters.
3. `browser` -- undetected headless browser (`patchright` + `playwright-stealth`).
4. `uc` -- undetected Chrome driver (`seleniumbase`), for the hardest protectors.

Your application asks a `Fetcher` for a URL and gets back a `FetchResult`. It
never imports `patchright`, `curl_cffi`, or `seleniumbase`, and never learns
which rung actually delivered the page.

## Why use it

- **A stable port, not a pile of tools.** Your code depends on `Fetcher` /
  `FetchResult`. Swapping, adding, or removing a tool is an adapter change, not a
  rewrite of your call sites.
- **Cost-aware escalation.** You only pay for the heavy tiers when the cheap ones
  fail. Open sites stay fast; hard sites still get through.
- **Fail-closed SSRF safety by construction.** A single shared predicate
  (`check_scheme_and_domain`) guards both the config-mutation path and the fetch
  path, so the two gates can never drift apart. Rejections raise `SSRFError`,
  never a silent pass. Scheme allowlist closes `javascript:` / `file:` / internal
  targets.
- **Domain-agnostic and reusable.** No hardcoded site list. The caller injects a
  `DomainPolicy`, so the same library serves a price monitor, a content archiver,
  or any tool that needs resilient fetching.
- **Policy separate from tools.** The `Router` (which tier to use) is a distinct
  layer from the adapters (the tools themselves), so selection strategy evolves
  independently of the fetchers.

## Typical use cases

- Daily price and availability monitoring of products on protected retailers.
- Scraping or archiving pages behind Cloudflare / Akamai / DataDome.
- Any backend that needs "get me this page, reliably, and tell me if you could
  not" without embedding browser-automation plumbing.

## Stable contract (consumer-facing)

- `autolycos.ports`: `Fetcher`, `FetchResult`, `Router`
- `autolycos.safety`: `DomainPolicy`, `check_scheme_and_domain`
- `autolycos.errors`: `SSRFError`, `FetchError`
- `autolycos.router`: `StaticRouter`

Everything else (`adapters/*`, `challenge`, `egress_proxy`) is internal and
reached only through a `Router` / `DomainPolicy` you construct.

## Install

```bash
pip install autolycos                 # core (http tier)
pip install "autolycos[tls]"          # + TLS impersonation
pip install "autolycos[browser]"      # + undetected browser
pip install "autolycos[uc]"           # + undetected Chrome driver
```

Extras are additive: install only the tiers you actually need. The heavier
browser and uc tiers pull in Chromium-class dependencies.

## Name

Autolycos, son of Hermes, was the master of disguise and sleight of hand. The
library wears the same trick: it changes its fingerprint to pass unnoticed.

## License

Apache-2.0. See [LICENSE](./LICENSE).
