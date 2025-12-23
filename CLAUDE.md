# CLAUDE.md

This file provides guidance to Claude Code when working with this repository.

## Repository Overview

**Exploring Kubernetes** is a "digital garden" of knowledge, designed to be the "Control Room" wing of the System Mage RPG universe. It serves as a comprehensive guide to Kubernetes primitives and patterns.

## Important Preferences

**Git Operations**: The user handles all git operations (commits, pushes, etc.) themselves. Do not commit or push changes.

**MkDocs Operations**: The user handles running `mkdocs serve` and `mkdocs build` themselves. Do not run these commands.

## Project Structure

- `docs/` - Markdown content organized by category (matches site navigation)
  - `core_concepts/` - Architecture, Pods, Nodes.
  - `workloads/` - Deployments, StatefulSets, DaemonSets.
  - `networking/` - Services, Ingress, NetworkPolicies.
  - `storage/` - PV, PVC, StorageClass.
  - `configuration/` - ConfigMaps, Secrets.
  - `observability/` - Logging, Monitoring, Probes.
- `mkdocs.yaml` - Site configuration and navigation
- `pyproject.toml` - Poetry dependencies

## Common Commands

```bash
# Install dependencies
poetry install

# Serve locally (http://localhost:8000)
poetry run mkdocs serve

# Build static site (ALWAYS use --strict for link validation)
poetry run mkdocs build --strict
```

**Link Validation:** The project uses `mkdocs-htmlproofer-plugin` to validate all internal links. Always build with `--strict` flag to catch broken links.

## Content Guidelines

### Tone and Style

Articles must balance **playfulness with professionalism** and be **technically accurate** while remaining **accessible**.

**Core Principles:**

- **The Analogy**: Use the "Control Room" / "Factory" analogy where appropriate to ground complex concepts.
- **Strong openings**: Ground in real-world problems (e.g., "The Shipping Container").
- **Professional yet engaging**: Use wit in parentheticals and asides, not emoji spam (limit to 1-3 per article, used strategically).
- **Technical rigor**: Include clean YAML manifests and precise terminology.
- **Structured learning**: Build from simple to complex examples; use clear section headers.
- **Direct voice**: Address reader as "you"; be confident but not arrogant.

**Required Sections:**

1. Opening paragraph(s) - hook with real-world relevance/analogy
2. Concept explanation - explain the K8s primitive
3. YAML Manifests - provide clean, annotated code
4. Result/Application - explain what happens when applied
5. Practice/Exercise (if applicable) - use `??? question` and `??? tip`
6. "Key Takeaways" table
7. Closing reflection

### Content Structure

- Articles should be teaching-focused, not just notes.
- Use mermaid diagrams for visual concepts. **Follow the project's 'Slate' color scheme for consistency:**
  - **Standard Node (Slate 800):** `fill:#2d3748,stroke:#cbd5e0,stroke-width:2px,color:#fff`
  - **Highlighted Node (Slate 700):** `fill:#4a5568,stroke:#cbd5e0,stroke-width:2px,color:#fff`
  - **Darker Node (Slate 900):** `fill:#1a202c,stroke:#cbd5e0,stroke-width:2px,color:#fff`
  - **Accent Node (Green):** `fill:#48bb78,stroke:#cbd5e0,stroke-width:2px,color:#fff`
  - *Always explicitly style nodes using these hex codes to ensure readability in the dark theme.*
- Use admonitions for tips and callouts:
  - Prefer `??? tip` (collapsible tips) for helpful insights.
  - **Avoid** `??? note` or `!!! note` (the "note" style doesn't render well in Material for MkDocs).
- **Code examples must include titles, line numbers, and annotations**:
  - Format: ` ```yaml title="pod-definition.yaml" linenums="1" `
  - Use `# (1)!`, `# (2)!`, etc. for inline annotations.
  - After the code block, provide numbered explanations.

### Interactive Elements: Tabs and Cards

Use tabs and cards to organize content interactively.

**When to Use Cards:**
Use `<div class="grid cards">` for **brief, scannable content** (e.g., comparing K8s object types).

**When to Use Tabs:**
Use `=== "Tab Name"` syntax for **detailed, related content** (e.g., comparing different deployment strategies or providing multi-language client examples).

---