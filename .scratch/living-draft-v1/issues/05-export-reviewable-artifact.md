# 05: Export a reviewable artifact

**What to build:** A user or agent can export an artifact into portable source, deterministic initial-state HTML, its manifest, local assets, and a visually faithful PDF.

**Blocked by:** 04: Preview, apply, and save artifact themes

**Status:** ready-for-agent

- [ ] Export validates source and freezes the saved theme snapshot.
- [ ] HTML is self-contained when feasible and copies larger assets with relative paths.
- [ ] Local Fontsource and managed Google font assets render without remote requests.
- [ ] Chromium emits a Letter PDF with backgrounds after fonts and layout settle.
