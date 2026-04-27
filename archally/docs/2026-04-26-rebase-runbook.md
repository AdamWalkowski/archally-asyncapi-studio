# Rebase runbook — handling the monthly upstream-sync PR

The auto-sync workflow opens a PR on the 1st of each month merging
`upstream/master` into `master`. This runbook is for the human reviewer.

## Happy path

1. CI on the PR is green.
2. `git log --grep '\[archally\]'` (run locally or in the PR) shows all expected
   customization commits.
3. Approve → merge (squash off; preserve commits so customizations remain
   discoverable).

## When the rebase failed → workflow fell back to merge

Symptom: PR contains a merge commit and conflict markers in some files.

1. Check out the sync branch locally:

       git fetch origin
       git checkout upstream-sync/YYYY-MM-DD

2. Identify which `[archally]` commits collided:

       git log --grep '\[archally\]' --merges --first-parent

3. Resolve conflicts file by file. Heuristics:
   - Upstream restructured a file we customized → reapply our change to the new
     location, drop the old hunk.
   - Upstream changed a feature we hide → if hide still works, no action; if hide
     mechanism broke, update `archally/config/disabled-features.json` AND the
     upstream patch.
   - Upstream changed `next.config.js` → re-verify `output`/`distDir`/`basePath`
     still pass through `NEXT_CONFIG_OUTPUT` / `PUBLIC_URL` env vars correctly.

4. Verify locally:

       corepack enable
       pnpm install --frozen-lockfile
       NEXT_CONFIG_OUTPUT=export PUBLIC_URL="" pnpm run build:studio
       ls apps/studio/build/  # expect index.html + _next/

5. Push and re-trigger CI:

       git add -A
       git commit -m "chore: [archally] resolve upstream-sync conflicts ($(date +%Y-%m-%d))"
       git push origin upstream-sync/YYYY-MM-DD

## Files most likely to conflict

| File | Why | Recovery hint |
|---|---|---|
| `apps/studio/next.config.js` | We may pin webpack fallbacks or polyfills | Reapply our changes after upstream's |
| `apps/studio/src/state/files.state.ts` | C-2 (`?url=` loader) lives here once Tier-1 lands | Check `git blame` on our hunk; reapply |
| `apps/studio/src/components/Sidebar/...` | C-6 (hide Save-to-GitHub) | CSS-based hides survive; component-tree edits don't |
| `apps/studio/package.json` | Upstream bumps deps; we usually don't add | Take upstream version; verify build |
| `apps/studio/public/_redirects` | Upstream owns it; we don't touch | Take upstream |
| `apps/studio/public/index.html` (or `app/layout.tsx`) | C-5 (logo) | Reapply our logo swap after upstream's edits |

## Reverting a bad sync

If the merged sync turns out to break production:

    git revert -m 1 <merge-commit-sha>
    git push origin master

Then dispatch `deploy-cf.yml` to re-deploy the previous good state.

## When to consider abandoning a sync

If conflicts are dense or upstream made a breaking architectural change (e.g.
migration to a new framework), close the PR with a note and:

1. Open a tracking issue documenting the upstream change.
2. Defer until we decide whether to follow.
3. The next month's auto-sync will reopen with cumulative diff — at that point,
   reassess.
