---

description: "Task list template for content creation"
---

# Tasks: [POST TITLE]

**Input**: Research notes, code snippets, screenshots
**Prerequisites**: plan.md (optional), spec.md (optional for complex posts)

**Organization**: Tasks are organized by content creation workflow

## Format: `[ID] [P?] [Phase] Description`

- **[P]**: Can run in parallel (independent tasks)
- **[Phase]**: Which phase this task belongs to (Research, Draft, Review, Publish)

<!--
  ============================================================================
  IMPORTANT: Adapt tasks based on post complexity. Simple posts may need
  fewer tasks; complex tutorials may need more detailed breakdown.
  ============================================================================
-->

## Phase 1: Research & Preparation

**Purpose**: Gather information, test commands, capture screenshots

- [ ] T001 Define post objective and key takeaways
- [ ] T002 [P] Test all commands/code in clean environment
- [ ] T003 [P] Capture screenshots (if needed)
- [ ] T004 Gather reference links and citations
- [ ] T005 Note environment details (versions, platforms)

---

## Phase 2: Draft Content

**Purpose**: Write the initial post content

- [ ] T006 Create file: `_posts/YYYY-MM-DD-post-title.md`
- [ ] T007 Write front matter (title, date, layout)
- [ ] T008 Draft introduction with clear problem statement
- [ ] T009 Write main content sections with headings
- [ ] T010 Add code blocks with proper syntax highlighting
- [ ] T011 Insert images with alt text
- [ ] T012 Add conclusion and key takeaways
- [ ] T013 Include reference links

---

## Phase 3: Review & Refine

**Purpose**: Verify accuracy and quality

- [ ] T014 Preview locally: `jekyll serve`
- [ ] T015 Verify all links are functional
- [ ] T016 Re-test all code snippets
- [ ] T017 Check mobile formatting
- [ ] T018 Proofread for clarity and typos
- [ ] T019 Verify against Constitution principles

---

## Phase 4: Publish

**Purpose**: Commit and deploy

- [ ] T020 Commit with message: `Add: [post title]`
- [ ] T021 Push to repository
- [ ] T022 Verify GitHub Pages deployment
- [ ] T023 Share (optional): social media, communities

---

## Dependencies & Execution Order

### Phase Dependencies

- **Research (Phase 1)**: No dependencies - can start immediately
- **Draft (Phase 2)**: Depends on Research completion
- **Review (Phase 3)**: Depends on Draft completion
- **Publish (Phase 4)**: Depends on Review completion

### Parallel Opportunities

- Testing commands and capturing screenshots can run in parallel
- Gathering references can happen alongside testing

---

## Notes

- [P] tasks = independent, can run in parallel
- Test code in a clean environment to ensure reproducibility
- Verify against Constitution before publishing
- Commit after completing each phase
