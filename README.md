# autolycos

Anti-bot fetching subsystem: escalating `Fetcher` tiers (`http` to `tls` to
`browser` to `uc`) behind a stable port, with fail-closed SSRF guards.
Domain-agnostic by design: the caller injects a `DomainPolicy`, so the library
carries no hardcoded allowlist.

> **Pre-release (`0.1.0a1`).** The public API is not frozen yet and may change
> before the first stable release. Published mainly to reserve the name and
> validate the release pipeline. Not recommended for production use yet.

## What it does

`autolycos` resolves access to bot-protected pages by escalating through
increasingly capable (and costly) fetcher tiers, stopping at the first success:

1. `http` -- plain HTTP with a pinned IP (SSRF-safe).
2. `tls` -- TLS fingerprint impersonation (`curl_cffi`).
3. `browser` -- undetected headless browser (`patchright` + `playwright-stealth`).
4. `uc` -- undetected Chrome driver (`seleniumbase`) for the hardest protectors.

Access selection is a separate concern (the `Router`) from the tools
(the adapters). SSRF safety is enforced fail-closed at both the config-mutation
gate and the fetch gate via a single shared predicate.

## Stable contract (consumer-facing)

- `autolycos.ports`: `Fetcher`, `FetchResult`, `Router`
- `autolycos.safety`: `DomainPolicy`, `check_scheme_and_domain`
- `autolycos.errors`: `SSRFError`, `FetchError`
- `autolycos.router`: `StaticRouter`

## Install

```bash
pip install autolycos                 # core (http tier)
pip install "autolycos[tls]"          # + TLS impersonation
pip install "autolycos[browser]"      # + undetected browser
pip install "autolycos[uc]"           # + undetected Chrome driver
```

## License

Apache-2.0. See [LICENSE](./LICENSE).
