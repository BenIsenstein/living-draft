# Use React Artifacts with a generated Astro runtime

Living Draft uses portable React TSX as the canonical Artifact source and generates Astro only as its execution shell. This preserves direct transfer into React applications while allowing Astro to render a Static Artifact without shipping React client code or hydrate an Interactive Artifact as one island; Astro-specific templates and Web Components would introduce a second source API and require translation.
