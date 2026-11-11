# client-kit

The shared docs every 3DWD client website follows to show the content its site owner manages in the Headless CMS, and the sync that keeps each client repo's copy identical.

## What's here

- `docs/cms/`: the client guide. `client-website.md` is the base (environment variables, the API, fetching CMS data, images, the revalidate route, testing). Each ability has its own doc that builds on it: `posts.md` and `galleries.md`, and one more per future ability. Every client repo gets the whole folder, whatever abilities its site uses; its own `docs/site.md` says which ones it reads.
- `clients.json`: the client repos the folder is synced to, and whether each one's sync pull requests auto-merge.
- `.github/workflows/sync.yml`: the sync.
- `templates/ci.yml`: the CI every client repo copies as `.github/workflows/ci.yml` when it is onboarded. It isn't synced.

The facts in `docs/cms/` come from the Headless CMS backend (`rmarafioti/3d_wd_backend`), whose `docs/endpoints.md` is the source of truth. This repo is maintained from the backend's Claude session, and its `/doc-sync` checks `docs/cms/` against the backend's docs.

## The rules

- `docs/cms/` is changed only here, never in a client repo. A client repo's CI fails when its copy differs from this repo's `main`.
- Nothing site-specific goes in `docs/cms/`. A site's name, pages, design, quirks, media folder and abilities in use belong in that repo's `docs/site.md`.
- Each ability doc links to the base's sections and never copies its code.

## How the sync works

On a push to `main` that changes `docs/cms/`, the Sync workflow runs once per repo in `clients.json`. It acts as the `3dwd-sync` GitHub App, with a token that lasts about an hour.

- A repo whose `docs/cms/` already matches is skipped.
- If the repo has an open sync pull request (branch `3dwd-sync/…`), it gets a new commit.
- Otherwise a new pull request is opened by `3dwd-sync[bot]`.
- With `autoMerge: true`, the pull request is set to auto-merge, so it merges once the repo's required CI checks pass.

**The auto-merge exception:** every pull request waits for Rich, except a sync pull request. That one merges on its own, and only when all of these hold:

- it comes from `3dwd-sync[bot]` on a `3dwd-sync/` branch;
- it changes nothing outside `docs/cms/`, which the client repo's own CI checks;
- the repo's required checks pass.

Set `autoMerge: true` only once the repo's ruleset requires those checks. Without required checks, auto-merge would merge at once.

A docs-only merge doesn't redeploy the site: each client's Vercel project skips the build when only `docs/` changed.

While a sync pull request is waiting, the drift check fails on the repo's other pull requests until the sync merges.

**Dry run:** Actions → Sync → Run workflow, with "dry run" ticked (the default). The run reports which repos differ and opens nothing.

## Adding a client repo

The steps are in the backend's `docs/deploy.md` → Onboarding a client website. In short:

1. Install the `3dwd-sync` App on the repo.
2. Add the repo to `clients.json` with `autoMerge: false`.
3. Merge the first sync pull request by hand.
4. In the client repo, add:
   - `docs/site.md`;
   - the `CLAUDE.md` imports;
   - `docs/cms/` in `.prettierignore`;
   - `.github/workflows/ci.yml` from `templates/ci.yml`.
5. Add a ruleset on `main` requiring the `cms` and `check` checks, and the Vercel Ignored Build Step.
6. Set `autoMerge: true`.

## Secrets

These are set in this repo's Actions secrets:

- `SYNC_CLIENT_ID`: the App's Client ID.
- `SYNC_APP_PRIVATE_KEY`: the App's private key. It is kept only in 1Password and in this secret.
