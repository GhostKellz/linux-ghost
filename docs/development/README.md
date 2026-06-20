# Development

Maintainer notes for linux-ghost.

## Release Prep

For a kernel version bump:

1. Update `_major`, `_minor`, and `_cachyos_tagrel` in `PKGBUILD`.
2. Verify the CachyOS source tag exists and patch source paths match the target major version.
3. Refresh `config/config` with the target source tree and reapply `config/ghost.fragment`.
4. Test that local patches in `patches/` apply in `prepare()`.
5. Build at least one default package with `makepkg -sf`.
6. Update `CHANGELOG.md`, `SECURITY.md`, and version-specific docs.

## Patch Policy

- Keep linux-ghost maintained patches in `patches/`.
- Prefer URL references for upstream CachyOS and linux-tkg patches.
- Document every local patch in [../../patches/README.md](../../patches/README.md).
- Avoid carrying stale local copies of upstream patches unless linux-ghost needs local modifications.

## Config Policy

- Keep the full base config in `config/config`.
- Put linux-ghost-specific overrides in `config/ghost.fragment`.
- Prefer `ghost.fragment` for project policy changes so version refreshes are easier to audit.
- When a Kconfig symbol is removed or renamed upstream, update docs in the same change as the config refresh.
