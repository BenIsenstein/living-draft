# Living Draft

Living Draft is a design-to-code environment where agents and people shape executable interface designs built from production-compatible React components.

## Language

**Artifact**:
An enduring authored interface design with one portable React composition root. It may represent a page, screen, document, or bounded flow, but not a routed application.
_Avoid_: Mockup, prototype, export, project

**Export**:
A disposable handoff deliverable derived from an Artifact at a point in time. It can contain source, manifest, static HTML, assets, and PDF without becoming a new Artifact.
_Avoid_: Artifact, release

**Artifact Theme**:
The complete theme snapshot saved with an Artifact and restored when that Artifact is reopened.
_Avoid_: Project theme, current theme, global theme

**Baseline Theme**:
The theme used to initialize a new Artifact. Changing it does not alter an existing Artifact Theme.
_Avoid_: Artifact Theme, project theme

**Component**:
A canonical shadcn/ui component mirrored by Living Draft with near-zero upstream divergence.
_Avoid_: Primitive, widget, block

**Custom Component**:
An assembly owned by one Artifact and built from Components, HTML, or other Custom Components.
_Avoid_: Composition, block

**Shared Custom Component**:
A Custom Component deliberately promoted for reuse by multiple Artifacts.
_Avoid_: Block, global component, composition

**Artifact Runtime**:
The generated Astro execution shell that renders one Artifact for development or output. It remains the Artifact Runtime even when no preview process is running.
_Avoid_: Artifact, Studio, session

**Living Draft Session**:
The supervised association among one Artifact, its running processes, connected Studio clients, and any Agent Connection. It remains active until explicitly stopped or expired by the supervisor.
_Avoid_: Browser tab, agent session, preview server

**Agent Connection**:
A replaceable attachment between a Living Draft Session and one external coding-agent session.
_Avoid_: Living Draft Session, provider

**Save Artifact Metadata**:
The action that validates and persists an Artifact's metadata, including its Artifact Theme. It does not checkpoint source code or create a revision.
_Avoid_: Save source, commit, checkpoint

**Static Artifact**:
An Artifact rendered without client hydration or a shipped React component runtime. Native browser behavior such as links and forms remains valid.
_Avoid_: Inert artifact, non-interactive artifact

**Interactive Artifact**:
An Artifact hydrated as one React root to demonstrate component interaction and local design state. It does not imply finished data access, mutations, authentication, or application integration.
_Avoid_: Application, production-ready app

**State Fixture**:
A catalog-only deterministic visual representation of one meaningful Component state, including states that normally require interaction or portals.
_Avoid_: Static Artifact, mockup

**Component Explorer**:
The searchable Studio view of Component APIs, State Fixtures, dependencies, provenance, usage, and hydrated demonstrations.
_Avoid_: Design library, component package
