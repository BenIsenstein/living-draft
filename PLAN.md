# Living Draft Implementation Plan

## Product Definition

**Name:** Living Draft  
**Tagline:** Design in code, by conversation.  
**Editor:** Living Draft Studio  
**CLI:** `living-draft`  
**Private package scope:** `@living-draft/*`

Living Draft produces executable design artifacts rather than mockups. The canonical artifact is React TSX composed from real shadcn/ui-compatible components. Astro is the artifact runtime wrapper, providing a Vite-powered development server and two output modes:

- Static artifacts render through Astro with no `client:*` directive. Astro emits HTML and CSS without shipping the React runtime or component JavaScript.
- Interactive artifacts render as one React island, initially with `client:load`. The entire artifact hydrates as one portable React root.

The artifact TSX does not import Astro code. It can therefore move directly into a Vite React SPA, Next.js project, or another React application. Only data access, effects, mutations, routing integration, and other application behavior should remain for a coding agent to implement.

## Product Principles

1. Use real code as the design source of truth.
2. Keep artifact TSX portable across Astro and React applications.
3. Mirror pinned shadcn/ui source with near-zero local divergence.
4. Prefer canonical shadcn composition, CVA, theme-aware utilities, and APIs.
5. Permit arbitrary Tailwind classes only when canonical methods are insufficient.
6. Keep the authoring loop local-first, CLI-controlled, and agent-friendly.
7. Derive HTML, PDF, dependency data, and previews from source. Never hand-edit generated output.
8. Separate in-memory preview, baseline theme application, artifact save, and export.
9. Keep canonical UI source separate from Living Draft fixtures and editor concerns.
10. Make every filesystem mutation explicit in output, atomic, and recoverable. Never create git commits automatically.

## Resolved Technology Decisions

### Component Source

- React 19-compatible TypeScript components.
- Current shadcn/ui Base UI lineage.
- Base Nova style.
- Tailwind CSS v4.
- CSS-variable shadcn theme system.
- `class-variance-authority` for variants.
- Current shadcn `cn` package rather than the legacy local `clsx` plus `tailwind-merge` helper.
- Modern evergreen Chrome, Safari, Firefox, and Edge.

Base UI is canonical because it is shadcn's current greenfield default. Radix remains mature and familiar, and React Aria remains attractive for advanced accessibility and internationalized collection behavior, but maintaining parallel bases would create multiple incompatible source APIs and defeat 1-to-1 transfer.

### Runtime And Studio

- Astro is the generated artifact wrapper and Vite-powered artifact dev server.
- Living Draft Studio is a React SPA.
- A Node core package owns configuration, artifacts, themes, exports, dependency analysis, and process supervision.
- A Node CLI exposes the core API to people and agents.
- Studio embeds the artifact dev-server URL in a plain iframe.
- Selection and visual annotation are deferred. Conversation is the v1 feedback mechanism.
- One supervisor manages Studio and artifact processes, ports, logs, reuse, and idle shutdown.
- Project configuration lives in `living-draft.json` and uses relative paths only.

### Outputs

- Canonical TSX source and artifact-local assembled components.
- `artifact.json` containing metadata, theme values, font descriptors, and direct/transitive dependency data.
- Static initial-state HTML, self-contained when practical.
- Chromium-generated PDF using web layout, Letter paper, backgrounds enabled, and sensible print margins.
- Large assets copied with relative paths; assets below configured thresholds may be inlined.

## Proposed Workspace

```text
living-draft/
  apps/
    studio/                    # React SPA editor and component explorer
  packages/
    cli/                       # living-draft executable
    core/                      # framework-neutral Node API
    ui/                        # pinned Base Nova shadcn mirror
    catalog/                   # fixtures, state matrices, hydrated demos
    shared-components/         # explicitly promoted reusable custom components
    artifact-runtime/          # generated Astro wrapper templates and bridge
    schemas/                   # config, artifact, theme, and protocol schemas
    opencode-adapter/          # first direct agent-provider integration
  artifacts/
    examples/                  # first-party example artifacts
  themes/
    base-nova-neutral.json
    warm-yoga.json
    active.css                 # generated, imported by the runtime baseline
  skills/
    living-draft/              # agent-facing workflow and export unpacking
  scripts/
    sync-shadcn.ts
    verify-upstream.ts
  living-draft.json
  components.json
  pnpm-workspace.yaml
```

Use pnpm workspaces. Keep package APIs independently testable even if the first release is distributed as one CLI package.

## Artifact Layout

An authored artifact starts as a folder in this workspace or any external project:

