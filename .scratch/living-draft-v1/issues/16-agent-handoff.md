# 16: Hand an exported artifact to a coding agent

**What to build:** A coding agent can consume a Living Draft export and reliably recreate its components, theme, fonts, and portable TSX in a target project.

**Blocked by:** 05: Export a reviewable artifact; 15: Generate accurate artifact dependency inventories

**Status:** ready-for-agent

- [ ] The Living Draft skill directs agents to current CLI guidance instead of embedding volatile instructions.
- [ ] The skill installs listed shadcn components rather than copying canonical UI source.
- [ ] The skill applies theme and font metadata to the target project's shadcn CSS setup.
- [ ] A representative export renders consistently after handoff to a separate target project.
