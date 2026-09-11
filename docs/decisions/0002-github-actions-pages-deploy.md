# 0002 — Deploy GitHub Pages via Actions, not the legacy branch builder

**Date:** 2026-08-18  **Status:** accepted, implemented

## Context

The first Pages deploy used the default "deploy from a branch" (legacy Jekyll)
builder. It failed four consecutive times with only "Page build failed."

## What was tried

| Attempt | Change | Result |
|---|---|---|
| 1 | original mirror | errored |
| 2 | removed `?` from filenames (wget had saved query strings into font names) | errored |
| 3 | replaced `+` and `,` in PageSpeed filenames with `_` | errored |
| 4 | removed the `CNAME` file | errored |

Attempts 2 and 3 were kept regardless — the `?` filenames made the FontAwesome
fonts unreachable over HTTP, a real bug.

A later legacy build did succeed, so the failures were likely transient on
GitHub's side. By then the Actions workflow was in place and is the supported
path anyway.

## Decision

`.github/workflows/pages.yml` uploads the repo root as a Pages artifact with no
Jekyll processing. Pushing it required adding the `workflow` scope to the `gh`
OAuth token (`gh auth refresh -s workflow`), which the owner did.

## Consequences

- With Actions deploys, the custom domain is stored in Pages settings, not
  written to a `CNAME` file. The file is kept in the repo anyway as a safeguard.
- `.nojekyll` is retained; harmless either way.
- GitHub has flagged that the pinned action versions target Node 20, which is
  being retired from runners. Bump `actions/*` versions when convenient.