```text
artifacts/customer-dashboard/
  artifact.tsx                 # one canonical React root
  artifact.json                # source manifest and saved theme snapshot
  components/                  # artifact-owned Custom Components
  assets/                      # images and other artifact-owned media
```

The CLI generates runtime files outside exported source:

```text
.living-draft/
  runtime/
    astro.config.mjs
    src/pages/index.astro
    src/styles/global.css
    src/styles/active.css
    package.json
  state.json
  logs/
```

The generated Astro page chooses exactly one rendering form from `artifact.json`:

```astro
<Artifact />
```

or:

```astro
<Artifact client:load />
```

Each artifact declares exactly one mode: `static` or `interactive`.

## Manifest Contract

`artifact.json` is the handoff contract. Theme values are embedded directly in this file; no separate theme JSON or generated CSS is included in an export. The Living Draft skill teaches a coding agent to apply these values to the target project's shadcn theme CSS.

Illustrative shape:

```json
{
  "$schema": "https://living-draft.dev/schemas/artifact.v1.json",
  "schemaVersion": 1,
  "slug": "customer-dashboard",
  "title": "Customer Dashboard",
  "description": "Account health and activity overview",
  "status": "draft",
  "mode": "static",
  "entry": "./artifact.tsx",
  "viewport": {
    "width": 1440,
    "height": 1000
  },
  "theme": {
    "name": "customer-dashboard",
    "base": "base-nova",
    "radius": "0.625rem",
    "light": {
      "background": "oklch(1 0 0)",
      "foreground": "oklch(0.145 0 0)"
    },
    "dark": {
      "background": "oklch(0.145 0 0)",
      "foreground": "oklch(0.985 0 0)"
    },
    "fonts": {
      "sans": {
        "family": "Inter Variable",
        "source": "fontsource",
        "package": "@fontsource-variable/inter",
        "fallback": "ui-sans-serif, system-ui, sans-serif"
      }
    }
  },
  "dependencies": {
    "direct": ["button", "card", "table"],
    "transitive": ["separator", "tooltip"],
    "packages": ["class-variance-authority", "lucide-react"],
    "utilities": ["cn"],
    "source": {
      "base": "base-ui",
      "style": "base-nova",
      "shadcnCommit": "<sha>",
      "shadcnCli": "<version>"
    }
  },
  "export": {
    "paper": "Letter",
    "printBackground": true
  }
}
```

The real schema must contain the complete current shadcn token set, including card, popover, primary, secondary, muted, accent, destructive, border, input, ring, chart, and sidebar pairs for light and dark modes. Fonts support system stacks, Fontsource packages, and managed downloaded Google Font assets with source and license metadata.

## Theme Model

### Initial Presets

- `base-nova-neutral`: active default.
- `warm-yoga`: derived from the supplied reference stylesheet's token values and font metadata.

Do not copy the reference stylesheet's global serif `h1`-`h4` rules, smoothing preferences, or other site-specific base typography into canonical component behavior.

### Four Distinct Actions

1. **Preview:** apply validated CSS variables to the artifact iframe/root in memory. No disk write.
2. **Apply:** write the current theme as the Studio/runtime baseline in generated `themes/active.css` or the configured active-theme file. `global.css` imports this stable generated file.
3. **Save:** validate and update artifact manifest metadata, including the current complete theme snapshot. Source edits already exist on disk through the coding agent and Vite HMR.
4. **Export:** produce a clean handoff folder, dependency graph, standalone HTML where feasible, copied large assets, and PDF.

These actions write files atomically but never stage or commit git changes.

### Font Handling

- Prefer Fontsource npm packages for deterministic local web and PDF rendering.
- Allow managed Google Font download when needed.
- Store font source, family, style, weight, package/version or local asset references, fallback stack, and license metadata in the manifest.
- Wait for `document.fonts.ready` before HTML capture or PDF generation.
- Do not depend on remote font requests in final HTML or PDF output.

## Core API

The core package should expose typed functions without depending on CLI prompts or Studio UI:

```ts
readConfig(projectRoot)
writeConfig(projectRoot, config)
discoverArtifact(path)
readArtifact(path)
validateArtifact(path)
createArtifact(options)
readTheme(path)
previewTheme(session, theme)
writeActiveTheme(projectRoot, theme)
saveArtifact(path, manifest)
analyzeDependencies(path)
createRuntime(path, options)
startArtifactServer(path, options)
startStudio(options)
stopSession(path)
exportArtifact(path, options)
    promoteCustomComponent(path, component)
```

