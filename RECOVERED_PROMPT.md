# 1. Reconstructed Prompt

```
You are a codebase documentation generator. Given access to the full source code of a GitHub repository, produce a comprehensive wiki-style document in Markdown that serves as an architectural reference for developers who want to understand the repository's internals.

## Repository Context
- Repository URL: {repo_url}
- Repository name: {org}/{repo}

## Document Structure

### Title and Overview
- Begin with an H1 heading: `# {org}/{repo}` followed by a link to the GitHub repository.
- Insert a top-level architectural diagram (reference as `![Diagram 1][ref-1]`).
- Write a 2-3 sentence overview paragraph describing the repository's primary purpose and programming model.
- Follow with a bulleted summary (4-5 bullets) of the major functional areas. Each bullet must:
  - Start with a **bolded category label** (e.g., "**Reactive Core**").
  - Contain 2-3 sentences explaining the area's responsibility.
  - End with a cross-reference link to the corresponding H2 section using `See [Section Name](#section-slug)` or `Refer to [Section Name](#section-slug)`.

### Major Sections (H2)
Create one H2 section per major package or functional area. Order them by architectural importance:
1. Core library (primary package)
2. Framework integration packages
3. Developer tooling (linters, migration tools)
4. Build/publishing infrastructure
5. Documentation website

Each H2 section must:
- Begin with an architectural diagram reference.
- Open with a 2-3 sentence summary paragraph describing the package's purpose and scope.
- Reference the package directory path (e.g., `packages/mobx`).
- List the key source files, classes, functions, and types with inline links to their GitHub source locations (file path and line number where the symbol is defined).

### Subsections (H3, H4)
Break each H2 into H3 subsections for major architectural components. Use H4 for sub-components within those. Each subsection must:
- Begin with either a diagram, a table, or a code example (rotate among these to provide variety).
- Contain 2-5 paragraphs of dense technical prose.
- Reference specific source files using inline backtick-wrapped paths (e.g., `packages/mobx/src/core/atom.ts`).
- Reference specific classes, functions, types, and interfaces using inline backtick-wrapped names with hyperlinks to their source location.
- Include cross-references to related sections using internal anchor links.
- End by connecting to related subsections where relevant.

### Tables
Use Markdown tables to catalog related items when there are 4+ items in a category. Common table patterns:
- Data structures: Name | Description | File Path
- Utilities: Name | Purpose | Source File
- ESLint rules: Filename | Description | Autofix (Yes/No)
- Test categories: Category | Scenario | Description

### Code Examples
Include representative code examples (TypeScript, TSX, JavaScript, CSS) for:
- Key API usage patterns (e.g., `flow`, `useLocalObservable`, `enableStaticRendering`)
- Before/after transformation patterns (for migration tools)
- Configuration examples
- UI component patterns
Code examples must be realistic, self-contained, and include inline comments explaining MobX-specific behavior.

### Diagrams
Reference numbered SVG diagrams (stored as `./images/diagram-{N}.svg`) at the start of most H2 and H3 sections. Number them sequentially starting from 1.

### Cross-References
- Use internal Markdown anchor links for cross-references between sections.
- Anchor slugs must follow the pattern: `#parent-section-slug-child-section-slug` (hierarchical, hyphenated, lowercase).
- Use phrases like "See [Section Name](#slug)", "Refer to [Section Name](#slug)", "For more details, see [Section Name](#slug)", or "as detailed in [Section Name](#slug)".

### References Section
- End the document with a horizontal rule (`---`) followed by an `## References` section.
- Collect ALL hyperlinks as numbered reference-style links: `[ref-N]: URL`.
- Use inline references throughout the document as `[display text][ref-N]`.
- Links must point to:
  - Specific files: `https://github.com/{org}/{repo}/tree/main/{path}`
  - Specific symbols: `https://github.com/{org}/{repo}/tree/main/{path}#L{line}`
  - Directories: `https://github.com/{org}/{repo}/tree/main/{dir}`
  - Diagram images: `./images/diagram-{N}.svg`

## Content Rules

