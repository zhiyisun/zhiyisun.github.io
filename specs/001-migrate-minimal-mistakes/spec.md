# Feature Specification: Migrate Blog to Minimal Mistakes Theme

**Feature Branch**: `001-migrate-minimal-mistakes`
**Created**: 2026-03-19
**Status**: Draft
**Input**: Migrate blog to minimal-mistakes Jekyll theme, keeping about.md and posts

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Blog Renders with New Theme (Priority: P1)

As a blog owner, I want my blog to display using the minimal-mistakes theme so that it has a modern, clean appearance while retaining all my existing content.

**Why this priority**: This is the core migration outcome. Without a working themed blog, the migration has no value.

**Independent Test**: After migration, visiting the blog URL displays the site with minimal-mistakes styling, navigation works, and all posts are accessible.

**Acceptance Scenarios**:

1. **Given** the migration is complete, **When** I visit the homepage, **Then** I see the minimal-mistakes theme layout with my blog title and posts listed
2. **Given** the migration is complete, **When** I click on any existing post, **Then** the post content displays correctly with proper formatting
3. **Given** the migration is complete, **When** I visit the About page, **Then** I see my about content styled with the new theme

---

### User Story 2 - Content Preservation (Priority: P1)

As a blog owner, I want all my existing posts and about page content preserved exactly so that no content is lost during the migration.

**Why this priority**: Content is the primary value of the blog. Losing posts or corrupting content would be unacceptable.

**Independent Test**: All 17 existing posts and the about page content are present and render correctly after migration.

**Acceptance Scenarios**:

1. **Given** the migration is complete, **When** I compare the old and new about page content, **Then** the text content matches (excluding theme-specific layout changes)
2. **Given** the migration is complete, **When** I count the posts, **Then** all 17 posts from the original blog are present
3. **Given** the migration is complete, **When** I open any post, **Then** the content (text, code blocks, images, links) renders correctly

---

### User Story 3 - Clean Repository Structure (Priority: P2)

As a blog owner, I want the repository to contain only minimal-mistakes theme files and my content so that the repository is clean and not a fork of the old theme.

**Why this priority**: A clean repository ensures maintainability and removes confusion about the blog's base theme.

**Independent Test**: Repository no longer contains files from the lanyon/poole theme; only minimal-mistakes theme files and user content remain.

**Acceptance Scenarios**:

1. **Given** the migration is complete, **When** I list repository files, **Then** no lanyon or poole theme files remain
2. **Given** the migration is complete, **When** I check git remote, **Then** the repository is not configured as a fork of poole/lanyon
3. **Given** the migration is complete, **When** I view git log, **Then** all historical commits are preserved

---

### Edge Cases

- Posts with special characters in titles or content render correctly
- Posts with embedded images or resources display properly
- Older posts with potentially outdated front matter still render
- Mobile/responsive layout works for all content types
- See FR-011 for 404 error page requirement

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: Blog MUST use the minimal-mistakes Jekyll theme (https://github.com/mmistakes/minimal-mistakes)
- **FR-002**: All existing posts in `_posts/` directory MUST be preserved with original content intact
- **FR-003**: The About page content from `about.md` MUST be preserved and accessible
- **FR-004**: Blog MUST maintain the same URL structure for posts (date-based permalinks)
- **FR-005**: All posts MUST render correctly with proper Markdown formatting (code blocks, lists, links, images)
- **FR-006**: Navigation MUST include links to homepage, about page, and posts list
- **FR-007**: Repository MUST NOT retain files specific to the lanyon/poole theme
- **FR-008**: Blog configuration (title, email, social links, description) MUST be migrated from existing config
- **FR-009**: RSS feed functionality MUST be preserved
- **FR-010**: Google Analytics tracking MUST be configured with user-provided tracking ID in `_config.yml`
- **FR-011**: 404 error page MUST be enabled using minimal-mistakes default layout

### Key Entities

- **Post**: Blog post with front matter (title, date, layout) and Markdown content body
- **Page**: Static page (e.g., About) with front matter and content
- **Theme Configuration**: Settings that control minimal-mistakes appearance and behavior
- **Site Configuration**: Blog-wide settings (title, URL, author info, social links)
- **Analytics Configuration**: Google Analytics tracking ID and related settings

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: All 17 existing posts are accessible and render without errors
- **SC-002**: About page displays original content with new theme styling
- **SC-003**: Homepage loads and displays post list within 3 seconds
- **SC-004**: No broken links or missing images across all pages
- **SC-005**: Blog passes Jekyll build without warnings or errors
- **SC-006**: RSS feed generates correctly with all posts included
- **SC-007**: Mobile view displays content correctly on screens 320px wide and above

## Clarifications

### Session 2026-03-19

- Q: What level of theme customization is expected beyond the default skin? → A: Choose alternate built-in skin (air, aqua, blush, plum, sun, sea, mint, lava, or dusk)
- Q: Should SEO metadata and analytics tracking be configured during this migration? → A: Google Analytics only - add GA tracking ID to _config.yml
- Q: Which specific skin should be selected? → A: Defer to visual preview - preview all 9 skins locally and select based on preference
- Q: Should a custom 404 error page be created, or use the theme's default? → A: Use minimal-mistakes default 404 page - just enable in config
- Q: Should the existing git commit history be preserved or should the migration start fresh? → A: Preserve history, update remote - keep all commits, remove lanyon remote, set origin to your repo

## Assumptions

- Current blog is running Jekyll (based on existing `_config.yml`), so migration is theme replacement, not platform migration
- Posts use standard Jekyll front matter format compatible with minimal-mistakes
- Social media usernames from current config (twitter, github, googleplus) should be migrated
- Blog description and title from current config should be preserved
- The minimal-mistakes theme will be installed as a gem-based theme (recommended approach)

### Updated: Theme Customization

- Theme skin: Use one of the 9 built-in alternate skins (not `default`).
- Skin selection: Defer to visual preview during migration. User will preview all skins locally and select based on preference.
- Available skins: air, aqua, blush, plum, sun, sea, mint, lava, dusk.
- No custom SCSS or layout overrides required (maintains gem-based simplicity).

### Updated: Analytics

- Google Analytics: Add `google_analytics_id` to `_config.yml` (user to provide tracking ID during migration).
- Uses minimal-mistakes built-in GA integration (no custom code required).

### Updated: Error Handling

- 404 Page: Use minimal-mistakes default 404 layout. Enable via config (no custom file needed).

### Updated: Git History

- Git History: Preserve all existing commits. Remove lanyon remote and set origin to personal repository.
- Repository will no longer appear as fork of poole/lanyon, but commit history remains intact.