All write APIs must validate paths against an explicit project/artifact root, use temporary files plus rename for atomicity, and return a structured mutation report.

## CLI Surface

Initial commands:

```text
living-draft init
living-draft create <slug>
living-draft open <path>
living-draft list
living-draft inspect <path>
living-draft apply-theme <path-or-name>
living-draft save <path>
living-draft export <path>
living-draft promote <custom-component>
living-draft stop [path]
living-draft doctor
```

`living-draft open <path>` should:

1. Resolve a canonical artifact path and nearest `living-draft.json`.
2. Validate the artifact manifest and entry module.
3. Detect the package manager and automatically install missing declared dependencies in the artifact project.
4. Generate or refresh `.living-draft/runtime`.
5. Start or reuse the supervised Astro/Vite artifact server.
6. Start or reuse Living Draft Studio.
7. Register provider/session launch context when available.
8. Open Studio to the artifact session unless `--no-open` is passed.
9. Print machine-readable URLs, process IDs, mutations, and next actions.

Automatic installs must be reported exactly, remain inside the target project, and never silently modify unrelated files.

## Studio And Server Protocol

### V1 Transport

- REST over HTTP for config reads, file commands, theme actions, exports, and presence.
- Artifact code and visual updates use Astro's Vite HMR directly.
- Studio receives a preview URL and embeds it in an iframe.
- Direct chat uses an agent-provider client API when a supported launch context is present.
- Unsupported providers use a queued prompt fallback surfaced by provider hooks.

The REST API should be versioned from the start, for example `/api/v1/...`, because Studio, the artifact server, and agent provider may later run on different machines.

### Process Supervisor

- One supervisor owns Studio and all active artifact servers.
- Sessions are keyed by canonical artifact path, not opaque user-facing IDs.
- Allocate ports dynamically and persist runtime state outside artifact source.
- Reuse healthy processes.
- Keep bounded logs.
- Stop after a configurable idle period when no Studio client or provider is attached.
- Bind to loopback by default.
- Validate `Host`, `Origin`, and session capability on mutating endpoints.

### Agent Session Attachment

Implement a provider interface rather than hard-coding Studio to one agent:

```ts
interface AgentProvider {
  id: string
  detect(environment: NodeJS.ProcessEnv): LaunchContext | null
  connect(context: LaunchContext): Promise<AgentConnection>
  sendPrompt(connection: AgentConnection, prompt: AgentPrompt): Promise<void>
  getPresence(connection: AgentConnection): Promise<AgentPresence>
}
```

OpenCode is the first direct adapter. The launch handshake should explicitly pass provider ID, session identity, project root, and callback/capability data through environment variables or an agreed launch payload. Do not rely on parent-process inference.

A direct attachment depends on APIs exposed by the agent host. Start implementation with a short OpenCode capability spike. If the active session cannot accept messages programmatically, preserve the same provider interface and implement an OpenCode hook that surfaces queued Studio prompts in the active/new agent session.

## Shadcn Mirror Strategy

### Pinning

Record all of the following in a machine-readable lock file:

- shadcn/ui repository commit SHA.
- shadcn CLI version.
- Base UI package versions.
- Base Nova preset/configuration.
- React, Tailwind, and `cn` versions.
- Generated component file hashes.
- Sync date and migration notes.

### Import Workflow

1. Initialize a clean Base UI + Base Nova + Tailwind v4 project through the pinned shadcn CLI.
2. Add every component present in the pinned documented Base UI index.
3. Copy generated canonical files into `@living-draft/ui` without stylistic rewrites.
4. Keep upstream imports and CVA definitions intact.
5. Generate the provenance lock and hashes.
6. Put all Living Draft fixtures, static overlay renderers, metadata, and demos in `@living-draft/catalog`.
7. Validate typecheck and build.

At the September 2026 research point, the documented Base UI index contains 64 entries. The pin, not this number, is authoritative because the registry is changing quickly.

### Update Workflow

1. Fetch a candidate upstream commit deliberately.
2. Generate the component set in a temporary clean project.
3. Diff generated output against `@living-draft/ui` using recorded hashes.
4. Produce added, removed, source-changed, dependency-changed, and token-changed reports.
5. Review and approve component migrations.
6. Update fixtures and demos only where behavior or API changed.
7. Run typecheck and builds before replacing the lock.

Never track `latest` implicitly.

## Component Coverage Plan

Mirror every component documented at the pinned Base UI snapshot, excluding application blocks. This includes primitives, composites, data visualization, and newer agent/chat components.

Suggested implementation waves:

