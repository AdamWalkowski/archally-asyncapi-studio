# Archally fork additions

Everything under `archally/` is owned by the `archally-asyncapi-studio` fork and is
**never** modified by upstream syncs. This directory holds:

- `theme/` — CSS overrides matching `@archally/viewer-shell` design tokens
- `config/` — Tier-1 customization configs (default-spec-loader, disabled features)
- `_headers` — Cloudflare Pages response headers (CSP `frame-ancestors`, etc.)
- `docs/` — fork rationale, CF Access policy guide, rebase runbook

## Customization commit convention

Customizations to upstream files (anything outside `archally/`) are tracked as
discrete commits with `[archally]` in the subject line, e.g.

    feat: [archally] read ?url= query param at boot
    feat: [archally] hide Save-to-GitHub action
    chore: [archally] strip telemetry init

Find every customization commit:

    git log --grep '\[archally\]'

This convention is what the upstream-sync rebase runbook checks after each merge
from `upstream/master` — see `docs/2026-04-26-rebase-runbook.md`.

## Upstream sync cadence

Monthly auto-sync PR via `.github/workflows/upstream-sync.yml`. Manual dispatch
also supported. Plan reference: `step-11b-asyncapi-studio-deployment.md` in the
sibling `blueprint-schema` repo, decision **D-11b-2** (auto-sync) and
**D-11b-impl-3** (cadence weekly → monthly).

## Required repo setup (one-time, manual)

Items the deploy and sync workflows expect to exist:

1. **Secrets** (Settings → Secrets and variables → Actions):
   - `CLOUDFLARE_API_TOKEN` — same value as in `blueprint-schema` repo
   - `CLOUDFLARE_ACCOUNT_ID` — same value as in `blueprint-schema` repo
   - `SYNC_PAT` — fine-grained PAT scoped to this repo (1-yr expiry recommended;
     permissions: `contents: read+write`, `pull-requests: read+write`,
     `workflows: read+write`)
2. **Label**: `upstream-sync` (color `#0E8A16`) — the workflow creates it via
   `gh label create` if missing, but pre-creating avoids the first-run permission
   prompt.
3. **Custom domain**: bind `studio.archally.dev` (or chosen subdomain of
   `*.archally.dev`) to the Pages project once the first deploy succeeds. See
   `docs/2026-04-26-cf-access-policy.md` for the full policy guide.
