<!--
SYNC IMPACT REPORT
==================
Version change: (none) → 1.0.0 (initial)
Modified principles: N/A (initial constitution)
Added sections:
  - Core Principles (3 principles: Content-First, Technical Accuracy, Simplicity)
  - Content Standards
  - Development Workflow
  - Governance
Templates requiring updates:
  - .specify/templates/plan-template.md: ⚠ pending (Constitution Check section needs alignment)
  - .specify/templates/spec-template.md: ✅ no changes needed (generic)
  - .specify/templates/tasks-template.md: ⚠ pending (references software-specific tasks)
  - .specify/templates/checklist-template.md: ✅ no changes needed (generic)
Follow-up TODOs:
  - TODO(CONSTITUTION_CHECK): Update plan-template.md Constitution Check section for blog context
  - TODO(TASKS_TEMPLATE): Simplify tasks-template.md for content-focused workflow
-->

# Zhiyi's Blog Constitution

## Core Principles

### I. Content-First
All content MUST prioritize reader value. Posts should focus on practical technical knowledge,
clear explanations, and reproducible results. Every post must have a clear purpose and
actionable takeaways. Avoid filler content; each post should solve a real problem or
document genuine learning.

**Rationale**: This is a technical blog for knowledge sharing. Content quality determines
reader trust and long-term value.

### II. Technical Accuracy
All technical claims MUST be verifiable. Code snippets, commands, and configurations
should be tested before publication. When documenting procedures, include environment
details (versions, platforms). Acknowledge limitations and edge cases. Cite sources
when referencing external work.

**Rationale**: Readers rely on accurate information for their own work. Inaccurate
technical content damages credibility and wastes readers' time.

### III. Simplicity (YAGNI)
Start with the simplest viable solution. Use Jekyll's built-in features before adding
custom plugins. Prefer Markdown over complex HTML. Add new features only when a clear
need emerges. Avoid over-engineering the blog structure.

**Rationale**: This is a personal blog, not a complex web application. Simplicity
reduces maintenance burden and keeps focus on content creation.

## Content Standards

**Formatting**:
- Posts MUST use front matter with title, date, and layout fields
- Code blocks MUST specify language for syntax highlighting
- Images MUST include alt text for accessibility
- Long posts SHOULD use section headings for readability

**Technical Posts**:
- Include environment/version information
- Provide complete, copy-pasteable commands where applicable
- Document both success and failure scenarios when relevant
- Link to official documentation for referenced tools

**Review Before Publishing**:
- Verify all links are functional
- Test code snippets in a clean environment
- Check for broken formatting on mobile view
- Ensure consistent terminology throughout

## Development Workflow

**Branch Strategy**:
- Create feature branches for new posts: `post/[YYYY-MM-DD]-[slug]`
- Use descriptive commit messages: `Add: [post title]` or `Fix: [specific issue]`
- Preview locally with `jekyll serve` before committing

**Post Creation Process**:
1. Create draft in `_posts/` with proper naming: `YYYY-MM-DD-title.md`
2. Write content following Content Standards
3. Test locally: verify formatting, links, code blocks
4. Commit and push
5. Deploy to GitHub Pages

**Maintenance**:
- Periodically review older posts for accuracy
- Update deprecated commands or tools when discovered
- Mark outdated posts with a note if significant changes occurred

## Governance

**Amendment Process**:
- Constitution changes require clear rationale documented in commit messages
- Changes SHOULD be reflected in the Sync Impact Report comment at file top
- Version follows semantic versioning: MAJOR.MINOR.PATCH
  - MAJOR: Principle additions/removals or fundamental redefinitions
  - MINOR: New sections or material expansions
  - PATCH: Clarifications, typo fixes, non-semantic refinements

**Compliance**:
- All new posts MUST adhere to Core Principles
- Before publishing, verify against Content Standards checklist
- Technical accuracy is the author's responsibility

**Version**: 1.0.0 | **Ratified**: 2026-03-19 | **Last Amended**: 2026-03-19
