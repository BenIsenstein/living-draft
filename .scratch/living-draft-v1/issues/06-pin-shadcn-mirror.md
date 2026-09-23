# 06: Pin and synchronize the shadcn mirror

**What to build:** Maintainers can reproduce the exact Base UI and Base Nova component source used by Living Draft and deliberately review future upstream changes.

**Blocked by:** None (can start immediately)

**Status:** ready-for-agent

- [ ] The mirror records the shadcn commit, CLI and package versions, preset, sync date, and generated hashes.
- [ ] Canonical component source builds with no unexplained local divergence.
- [ ] A sync check reports added, removed, source-changed, dependency-changed, and token-changed components.
- [ ] Updating the mirror never follows an implicit latest version.
