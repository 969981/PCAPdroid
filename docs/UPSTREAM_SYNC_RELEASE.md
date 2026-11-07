# Upstream sync and automatic APK pre-releases

## Repositories and branches

- Mirror: https://github.com/969981/PCAPdroid
- Canonical upstream: https://github.com/emanuele-f/PCAPdroid
- The mirror's default branch is `1.9.x`; the main upstream development branch is `master`.
- Both `master` and `1.9.x` are monitored. Other imported topic branches are left untouched.

## Scheduled workflow

Workflow: [upstream-sync-release.yml](../.github/workflows/upstream-sync-release.yml)

- Check every six hours at minute 17 UTC (01:17, 07:17, 13:17, 19:17 Asia/Singapore).
- Also support **Actions → Sync upstream and publish changed builds → Run workflow**.
- On its initial installation the source SHAs are treated as baselines and do not trigger releases.
- Manually select `publish_current=true` to build/publish the current source even if unchanged since the initial baseline.
- To avoid duplicate releases, each upstream commit gets its own tag:
  - `upstream-master-<12-hex-sha>`
  - `upstream-1-9-x-<12-hex-sha>`
- If an automatic build fails, the release is **not** created, and the next scheduled check retries the missing release.
- Releases are explicitly marked as **pre-releases**, and contain the debug APK plus a SHA-256 checksum.
- Build runs `./gradlew --no-daemon test assembleStandardDebug` with Java 21, Android SDK and the declared native build dependencies.
- The standard debug package uses the `.debug` application-ID suffix. It is not the official developer-signed release and cannot be treated as equivalent to the official APK.
- Releases occur only after tests and APK compilation succeed. There is no emulator UI/functional-playability test in this workflow.

## Synchronization safety

- `master`: fast-forward-only synchronization. If the local branch diverges, the job fails rather than rewriting user changes.
- `1.9.x`: the workflow lives on this default branch. The job merges new upstream commits into the local branch to **retain the workflow**. Merge conflicts fail without force-pushing.
- Both jobs require shared history; force rewrites and unrelated histories are rejected.
- A source build checks out the exact upstream commit, rather than an unverified merged snapshot.
- A release is skipped if the upstream commit has not yet been mirrored into the user's repository.
- GitHub Actions uses the repository `GITHUB_TOKEN` with `contents: write` only where needed, without a separate PAT. The build checkout does not persist credentials.

## Validation

1. Check the workflow on the Actions tab of the mirror repository.
2. On first execution with the two initial source SHAs, both sync checks should report no new changes and publish no release.
3. After a real upstream commit appears, expect a fast-forward or safe merge, an Android build, and a new pre-release under the Releases tab.
4. If a build or release fails, inspect the run logs. A successful source mirror alone does not prove an APK was published.
5. Scheduled jobs can be delayed by GitHub and can be disabled by GitHub after prolonged inactivity; inspect the Actions tab periodically.

If you later change the mirror's default branch, copy the workflow to the new default branch before switching, because GitHub schedules execute from the repository's default branch.
