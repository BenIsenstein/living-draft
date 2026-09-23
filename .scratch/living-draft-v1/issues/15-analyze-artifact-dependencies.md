# 15: Generate accurate artifact dependency inventories

**What to build:** Saving or exporting an artifact automatically discovers everything a coding agent must install or carry over, without relying on a manually maintained list.

**Blocked by:** 06: Pin and synchronize the shadcn mirror; 10: Browse selection and toggle controls

**Status:** ready-for-agent

- [ ] Static import analysis follows Custom Components recursively.
- [ ] The manifest separates direct and transitive shadcn components.
- [ ] Packages, utilities, fonts, and assets are recorded.
- [ ] Validation fails clearly on unresolved or unsupported imports.
