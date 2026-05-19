# Contributing to docker-netbootxyz

Thanks for your interest in contributing. This repo builds the official `netbootxyz/netbootxyz` container image.

## Pull request CI

The `build` workflow (`.github/workflows/build.yml`) runs on every PR against `master`. What happens depends on whether the PR is from a branch in this repo or from a fork.

### Same-repo PRs

PRs opened from a branch in `netbootxyz/docker-netbootxyz` get the full pipeline:

1. Multi-arch build for `linux/amd64` and `linux/arm64`.
2. Per-arch digests pushed to GitHub Container Registry.
3. A multi-arch manifest pushed to both Docker Hub and GHCR, tagged `pr-<number>` and `pr-<number>-<sha>`.
4. A comment is posted on the PR with `docker pull` / `docker run` instructions for testing the image.

### Fork PRs

PRs opened from a forked repo get a **build-only** verification:

1. Multi-arch build for `linux/amd64` and `linux/arm64` (no push).
2. A comment is posted on the PR confirming the build succeeded and pointing maintainers at the publish steps below.

No image is pushed, because GitHub deliberately withholds repository secrets (Docker Hub / GHCR tokens) from workflow runs triggered by fork PRs. This is a security boundary — without it, any opened PR could exfiltrate the registry credentials.

## Publishing a test image for a fork PR (maintainers)

After reviewing the diff, push the PR head to a branch in this repo. That fires the same `build` workflow with secrets available, producing a `pr-<number>` test image:

```bash
gh pr checkout <PR-number>
git push origin HEAD:test/pr-<PR-number>
```

The build will tag the resulting image `pr-<PR-number>` on both Docker Hub and GHCR. To clean up afterward, delete the test branch:

```bash
git push origin --delete test/pr-<PR-number>
```

### Alternative: `workflow_dispatch`

The `build` workflow also accepts a manual trigger. From the **Actions → build → Run workflow** menu, pick a branch and optionally set `tag_suffix` to label the resulting image (e.g. `tag_suffix=fix-healthcheck` produces `test-fix-healthcheck`). Useful for one-off rebuilds of a branch without a PR.