### Wave 1: Static Foundations

- Alert, Aspect Ratio, Avatar, Badge, Breadcrumb, Button, Button Group.
- Card, Empty, Input, Input Group, Item, Kbd, Label, Marker.
- Native Select, Pagination, Separator, Skeleton, Spinner, Table, Textarea, Typography.

### Wave 2: Stateful Form Controls

- Checkbox, Field, Input OTP, Progress, Radio Group, Slider, Switch.
- Toggle, Toggle Group.

### Wave 3: Disclosure And Navigation

- Accordion, Collapsible, Navigation Menu, Sidebar, Tabs.
- Scroll Area, Resizable, Direction.

### Wave 4: Overlays And Menus

- Alert Dialog, Context Menu, Dialog, Drawer, Dropdown Menu.
- Hover Card, Menubar, Popover, Select, Sheet, Toast, Tooltip.

### Wave 5: Complex Composites

- Calendar, Carousel, Chart, Combobox, Command.
- Data Table, Date Picker.

### Wave 6: Agent And Communication UI

- Attachment, Bubble, Message, Message Scroller, Questionnaire.
- Any additional entries present in the pinned index.

Each component receives catalog metadata with:

- Name, description, source path, upstream docs URL, and provenance.
- Public exports and composition anatomy.
- Variants, sizes, and defaults.
- Meaningful visual states: default, hover representation where useful, focus-visible, active, disabled, invalid, loading, open, selected, checked, empty, and responsive states as applicable.
- Static deterministic fixtures for agents, screenshots, and PDF/reference use.
- One hydrated behavior demo for pointer and keyboard exploration.
- Direct and transitive component/package dependencies.
- Canonical usage snippet.

For portals and overlays, catalog-only static fixture renderers reproduce canonical anatomy and classes in deterministic open states. They do not replace or modify the functional canonical components.

The required v1 gates are typecheck and build. Extra axe, keyboard, visual-regression, and PDF snapshot gates are intentionally deferred until the architecture stabilizes; upstream behavior remains intact in the mirror.

## Component Explorer

Living Draft Studio should generate its explorer from catalog metadata rather than hand-maintained routes.

Required capabilities:

- Search by component name, category, state, and dependency.
- Switch light/dark and named themes.
- Edit theme variables in memory.
- View variant/API matrix.
- View deterministic static states.
- Launch one hydrated demo.
- Show source path and upstream provenance.
- Show direct/transitive dependencies.
- Copy canonical import and usage snippets.
- Preview common mobile, compact, and desktop viewport widths.

## Artifact Dependency Analysis

Use the TypeScript compiler API or a focused parser to walk the artifact's static import graph recursively.

Classify:

- Direct shadcn components imported by artifact and artifact-local components.
- Transitive shadcn components imported by those components.
- Runtime npm packages.
- Living Draft utilities.
- Assets and font packages.

Write the analyzed graph into `artifact.json` during Save/Export. Fail validation for unresolved imports. Do not require agents to maintain dependency lists manually.

## Export Pipeline

1. Validate manifest and source.
2. Analyze and write dependencies.
3. Freeze the saved theme snapshot.
4. Generate a clean Astro runtime for the declared mode.
5. Build the static initial state.
6. Inline CSS, fonts, and small assets up to configured limits.
7. Copy larger images, fonts, and media with relative paths.
8. Strip Studio/runtime control APIs from output.
9. Open the built result in Chromium.
10. Wait for fonts and stable layout.
11. Print Letter PDF with backgrounds.
12. Copy source files, artifact-local components, assets, HTML, PDF, and final `artifact.json` into the export folder.

An interactive artifact's HTML export is intentionally a no-runtime initial-state snapshot. The canonical TSX retains interactivity for the target coding agent.

Example output:

```text
exports/customer-dashboard/
  source/
    artifact.tsx
    components/
    assets/
  artifact.json
  customer-dashboard.html
  customer-dashboard.pdf
  assets/                       # only assets not inlined into HTML
```

## Shared Custom Component Promotion

Custom Components begin in `artifacts/<slug>/components`. A deliberate command later promotes a selected Custom Component into `@living-draft/shared-components`, updates imports, adds registry/catalog metadata, and reports affected Artifacts. Do not prematurely centralize one-off Custom Components.

## Agent Skill

Create an installable Living Draft skill that stays concise and directs agents to current CLI help rather than duplicating volatile implementation instructions.

The skill should teach agents to:

