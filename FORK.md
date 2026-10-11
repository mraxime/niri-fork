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

## Paru installation and updates

`niri-max-git` replaces stock `niri` through package conflicts. It provides the
actual upstream `niri` version and `wayland-compositor`, uses native names, and includes
upstream portals, documentation, and shell completions. User configuration stays
unchanged. Restart the Niri session when ready to use the installed build.

Add to `~/.config/paru/paru.conf`, preserving other entries:

```ini
Include = /etc/paru.conf

[niri-max]
Url = https://github.com/mraxime/niri-fork.git
Path = packaging/aur
```

Keep `Devel` enabled in the included system configuration. Paru discovers the
PKGBUILD on the repository's default `fork` branch; its Git source explicitly
tracks `fork`, including rewritten heads. No AUR registration is needed.

```sh
paru -S niri-max-git
paru -Syu
```

The first command builds and installs, prompting to remove conflicting stock
`niri`. The second refreshes repositories and checks Git-source updates. Package
versions combine niri's native version with commit time and hash, without tags.
Commit time keeps versions ordered when revision counts shrink during rebases.
Paru records the installed Git ref during installation; package review stays enabled.

For a build without installation:

```sh
git clone https://github.com/mraxime/niri-fork.git
cd niri-fork/packaging/aur/niri-max-git
makepkg
```

To update a local checkout after automatic rebases, save local changes, fetch
`origin`, and reset the clean `fork` branch to `origin/fork` before rebuilding.

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
