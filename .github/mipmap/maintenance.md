# Maintaining the Mipmap fork

This repository tracks `pingdotgg/t3code`. Its `main` branch is the maintained Mipmap version, including upstream history and our customizations. Use short-lived feature branches and PRs into this fork's `main`; no extra permanent customization branch is needed.

## Rules for agents

- Keep each custom feature in a cohesive module within the existing application/package structure. Follow upstream conventions. Do not build a generic plugin framework just to isolate one feature.
- Keep changes to upstream-owned files limited to necessary integration points, such as navigation, route registration, typed RPC contracts, and service registration. Put feature logic in its own files; keep transport and registration code thin.
- Before changing shared infrastructure, check for an existing extension point. If a shared change is necessary, explain why in the PR and list the integration points that future upstream updates may affect.
- Do not mix a custom feature with unrelated refactors, dependency upgrades, formatting, file moves, or generated-file churn. Small diffs make upstream merges easier to assess.
- Follow existing boundaries for server services, contracts, and clients. Access skills on the destination environment's server through authorized service methods; do not assume the browser or desktop is on the same machine as the files.
- Keep reusable skill parsing/storage independent of provider-specific discovery paths and UI. Use small provider adapters where needed. Review support for web, desktop, mobile, remote environments, and each relevant provider.
- Test meaningful feature behavior and affected integration points. Include applicable checks and remaining limitations in the PR. Do not claim that a clean Git merge establishes behavioral compatibility.
- Preserve upstream history with merge commits. Never rebase or force-push shared `main`, automatically choose `ours`/`theirs`, or discard upstream fixes to make a conflict disappear.
- Resolve conflicts deliberately, preserving upstream fixes and the fork's intended behavior, then run affected checks. Keep unresolved updates as drafts.
- Keep upstream-facing improvements separable when practical. Publishing contributions to upstream still requires the maintainer's authorization.

## Updating upstream

`.github/workflows/mipmap-upstream-sync.yml` runs on pushes to this fork's `main`, including feature PR merges, and on manual dispatch. There is no schedule and no trigger for upstream-only changes. If nobody changes the fork, it stays at its current upstream version until a manual run.

The workflow checks the latest upstream `main` and does nothing if it is already included. Otherwise it prepares one draft PR on `mipmap/upstream-sync`. A pending update PR is left intact so a later run cannot overwrite someone's conflict resolutions. After that PR merges, the next check finds upstream already included unless new upstream commits arrived.

For a clean merge, the update branch contains a merge commit preserving fork changes. For conflicts, it contains upstream's head and the draft PR lists the conflicting files. Merge the fork's `origin/main` into that update branch, resolve conflicts, and validate before merging the PR.

The job defaults to `GITHUB_TOKEN` with repository contents and PR write permissions. Repository Actions settings must allow Actions to create pull requests. Mipmap's current organization/enterprise policy prohibits this, so the workflow instead prepares the update branch and puts a draft PR creation link and body in its job summary. Other API errors fail the job. It does not automatically approve or merge PRs.

For fully automatic PR creation, either have the organization administrator allow it or configure the optional `MIPMAP_UPSTREAM_SYNC_TOKEN` Actions repository secret with a GitHub App installation token or a fine-grained token scoped to this fork, with Contents and Pull requests write access. Workflow file updates may additionally need Workflows write access. An App token must be refreshed before expiry; a fixed installation token is not a permanent setup. Never commit tokens. Changes created with this optional token can trigger CI, unlike those created with `GITHUB_TOKEN`.

GitHub does not trigger push or PR CI for PRs created with `GITHUB_TOKEN`. A maintainer can close and reopen the draft PR to trigger PR CI, or explicitly run appropriate checks. Upstream workflows use custom Blacksmith runners and deployment credentials that this fork may not have; an absent, queued, or skipped check is not a pass. Configure fork-compatible runners and required checks before making updates eligible for automatic merging.

Run manually with `gh workflow run mipmap-upstream-sync.yml --repo mipmap-ai/t3code --ref main`.
