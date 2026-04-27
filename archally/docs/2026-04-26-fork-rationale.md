# Fork rationale — archally-asyncapi-studio

**Status:** active fork of [`asyncapi/studio`](https://github.com/asyncapi/studio).
**Why:** the architecture-viewer (in
[`AdamWalkowski/blueprint-schema`](https://github.com/AdamWalkowski/blueprint-schema))
embeds Studio in an iframe. The hosted instance at `studio.asyncapi.com` cannot
share a Cloudflare Access session with the architecture-viewer, exposes
upstream features irrelevant to read-only consumption (Save-to-GitHub, "Open
file"), and offers no build-time config injection.

Self-hosting this fork solves all three. See `step-11b-asyncapi-studio-deployment.md`
in the sibling repo for the locked-decision table (D-11b-1..6) and full plan
context.

## Scope of this fork

- Add `archally/` (theme, config, docs, response headers)
- Add `.github/workflows/deploy-cf.yml` (Cloudflare Pages deploy)
- Add `.github/workflows/upstream-sync.yml` (monthly upstream-merge PR)
- Discrete commits to upstream files, tagged `[archally]` in the subject

## Out of scope

- Renaming to `main` (upstream is `master`; we follow upstream)
- Changes to upstream's existing 27 workflows (they continue to operate,
  unmodified)
- Republishing as an npm package (Studio publishes; we don't)

## Upstream attribution

This is a fork of Apache-2.0 licensed software. The original `LICENSE` file is
preserved unchanged. Footer attribution to upstream is added as part of
Tier-2 customization **C-8** in a follow-up step.
