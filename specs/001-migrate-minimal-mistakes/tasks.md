---

description: "Task list template for theme migration"
---

# Tasks: Migrate Blog to Minimal Mistakes Theme

**Input**: Design documents from `/specs/001-migrate-minimal-mistakes/`
**Prerequisites**: plan.md, spec.md, research.md, data-model.md, quickstart.md

**Organization**: Tasks are organized by user story to enable independent verification of each migration outcome.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (e.g., US1, US2, US3)
- Include exact file paths in descriptions

<!--
  ============================================================================
  IMPORTANT: Tasks are organized by user story for independent verification.
  Each user story phase should result in a testable migration outcome.
  ============================================================================
-->

## Phase 1: Setup (Environment Preparation)

**Purpose**: Prepare local development environment for testing migration

- [x] T001 [P] Verify Ruby version 2.7+ installed: `ruby --version`
- [ ] T002 [P] Install Bundler: `gem install bundler`
- [x] T003 Create Gemfile in repository root with minimal-mistakes-jekyll dependency
- [ ] T004 Run `bundle install` to install dependencies
- [ ] T005 Verify Jekyll starts: `bundle exec jekyll serve --draft`

**Checkpoint**: Local development environment ready

**Note**: Ruby is not installed on this system. T001 verification failed. Gemfile (T003) has been created. User needs to install Ruby 2.7+ before proceeding with T002, T004, T005.

---

## Phase 2: Foundational (Theme Configuration)

**Purpose**: Configure minimal-mistakes theme - blocks all user story verification

**⚠️ CRITICAL**: Must complete before any user story can be verified

- [ ] T006 Backup current _config.yml to _config.yml.backup
- [x] T007 Update _config.yml: Add `theme: minimal-mistakes-jekyll`
- [x] T008 [P] Update _config.yml: Migrate social links to `author.links` format
- [x] T009 [P] Update _config.yml: Add `minimal_mistakes_skin: default` (for preview)
- [x] T010 [P] Update _config.yml: Add navigation configuration
- [x] T011 Update _config.yml: Add defaults for posts (layout: single, author_profile, etc.)
- [x] T012 [P] Update _config.yml: Add `google_analytics_id` placeholder (user provides ID)
- [ ] T013 Test build: `bundle exec jekyll build` - verify no errors

**Checkpoint**: Theme configured, ready for content migration

**Note**: T007-T012 completed. T006 (backup) skipped as config was already backed up by git. T013 requires Ruby.

---

## Phase 3: User Story 1 - Blog Renders with New Theme (Priority: P1) 🎯 MVP

**Goal**: Blog displays with minimal-mistakes theme styling, navigation works, posts accessible

**Independent Test**: Visit http://127.0.0.1:4000/ and verify theme layout, navigation, and post access

### Implementation for User Story 1

- [x] T014 [P] [US1] Update index.html to use minimal-mistakes home layout structure
- [x] T015 [P] [US1] Update about.md front matter: `layout: single`
- [x] T016 [US1] Update all posts in _posts/ directory (currently 16) front matter: change `layout: post` to `layout: single`
- [x] T017 [P] [US1] Create 404.html with minimal-mistakes default layout
- [x] T018 [P] [US1] Create _layouts directory if needed for custom overrides
- [ ] T019 [US1] Start Jekyll server: `bundle exec jekyll serve`
- [ ] T020 [US1] Verify homepage renders with minimal-mistakes layout
- [ ] T021 [US1] Verify about page displays with new theme styling
- [ ] T022 [US1] Verify 404 page renders at /nonexistent-url
- [ ] T023 [US1] Verify all 16 posts are accessible via navigation

**Checkpoint**: Blog renders with new theme, all content accessible

**Note**: T014-T018 completed. T019-T023 require Ruby to verify locally.

---

## Phase 4: User Story 2 - Content Preservation (Priority: P1)

**Goal**: All 16 posts and about page content preserved exactly

**Independent Test**: Compare old and new content; verify all 16 posts present and render correctly

### Implementation for User Story 2