- Run `living-draft open <artifact-path>` from an active session.
- Use real `@living-draft/ui` components and canonical shadcn composition.
- Load the official shadcn skill for component-specific guidance.
- Prefer tokens, variants, and theme-aware utilities before arbitrary Tailwind values.
- Keep artifact source framework-portable and free of Astro imports.
- Choose static or interactive mode intentionally.
- Read Studio feedback through the OpenCode adapter/hook.
- Run Save and Export.
- Read exported `artifact.json` and apply its theme values, fonts, dependencies, and artifact code to a target project.
- Avoid copying Living Draft's canonical component source into exports; install the listed shadcn components instead.

## Delivery Phases

### Phase 0: Architecture Spikes

- Verify a representative Base UI component set in generated Astro wrappers.
- Prove static mode emits no React client bundle.
- Prove interactive whole-root hydration.
- Prove external artifact HMR through generated `.living-draft/runtime`.
- Test Base UI portals in an iframe.
- Investigate OpenCode direct-session capabilities and define fallback hooks.
- Prove local Fontsource and downloaded Google Font PDF embedding.

Exit criterion: documented prototypes settle all high-risk boundaries.

### Phase 1: Workspace And Core

- Initialize pnpm monorepo and private package scope.
- Add schemas and `living-draft.json`.
- Implement artifact/config/theme read-write APIs.
- Implement generated Astro runtime.
- Implement process supervisor and `living-draft open/stop/doctor`.
- Add Base Nova neutral and `warm-yoga` presets.

Exit criterion: an external TSX artifact opens with HMR in a browser through Astro.

### Phase 2: Canonical UI Mirror

- Pin upstream commit and CLI.
- Install all documented Base UI components.
- Generate provenance lock and hashes.
- Complete Wave 1 and Wave 2 catalog coverage.
- Add sync/diff tooling.

Exit criterion: canonical package builds with no unexplained upstream diffs.

### Phase 3: Studio V1

- Build Studio shell and artifact iframe.
- Add component explorer, search, state matrices, and hydrated demos.
- Add responsive viewport controls.
- Add theme editor with Preview and Apply.
- Add artifact metadata Save.
- Display process/provider presence.

Exit criterion: a person can browse components, open an artifact, edit its theme, apply a baseline, and save its manifest.

### Phase 4: Complete Component Catalog

- Complete Waves 3 through 6.
- Add deterministic overlay fixtures.
- Ensure every pinned component has API matrix, state coverage, dependency metadata, and one live demo.

Exit criterion: 100% of the pinned documented component index meets the catalog contract.

### Phase 5: Export And Handoff

- Implement static import graph analysis.
- Implement source packaging and asset thresholds.
- Implement standalone static HTML.
- Implement Chromium PDF.
- Implement the agent skill's manifest unpacking workflow.
- Add `living-draft export` and export UI.

Exit criterion: an exported artifact can be handed to a coding agent, installed from its dependency inventory, themed from its manifest, and rendered consistently.

### Phase 6: Agent Session Loop

- Implement OpenCode provider adapter if direct API support is viable.
- Otherwise implement explicit OpenCode launch handshake plus queued-feedback hook.
- Add provider presence and chat to Studio.
- Keep provider interfaces suitable for Claude Code, Codex, and Copilot adapters.

Exit criterion: an agent launches Studio, the user sends feedback in Studio, and the same agent workflow receives and acts on it without manual transcript copying.

### Phase 7: Hardening

- Add accessibility and keyboard smoke gates.
- Add visual regression across themes and viewports.
- Add PDF snapshot checks.
- Add server security, recovery, crash, and stale-process tests.
- Add optional remote supervisor/runtime transport for future cloud execution.

## Deferred Decisions

These should not block v1:

- Direct visual element selection and annotation.
- Hosted/cloud Studio and remote artifact machines.
- MCP in addition to provider adapters.
- Radix or React Aria mirrors.
- Application blocks from the shadcn registry.
- Multiple entries or modes in one artifact.
- Tagged/strictly accessible PDF generation.
- Automatic git commits or revision snapshots.
- Browser-authored TSX editing.
- Shared per-project Custom Components beyond explicit promotion.

## First Implementation Task

Begin with Phase 0, not bulk component copying. Create three representative artifacts:

1. Static marketing layout using Button, Card, Badge, and Typography.
2. Static dashboard using Sidebar, Table, Chart, and a deterministic open Popover fixture.
3. Interactive form using Field, Select, Dialog, and Toast as one hydrated React root.

Use these to validate no-JS output, whole-root hydration, Base UI portals, iframe behavior, theme scoping, font loading, HTML export, and PDF before committing to the full 64-component mirror.
