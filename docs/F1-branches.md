# F1 branch separation and upstream contribution

## Branches

| Branch | Purpose | Upstream PR? |
| --- | --- | --- |
| `fix/android-auto-session-reactivation` | Playback-session fix only; based on upstream `7d2bd42e0fea98a796ba069b56c48ef07d874549` | Yes: use this as the head branch |
| `feature/android-auto-f1` | Clean fix plus F1 installation identity, projection coexistence wrappers, build workflow and test documentation | No |
| `master` | Fork's Actions launcher | No: it contains fork-specific workflow changes |

The clean commit is `fe8ca9cd6037b8a44a6666279e43136758b1058d`. It adds five lines in `fermata/src/main/java/me/aap/fermata/media/service/MediaSessionCallback.java`. The F1 branch descends from this commit and uses the same playback implementation.

## Where each change belongs

| Change | Location | Branch |
| --- | --- | --- |
| Reactivate inactive session on STATE_PLAYING | `MediaSessionCallback.java` | Clean fix, inherited by F1 |
| Unique projection component | `fermata/src/auto/java/me/aap/fermata/auto/f1/F1CarService.java` | F1 only |
| Unique Mirror component | `fermata/src/auto/java/me/aap/fermata/auto/f1/F1MirrorService.java` | F1 only |
| Select the unique service classes | `fermata/src/auto/AndroidManifest.xml` | F1 only |
| F1 labels | `fermata/src/main/res/values/strings.xml` | F1 only |
| Package suffix .f1, arm64 build and artifact verification | `.github/workflows/build-apk.yml` | F1 only |
| Test guide, branch guide and PR text | `docs/F1-testing.md`, `docs/F1-branches.md`, `docs/PR-session-reactivation.md` | F1 only |

The two F1 service classes are empty subclasses. The original CarService and MirrorService implementations are retained. Their purpose is side-by-side discovery in Android Auto, independently of the playback-session correction. These are test customisations in a release APK, not an Android Gradle debug build.

## Workflow for future changes

1. Implement production playback fixes on the clean fix branch.
2. Merge that branch into the F1 branch to test it. Do not merge the F1 branch into the clean branch.
3. Make package naming, service wrappers, diagnostics and test build changes only on F1.
4. Before opening an upstream PR, review the clean branch against the intended upstream base. The current expected diff is one Java file with five additions.
5. Copy the text from [PR-session-reactivation.md](PR-session-reactivation.md) into the PR body. Keep the description file on F1; it need not become an upstream source file.
6. Update the validation paragraph after extended testing. Do not mark unperformed checks as passed.

With remotes named `upstream` (AndreyPavlenko/Fermata) and `origin` (as8o/Fermata), the update direction is:

```sh
git fetch origin
git switch feature/android-auto-f1
git merge --no-edit origin/fix/android-auto-session-reactivation
```

To audit the currently tested fix against its pinned upstream base:

```sh
git diff --stat 7d2bd42e0fea98a796ba069b56c48ef07d874549 origin/fix/android-auto-session-reactivation
git diff 7d2bd42e0fea98a796ba069b56c48ef07d874549 origin/fix/android-auto-session-reactivation
git merge-base --is-ancestor origin/fix/android-auto-session-reactivation origin/feature/android-auto-f1
git diff origin/fix/android-auto-session-reactivation origin/feature/android-auto-f1 -- fermata/src/main/java/me/aap/fermata/media/service/MediaSessionCallback.java
```

Expected: one changed Java file in the first diff; a successful ancestor check; no output from the last diff. Fetch and compare the current upstream master before submission, since upstream may advance after testing. Avoid basing the upstream PR on this fork's master.

## Current test evidence

The reporter confirms that F1 projection and Mirror now appear alongside the enabled original Fermata after the unique service-class change. Initial steering-control testing appears successful; extended testing is pending. Visibility in the launcher does not by itself establish that all Mirror functions were tested.

The initial failure, before the class rename, and successful coexistence after it support a host discovery interaction. They do not reveal Android Auto's internal deduplication implementation.

The validated APK was produced by [Actions run 40416458239](https://github.com/as8o/Fermata/actions/runs/40416458239). The default-branch workflow launched that run and checked out the F1 test branch; its displayed head SHA therefore identifies the launcher, not necessarily the APK source. The artifact's `source-commit.txt` records the checked-out source used for the APK.