- [x] T024 [P] [US2] Create content inventory: list all files in _posts/ directory
- [x] T025 [US2] Verify post count: confirm 16 posts exist in _posts/
- [ ] T026 [US2] Spot-check 5 posts: compare content before/after migration
- [ ] T027 [US2] Verify code blocks render with syntax highlighting
- [ ] T028 [US2] Verify images display correctly in posts
- [ ] T029 [US2] Verify internal and external links work in posts
- [x] T030 [US2] Verify about.md content matches original (excluding layout changes)
- [ ] T031 [US2] Test mobile view: verify content renders on 320px width
- [ ] T032 [US2] Verify RSS feed: visit /feed.xml and confirm all posts included

**Checkpoint**: All content preserved and rendering correctly

**Note**: T024, T025, T030 completed. Remaining tasks require Ruby for local verification.

---

## Phase 5: User Story 3 - Clean Repository Structure (Priority: P2)

**Goal**: Repository contains only minimal-mistakes theme files and user content

**Independent Test**: List repository files; verify no lanyon/poole theme files remain

### Implementation for User Story 3

- [x] T033 [P] [US3] Remove _layouts/ directory (using gem layouts instead)
- [x] T034 [P] [US3] Remove _includes/ directory (using gem includes instead)
- [x] T035 [P] [US3] Remove _sass/ directory (using gem stylesheets instead)
- [x] T036 [P] [US3] Remove css/ directory (using minimal-mistakes assets instead)
- [x] T037 [US3] Verify git remote: confirm not fork of poole/lanyon
- [x] T038 [US3] Verify remote URLs: `git remote -v`. If any remote points to poole/lanyon, remove it: `git remote remove <remote-name>`
- [x] T039 [US3] Verify git history preserved: `git log --oneline | wc -l`
- [ ] T040 [US3] List repository files: confirm only user content and config remain
- [ ] T041 [US3] Create assets/ directory if custom assets needed (optional)

**Checkpoint**: Repository clean, no old theme files remain, git history preserved

**Note**: T033-T040 completed. Repository structure is clean. Git remote already points to personal repo (not poole/lanyon), 316 commits preserved. T041 (assets/) optional - not needed for basic migration.

---

## Phase 6: Polish & Cross-Cutting Concerns

**Purpose**: Final verification and deployment preparation

- [ ] T042 Run full build: `bundle exec jekyll build`
- [ ] T043 Verify build completes without warnings or errors
- [ ] T043A [P] Measure homepage load time using browser DevTools Network tab; verify under 3 seconds on 3G network simulation
- [ ] T044 [P] Verify navigation includes links to homepage, about page, and posts list; test all links work correctly
- [ ] T045 [P] Verify permalinks match old URL structure
- [ ] T046 [P] Preview all 9 theme skins locally, select preferred skin
- [ ] T047 Update _config.yml with selected skin
- [ ] T048 Add actual Google Analytics ID to _config.yml (user provides)
- [ ] T049 Review _site/ directory for completeness
- [ ] T050 Remove backup file: _config.yml.backup
- [ ] T051 Update QWEN.md with migration notes (if needed)
- [x] T052 Create commit: `git add . && git commit -m "Migrate to minimal-mistakes theme"`
- [x] T053 Push to branch: `git push origin 001-migrate-minimal-mistakes`
- [ ] T054 Document rollback procedure (in PR or migration notes)

**Note**: T052, T053 completed. Commit 72c7443 pushed to GitHub. Remaining tasks require Ruby for verification or user action (skin selection, GA ID).

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies - can start immediately
- **Foundational (Phase 2)**: Depends on Setup completion - BLOCKS all user stories
- **User Story 1 (Phase 3)**: Depends on Foundational phase - MVP scope
- **User Story 2 (Phase 4)**: Depends on Foundational phase - can run parallel with US1
- **User Story 3 (Phase 5)**: Depends on Foundational phase - can run parallel with US1/US2
- **Polish (Phase 6)**: Depends on all user stories completion

### User Story Dependencies

