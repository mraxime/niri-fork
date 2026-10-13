# Fork

A sole tiled window fills the work area. Population changes reapply automatic
sizing; manual sizing releases it until the next population change. Explicit
opening intent and fullscreen behavior stay native. The fork enables this policy
at startup and preserves it across configuration reloads.

Fork-only work: upstream prohibits LLM-created contributions.

## Branches and sync

- `main`: unmodified `niri-wm/niri:main` mirror.
- `fork`: default branch; feature, local defaults, automation, packaging.

The scaffold in `.github/workflows/fork-sync.yml` checks pushed SHAs without
publishing. Six-hour scheduled runs skip checks when upstream is already included;
manual runs always rebase, check, and publish. Automatic rebases retain authors
and use the GitHub Actions bot as committer.

The single job runs upstream's Linux checks, randomized/slow tests, and
Alpine/Fedora/FreeBSD builds. Nix remains advisory. Successful syncs publish both
heads atomically, with a patch lease and a fast-forward-only mirror. Conflicts or
failed required checks leave both published heads unchanged.

GitHub schedules can be delayed or disabled after 60 days of public-repository
inactivity. Keep Actions failure notifications enabled in your GitHub settings.

## Build and update

```sh
git clone https://github.com/mraxime/niri-fork.git
cd niri-fork
makepkg
```

The co-installable package depends on stock `niri`. It installs separate
`niri-fork`, `niri-fork-session`, `niri-fork.service`, and `niri-fork.desktop` paths;
stock niri and user configuration stay unchanged. Installation is optional:
`sudo pacman -U <built-package>`. No AUR or automatic package updates are configured.

After automatic rebases, update a **clean** checkout:

```sh
git fetch origin
git switch fork
git reset --hard origin/fork
makepkg --force
```

Save local changes before resetting. Package versions use commit time because
revision counts can shrink during rebases.

## Conflict recovery

Disable the sync workflow in Actions while resolving locally. A fresh clone needs
`git remote add upstream https://github.com/niri-wm/niri.git`.

```sh
git fetch origin
git fetch upstream main
git switch fork
git branch recovery/fork-before-rebase origin/fork
expected=$(git rev-parse origin/fork)
git reset --hard "$expected"
git rebase upstream/main
# Resolve conflicts, stage scoped files, git rebase --continue; run checks.
git push --force-with-lease="refs/heads/fork:$expected" origin fork
```

Re-enable the sync workflow and dispatch it manually to check and update the
mirror. Rebase upstream changes; keep mirror updates fast-forward-only.