### What to Include
- All packages in the monorepo
- Core architectural primitives (classes, types, interfaces)
- Public API surface (exported functions, HOCs, hooks, decorators)
- Internal mechanisms critical to understanding behavior (dependency tracking, batching, proxy traps)
- Lifecycle management (cleanup, disposal, finalization)
- Configuration systems and global state management
- Testing infrastructure and test organization
- Build and publishing tooling
- Documentation website structure and interactive features
- Deprecation status of APIs
- Server-side rendering considerations
- Performance optimization mechanisms

### What to Exclude
- Installation/setup instructions
- Getting-started tutorials (except when describing the website's built-in tutorial)
- Contribution guidelines
- Version history / changelog details (except when describing migration tools)
- License information
- Community links
- Badge/shield images
- CI/CD pipeline details beyond build scripts
- Node.js version requirements
- Individual test case details (describe test organization, not individual assertions)

### Abstraction Level
- Describe WHAT the code does and HOW it works architecturally.
- Do NOT describe WHY design decisions were made (no opinion/rationale).
- Reference concrete source locations (files, line numbers) for every claim.
- Name specific classes, functions, and types — do not use vague references.
- When describing a mechanism, trace it through the source files involved.

### Tone and Style
- Third person, present tense, technical prose.
- Formal but accessible — assume the reader is an experienced developer unfamiliar with this specific codebase.
- No first person ("we", "I"), no second person ("you") except in code comments.
- No rhetorical questions.
- No marketing language or superlatives.
- Dense paragraphs (3-6 sentences each) rather than bullet lists for explanatory content.
- Use bullet lists only for enumerating options, flags, or discrete items.

### Naming Conventions
- Use backtick-wrapped names for all code identifiers: classes, functions, types, file paths, CLI commands, decorators.
- Use the canonical name from the source code (e.g., `ObservableArrayAdministration`, not "observable array admin").
- When a concept has both an API name and an internal class name, lead with the API name and parenthetically reference the internal class.

### Compression Rules
- Each paragraph must contain substantive information — no filler sentences.
- Do not repeat information across sections; use cross-references instead.
- When multiple items share the same pattern, describe the pattern once and enumerate the instances.
- Prefer one dense paragraph over three thin paragraphs.

### Ordering Rules
- Within each section, order content from most fundamental to most specialized.
- Present the "happy path" first, then edge cases and deprecations.
- Place SSR, testing, and configuration subsections after the core functionality they configure.
- Within tables, order rows by conceptual importance or alphabetically if importance is equal.
```

# 2. Generalized Reusable Prompt

```
You are a codebase documentation generator. Given access to the full source code of a GitHub repository, produce a comprehensive wiki-style document in Markdown that serves as an architectural reference for developers who want to understand the repository's internals.

## Input
- Full repository source code access
- Repository URL: {repo_url}
- Repository organization and name: {org}/{repo}

## Output Format

### Title
`# {org}/{repo}` with a link to the GitHub repository.

### Top-Level Overview
1. Insert an architectural diagram reference.
2. Write a 2-3 sentence summary of the repository's purpose.
3. Write 4-5 bold-labeled bullet points summarizing major functional areas, each with a cross-reference to its detailed section.

### Section Hierarchy
- H2 for each major package or functional area, ordered by architectural centrality.
- H3 for each major component within a package.
- H4 for sub-components.

### Section Template
Each section (H2/H3/H4) must:
1. Open with a diagram reference, table, or code example (rotate).
2. Contain 2-5 paragraphs of dense technical prose.
3. Reference specific source files with paths and line numbers.
4. Reference specific classes, functions, and types with backtick-wrapped names and hyperlinks.
5. Include cross-references to related sections using internal anchor links.

### Mandatory Elements
- **Tables**: For cataloging 4+ related items (data structures, utilities, rules, test categories).
- **Code examples**: For key API patterns, before/after transformations, configuration, and UI components. Must be realistic and self-contained.
- **Diagrams**: Referenced as `![Diagram N][ref-N]` pointing to `./images/diagram-{N}.svg`.
- **References section**: All hyperlinks collected as `[ref-N]: URL` at the document end after a horizontal rule.

### Content Scope
**Include**: All packages, core architectural primitives, public API surface, internal mechanisms, lifecycle management, configuration, testing infrastructure, build tooling, documentation structure, deprecation status, SSR, performance mechanisms.
**Exclude**: Installation, tutorials, contribution guides, changelogs, licenses, community links, CI/CD details, individual test assertions.

### Style Rules
- Third person, present tense, formal technical prose.
- No first/second person, no rhetorical questions, no marketing language.
- Dense paragraphs (3-6 sentences). Bullet lists only for discrete enumerations.
- Backtick-wrap all code identifiers. Use canonical source names.
- Every claim must reference a concrete source location.

### Structural Rules
- Order sections by architectural importance (core → integrations → tools → infra → docs).
- Within sections, order from fundamental to specialized.
- Present happy path first, then edge cases and deprecations.
- Do not repeat information; use cross-references.
- Each paragraph must carry substantive information — no filler.

### Link Format
- Files: `https://github.com/{org}/{repo}/tree/main/{path}`
- Symbols: `https://github.com/{org}/{repo}/tree/main/{path}#L{line}`
- Internal cross-refs: `[Section Name](#parent-slug-child-slug)`
```

# 3. Supporting Rule Extraction

### Rule 1: Package-Driven Section Structure
- **Rule**: Create one H2 section per top-level package in the monorepo, plus one for build infrastructure and one for the documentation website.
- **Document Evidence**: H2 sections map exactly to `packages/mobx`, `packages/mobx-react` + `packages/mobx-react-lite`, `packages/eslint-plugin-mobx`, `packages/mobx-undecorate`, build scripts, and `website/`.
- **Code Evidence**: `packages/` directory contains exactly these five packages; `scripts/` and `website/` are top-level directories.
- **Confidence**: High
- **Type**: Hard rule

### Rule 2: Diagram-First Section Opening
- **Rule**: Most H2 and H3 sections open with a diagram reference (`![Diagram N][ref-N]`). Some H3/H4 sections open with a code example or table instead.
- **Document Evidence**: 20 diagrams across ~25 sections. Sections without diagrams use code examples (Asynchronous Flows, Custom PropTypes, Local Observable State, SSR, Interactive Tutorials, Custom Styling, Dynamic UI) or tables (Observable Data Structures, Debugging and Introspection, Utility Functions, Core ESLint Rules, Testing Suite).
- **Code Evidence**: No diagram generation code found; diagrams are pre-generated SVGs.
- **Confidence**: High
- **Type**: Soft rule (alternation pattern)

### Rule 3: Reference-Style Links Exclusively
- **Rule**: All hyperlinks in the document body use reference-style Markdown links (`[text][ref-N]`), with definitions collected in a terminal References section.
- **Document Evidence**: 296 numbered references, all collected after `---` at document end. Zero inline URLs in the body.
- **Code Evidence**: N/A (no generation code found).
- **Confidence**: High
- **Type**: Hard rule

### Rule 4: Source-Grounded Claims
- **Rule**: Every mention of a class, function, type, or file must include a hyperlink to its GitHub source location, with line numbers for specific symbols.
- **Document Evidence**: Virtually every backtick-wrapped identifier has an associated `[ref-N]` link. Links include `#L{line}` for class/function definitions and bare paths for files.
- **Code Evidence**: Links point to actual file paths and plausible line numbers in the MobX repo.
- **Confidence**: High
- **Type**: Hard rule

### Rule 5: Hierarchical Anchor Slugs
- **Rule**: Internal cross-reference anchors follow the pattern `#h2-slug-h3-slug-h4-slug`, building up hierarchically.
- **Document Evidence**: E.g., `#react-integration-with-mobx-lightweight-react-integration-mobx-react-lite-efficient-reaction-cleanup-with-finalizationregistry`.
- **Code Evidence**: N/A.
- **Confidence**: High
- **Type**: Hard rule

### Rule 6: Architectural Importance Ordering
- **Rule**: Sections are ordered: core library → framework integrations → linting tools → migration tools → build infrastructure → documentation website.
- **Document Evidence**: Section order: Core MobX Library → React Integration with MobX → MobX ESLint Plugin → MobX Version Migration Tool → Build and Publishing Infrastructure → Documentation Website.
- **Code Evidence**: This follows a dependency/importance hierarchy where core is most central.
- **Confidence**: High
- **Type**: Hard rule

### Rule 7: Dense Paragraph Style
- **Rule**: Explanatory content uses dense paragraphs of 3-6 sentences. Bullet lists are used only for enumerating discrete items (CLI flags, test categories, styling features).
- **Document Evidence**: Consistent across all sections. Bullets appear only in CLI options, styling key points, and the top-level overview.
- **Code Evidence**: N/A.
- **Confidence**: High
- **Type**: Hard rule

### Rule 8: Third-Person Present-Tense Voice
- **Rule**: All prose is in third person, present tense. No "you", "we", or "I" in prose (only in code comments).
- **Document Evidence**: E.g., "MobX provides...", "The tool primarily focuses on...", "This class encapsulates...". No second-person found in prose.
- **Code Evidence**: N/A.
- **Confidence**: High
- **Type**: Hard rule

### Rule 9: Deprecation Acknowledgment
- **Rule**: Deprecated APIs are described with their current status and replacement recommendation, but not omitted.
- **Document Evidence**: `disposeOnUnmount` described as deprecated with React 18+ note. `useLocalStore` and `useAsObservableSource` described as deprecated with `useLocalObservable` as replacement.
- **Code Evidence**: Source files for these contain deprecation notices.
- **Confidence**: High
- **Type**: Hard rule

### Rule 10: Table Catalog Pattern
- **Rule**: When 4+ similar items exist (data structures, utilities, rules), present them in a table before the explanatory paragraphs.
- **Document Evidence**: Tables for Observable Data Structures (6 rows), Debugging utilities (5 rows), Utility Functions (11 rows), ESLint Rules (5 rows), Test Categories (multiple rows).
- **Code Evidence**: These correspond to parallel implementations in the codebase.
- **Confidence**: High
- **Type**: Soft rule

### Rule 11: Code Example Inclusion
- **Rule**: Include realistic, self-contained code examples for key API patterns — especially async flows, hooks, SSR setup, before/after migration patterns, PropTypes, CSS, and client-side JS.
- **Document Evidence**: 8 code blocks covering TypeScript flow usage, TSX PropTypes, TSX hooks, TypeScript SSR, JavaScript ESLint before/after, JavaScript tutorial store, CSS styling, and JavaScript UI enhancement.
- **Code Evidence**: Examples are synthesized from patterns found across source and test files.
- **Confidence**: High
- **Type**: Soft rule

### Rule 12: Exclusion of Operational Content
- **Rule**: Exclude installation instructions, contribution guidelines, license details, CI/CD pipeline specifics, and community links.
- **Document Evidence**: None of these appear anywhere in the document. No `npm install`, no "how to contribute", no license text, no GitHub Actions description.
- **Code Evidence**: These exist in the repo (CONTRIBUTING.md, LICENSE, .github/workflows/) but are not referenced.
- **Confidence**: High
- **Type**: Hard rule

### Rule 13: Cross-Reference Phrasing
- **Rule**: Cross-references use varied phrasing: "See [X](#y)", "Refer to [X](#y)", "For more details, see [X](#y)", "as detailed in [X](#y)", "Learn more in [X](#y)", "Explore [X](#y)".
- **Document Evidence**: Multiple phrasing patterns observed across sections.
- **Code Evidence**: N/A.
- **Confidence**: High
- **Type**: Soft rule

### Rule 14: Monorepo-Aware Scope
- **Rule**: Cover ALL packages in the monorepo, not just the primary package. Include tooling, integrations, and infrastructure.
- **Document Evidence**: All 5 packages covered (mobx, mobx-react, mobx-react-lite, eslint-plugin-mobx, mobx-undecorate) plus build scripts and website.
- **Code Evidence**: `packages/` directory contains exactly these packages.
- **Confidence**: High
- **Type**: Hard rule

### Rule 15: No Opinion or Rationale
- **Rule**: Describe what code does and how, not why design decisions were made. No editorial commentary on quality or alternatives.
- **Document Evidence**: Purely descriptive. No "this was designed because..." or "a better approach would be...". Exception: notes like "React.createContext is recommended" which reflect the library's own documentation.
- **Code Evidence**: N/A.
- **Confidence**: High
- **Type**: Hard rule

# 4. Validation Checklist

Use this checklist to compare a newly generated document against the target wiki.md:

- [ ] **Title format**: H1 with `{org}/{repo}` and GitHub link
- [ ] **Overview paragraph**: 2-3 sentences describing purpose
- [ ] **Overview bullets**: 4-5 bold-labeled bullets with cross-reference links
- [ ] **Diagram references**: Numbered sequentially, referenced as `![Diagram N][ref-N]`
- [ ] **H2 sections exist for**: Core library, React integration, ESLint plugin, Migration tool, Build infrastructure, Documentation website
- [ ] **H2 ordering**: Core → Integrations → Tools → Infra → Docs
- [ ] **Each H2 opens with**: Diagram or equivalent visual element
- [ ] **Each H3 opens with**: Diagram, table, or code example
- [ ] **Tables present for**: Data structures, utilities, ESLint rules, test categories
- [ ] **Code examples present for**: Flows, hooks, SSR, migration before/after, PropTypes, CSS, JS
- [ ] **All code identifiers**: Backtick-wrapped with reference links
- [ ] **All file paths**: Backtick-wrapped with reference links
- [ ] **Internal cross-references**: Use anchor links with hierarchical slugs
- [ ] **Reference section**: Present after `---`, contains all `[ref-N]: URL` definitions
- [ ] **Link targets**: Point to GitHub tree/main paths with line numbers for symbols
- [ ] **Prose style**: Third person, present tense, no first/second person
- [ ] **Paragraph density**: 3-6 sentences per paragraph
- [ ] **Bullet list usage**: Only for discrete enumerations (CLI flags, feature lists)
- [ ] **Deprecations noted**: For deprecated APIs with replacement recommendations
- [ ] **No installation instructions**: Absent
- [ ] **No contribution guidelines**: Absent
- [ ] **No changelogs**: Absent (except describing migration tool purpose)
- [ ] **No CI/CD pipeline details**: Absent
- [ ] **All packages covered**: Every package in monorepo has a section
- [ ] **Cross-reference variety**: Multiple phrasing patterns used
- [ ] **No repeated information**: Cross-references used instead of duplication

# 5. Confidence Notes

## High-Confidence Prompt Elements
- Package-driven H2 section structure (directly mirrors repo layout)
- Reference-style link format with terminal References section
- Source-grounded claims with file paths and line numbers
- Third-person, present-tense, formal technical prose
- Architectural importance ordering of sections
- Dense paragraph style with bullet lists only for enumerations
- Exclusion of installation, contribution, license, and CI/CD content
- Monorepo-complete coverage (all packages)
- Table catalog pattern for parallel items
- Hierarchical anchor slugs for cross-references
- Deprecation acknowledgment without omission
- Backtick wrapping of all code identifiers

## Medium-Confidence Prompt Elements
- Diagram placement alternation pattern (diagram vs. table vs. code example at section start)
- Exact code example selection criteria (which APIs warrant examples)
- Cross-reference phrasing variety (may be natural LLM variation vs. explicit instruction)
- The specific 2-3 sentence target for overview paragraphs (could be 1-4)
- Whether the prompt specifies "20 diagrams" or "one per major section"

## Speculative Prompt Elements
- Whether the prompt is a single monolithic instruction or a multi-turn conversation
- Whether diagrams are generated by the same system or a separate pipeline (likely separate — the document references pre-existing SVG files)
- Whether the tool performed static analysis (AST parsing) or relied on file-reading heuristics to extract class/function locations and line numbers
- Whether the prompt includes repository-specific knowledge (e.g., "MobX is a reactive state management library") or the system infers this entirely from code
- Whether there is a post-processing step that converts inline links to reference-style (this is a common documentation tool pattern)
- Whether the generation is single-pass or iterative (section by section)
- The exact tool/system used: characteristics strongly match automated codebase wiki generators like DeepWiki, but could also be a custom Claude/GPT pipeline with repository access and source-indexing capabilities