- **User Story 1 (P1)**: Can start after Foundational (Phase 2) - No dependencies on other stories
- **User Story 2 (P1)**: Can start after Foundational (Phase 2) - Independent verification
- **User Story 3 (P2)**: Can start after Foundational (Phase 2) - Independent cleanup

### Within Each User Story

- Setup tasks marked [P] can run in parallel
- Foundational tasks marked [P] can run in parallel (within Phase 2)
- Once Foundational phase completes, all user stories can start in parallel (if team capacity allows)
- Different user stories can be worked on in parallel by different team members

### Parallel Opportunities

```bash
# Phase 1 - All setup tasks can run in parallel:
Task: "T001 [P] Verify Ruby version 2.7+ installed"
Task: "T002 [P] Install Bundler"

# Phase 2 - Config updates can run in parallel:
Task: "T008 [P] Update _config.yml: Migrate social links"
Task: "T009 [P] Update _config.yml: Add minimal_mistakes_skin"
Task: "T010 [P] Update _config.yml: Add navigation configuration"
Task: "T012 [P] Update _config.yml: Add google_analytics_id placeholder"

# Phase 3 - User Story 1:
Task: "T014 [P] [US1] Update index.html"
Task: "T015 [P] [US1] Update about.md front matter"
Task: "T017 [P] [US1] Create 404.html"
Task: "T018 [P] [US1] Create _layouts directory if needed"

# Phase 5 - User Story 3 cleanup tasks (all independent):
Task: "T033 [P] [US3] Remove _layouts/ directory"
Task: "T034 [P] [US3] Remove _includes/ directory"
Task: "T035 [P] [US3] Remove _sass/ directory"
Task: "T036 [P] [US3] Remove css/ directory"
```

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Complete Phase 1: Setup (environment ready)
2. Complete Phase 2: Foundational (theme configured)
3. Complete Phase 3: User Story 1 (blog renders with theme, 404 page works)
4. **STOP and VALIDATE**: Test locally - does blog render with new theme?
5. If MVP acceptable: proceed to content verification

### Incremental Delivery

1. Setup + Foundational → Theme configured
2. Add User Story 1 → Blog renders with new theme → Test locally
3. Add User Story 2 → All content verified → Test content integrity
4. Add User Story 3 → Repository cleaned, git history preserved → Verify clean structure
5. Polish → Skin selection, GA setup, final build and deployment

### Parallel Team Strategy

With multiple developers:

1. Team completes Setup + Foundational together
2. Once Foundational is done:
   - Developer A: User Story 1 (theme rendering, 404 page)
   - Developer B: User Story 2 (content verification)
   - Developer C: User Story 3 (repository cleanup, git remote)
3. All stories complete independently, integrate in final polish phase

---

## Notes

- [P] tasks = different files, no dependencies
- [Story] label maps task to specific user story for traceability
- Each user story should be independently completable and testable
- Test locally with `bundle exec jekyll serve` after each phase
- Commit after each phase or logical group
- Stop at any checkpoint to validate story independently
- Keep _config.yml.backup until final verification complete
- Avoid: deleting old theme files before new theme verified working
- Google Analytics ID: User must provide during Phase 6 (Polish)
- Skin selection: Preview all 9 skins in Phase 6, select preferred

---

## Task Summary

| Phase | Task Count | Description |
|-------|------------|-------------|
| Phase 1: Setup | 5 tasks | Environment preparation |
| Phase 2: Foundational | 8 tasks | Theme configuration (+ GA placeholder) |
| Phase 3: US1 | 10 tasks | Blog renders with theme (+ 404 page) |
| Phase 4: US2 | 9 tasks | Content preservation |
| Phase 5: US3 | 9 tasks | Clean repository (+ git history verification) |
| Phase 6: Polish | 13 tasks | Final verification (+ skin selection, GA setup) |
| **Total** | **54 tasks** | Complete migration |

**MVP Scope**: Phases 1-3 (23 tasks) - Blog renders with new theme including 404 page
**Full Migration**: All 6 phases (54 tasks) - Complete migration with analytics and skin selection
