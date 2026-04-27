# Cloudflare Access policy — archally-asyncapi-studio

The architecture-viewer iframes Studio. The browser only sends Cloudflare
Access cookies into the iframe when:

1. Both origins are CF Access-protected.
2. Both share an Access **Application** (or Application Group) — same session
   settings, same identity provider.
3. SameSite cookies are issued under a shared parent domain (`.archally.dev`).

Mismatched policies cause the iframe to bounce through the CF Access login on
every page view, breaking the experience.

## One-time dashboard setup (in order)

1. **First deploy.** Run `.github/workflows/deploy-cf.yml` (manual dispatch). The
   workflow calls `wrangler pages project create archally-asyncapi-studio` (idempotent),
   then deploys. Successful run yields a `*.archally-asyncapi-studio.pages.dev`
   preview URL.
2. **Bind custom domain.** Cloudflare dashboard → Pages → `archally-asyncapi-studio`
   → Custom domains → Add `studio.archally.dev` (or chosen subdomain). DNS
   provisioning is automatic if Cloudflare manages the zone.
3. **Add hostname to existing Access Application.** Zero Trust → Access →
   Applications → find `archally-architecture-${project}` (or whatever you named
   the architecture-viewer app). Edit → Application configuration → Application
   domain → add `studio.archally.dev`. Save.
4. **Verify.** Open `https://studio.archally.dev/?url=https://example.com/spec.yaml`
   in an incognito window. Expected: one CF Access login, then Studio loads. Open
   the architecture-viewer's AsyncAPI tab in the *same* browser session: Studio
   should load inside the iframe with **no second login prompt**.

## Alternative: Application Group (recommended once you add a third app)

If you'll have ≥3 CF Access-protected viewers, create an **Application Group**:

1. Zero Trust → Access → Application Groups → Create `archally-viewers`.
2. Move `archally-architecture-${project}` into it.
3. Add `archally-asyncapi-studio` to it.
4. Define identity-provider rules at the group level → policy edits propagate.

## CSP / iframe headers

Live in `archally/_headers` (CF Pages picks up `_headers` at the deploy root).
The deploy workflow copies it from `archally/_headers` into `apps/studio/build/_headers`
before `wrangler pages deploy`.

Current rule:

    /*
      Content-Security-Policy: frame-ancestors https://*.archally.dev https://*.pages.dev

`*.pages.dev` is overly permissive (any CF Pages site can iframe Studio). Tighten
to `https://architecture.archally.dev` (or the precise hostname pattern) once the
architecture-viewer is bound to a stable subdomain.

## Token rotation

`CLOUDFLARE_API_TOKEN` is shared between this repo and `blueprint-schema`.
Rotation procedure:

1. CF dashboard → My Profile → API Tokens → Roll the existing token.
2. Update **two** GitHub repo secrets:
   - `AdamWalkowski/blueprint-schema` → Settings → Secrets → `CLOUDFLARE_API_TOKEN`
   - `AdamWalkowski/archally-asyncapi-studio` → Settings → Secrets → `CLOUDFLARE_API_TOKEN`
3. Re-dispatch one deploy workflow on each repo to verify the new token works.

Calendar reminder: rotate annually or on any suspected leak.
