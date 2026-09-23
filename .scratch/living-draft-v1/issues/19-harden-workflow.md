# 19: Harden the complete workflow

**What to build:** Maintainers can trust the completed design-to-handoff workflow across interaction, visual output, PDF output, process failures, and local server security boundaries.

**Blocked by:** 13: Browse complex composites; 14: Browse communication and agent components; 16: Hand an exported artifact to a coding agent; 17: Continue an OpenCode session from Studio; 18: Promote a Custom Component for shared use

**Status:** ready-for-agent

- [ ] Accessibility and keyboard smoke checks cover representative interactive demos.
- [ ] Visual regression covers supported themes and viewport classes.
- [ ] PDF snapshot checks detect export regressions.
- [ ] Supervisor tests cover host/origin validation, recovery, crashes, stale processes, and cleanup.
