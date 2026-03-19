# Research: Migrate Blog to Minimal Mistakes Theme

**Created**: 2026-03-19
**Updated**: 2026-03-19 (post-clarification)
**Feature**: [spec.md](spec.md) | [plan.md](plan.md)

---

## Decision: Migration Approach

**What was chosen**: Gem-based theme installation with minimal-mistakes-jekyll gem

**Why chosen**:
- Simplest approach aligned with Constitution Principle III (Simplicity/YAGNI)
- GitHub Pages supports gem-based themes
- Theme updates become trivial (just update gem version)
- No need to manage theme source files in repository
- Keeps repository focused on content, not theme code

**Alternatives considered**:
1. **Fork minimal-mistakes and customize**: Rejected - adds unnecessary complexity, harder to maintain, violates Simplicity principle
2. **Copy theme files into repository**: Rejected - bloats repository, makes theme updates difficult
3. **Remote theme via jekyll-remote-theme**: Rejected - gem-based is simpler and natively supported by GitHub Pages

---

## Decision: File Inventory

**What was identified**: Current repository structure analysis

### Files to Preserve (User Content)
- `_posts/*.md` - All 17 blog posts
- `about.md` - About page content
- `_config.yml` - Configuration (needs update, not replacement)
- `index.html` - Homepage layout (may need minor updates)
- `feed.xml` - RSS feed (should be preserved)

### Files to Remove (Lanyon/Poole Theme)
- `_layouts/` - Lanyon theme layouts (will use minimal-mistakes gem layouts)
- `_includes/` - Lanyon theme includes (will use minimal-mistakes gem includes)
- `_sass/` - Lanyon theme stylesheets (will use minimal-mistakes gem stylesheets)
- `css/` - Lanyon theme CSS (will use minimal-mistakes gem assets)
- `Gemfile` / `Gemfile.lock` - Will be regenerated with minimal-mistakes dependency

### Files to Create
- `Gemfile` - Ruby dependency file with minimal-mistakes-jekyll gem
- `_config.yml` updates - Add minimal-mistakes theme configuration, GA tracking
- `assets/` directory - For custom assets if needed
- `404.html` - Optional: Use minimal-mistakes default 404 layout

**Rationale**: Clear separation between user content (posts, pages, config) and theme files (layouts, includes, stylesheets). This ensures content preservation while cleanly removing old theme.

---

## Decision: Configuration Mapping

**What was chosen**: Map existing config to minimal-mistakes structure

### Existing Config Values (to preserve)
```yaml
title: Zhiyi's Blog
email: zhiyisun@gmail.com
description: >
  I am Zhiyi Sun. I like Open Source Software(OSS), especially Linux...
baseurl: ""
url: "http://zhiyisun.github.io"
twitter_username: zhiyisun
github_username: zhiyisun
googleplus_username: 112990371546622291108
markdown: kramdown
```

### Minimal-Mistakes Required Additions
```yaml
theme: minimal-mistakes-jekyll
minimal_mistakes_skin: [air|aqua|blush|plum|sun|sea|mint|lava|dusk]  # User selects during preview

author:
  name: Zhiyi Sun
  links:
    - type: twitter
      url: https://twitter.com/zhiyisun
    - type: github
      url: https://github.com/zhiyisun

google_analytics_id: UA-XXXXX-X  # User provides during migration
```

**Why chosen**: Preserves all existing blog identity while adopting minimal-mistakes structure. The `author.links` format is minimal-mistakes convention for social links.

**Alternatives considered**:
1. **Keep old social link format**: Rejected - minimal-mistakes uses `author.links` convention
2. **Use default skin**: Rejected - user chose alternate skin for more personalized look

---

## Decision: Front Matter Compatibility

**What was found**: Existing posts use standard Jekyll front matter

### Current Post Front Matter Pattern
```yaml
---
layout: post
title: "Post Title"
---
```

### Minimal-Mistakes Requirements
```yaml
---
layout: single  # or 'posts' for blog posts
title: "Post Title"
date: YYYY-MM-DD HH:MM:SS +TZ
categories: [category1, category2]  # optional
tags: [tag1, tag2]  # optional
excerpt: "Custom excerpt"  # optional
toc: true  # optional
toc_label: "On this page"  # optional
```

**Compatibility Assessment**: 
- `layout: post` → needs to change to `layout: single` (minimal-mistakes convention)
- Date format in filenames (`YYYY-MM-DD-title.md`) is compatible
- Title field is compatible
- Categories/tags are optional in both

**Migration Strategy**: 
- Option A: Bulk update all posts' front matter (change `layout: post` to `layout: single`)
- Option B: Create `_layouts/post.html` that inherits from `single` layout

**Chosen**: Option A - Bulk update is cleaner and follows minimal-mistakes conventions directly.

---

## Decision: GitHub Pages Compatibility

**What was confirmed**: minimal-mistakes-jekyll is compatible with GitHub Pages

**Requirements**:
1. Use `github-pages` gem OR specify theme in `_config.yml` with proper Gemfile
2. GitHub Pages build process will install theme gem automatically

**Recommended Setup**:
```ruby
# Gemfile
source "https://rubygems.org"
gem "minimal-mistakes-jekyll"
gem "jekyll"
gem "kramdown"
gem "jekyll-feed"
```

```yaml
# _config.yml
theme: minimal-mistakes-jekyll
```

**Why chosen**: Standard gem-based approach works seamlessly with GitHub Pages build process.

---

## Decision: Theme Skin Selection

**What was chosen**: Defer skin selection to visual preview during migration

**Why chosen**:
- Skin preference is subjective and visual
- User can preview all 9 skins locally in minutes
- Changing skin is trivial (single config line change)
- Avoids back-and-forth communication delay

**Available Skins** (9 options):
| Skin | Description |
|------|-------------|
| air | Light, airy theme with white background |
| aqua | Blue-tinted theme |
| blush | Pink-tinted theme |
| plum | Purple-tinted theme |
| sun | Yellow-tinted theme |
| sea | Teal/green-tinted theme |
| mint | Fresh green-tinted theme |
| lava | Red-tinted theme |
| dusk | Dark theme (dark mode) |

**Preview Process**:
1. Install minimal-mistakes gem
2. Run `bundle exec jekyll serve`
3. Change `minimal_mistakes_skin` in `_config.yml`
4. Refresh browser to see each skin
5. Select preferred skin

---

## Decision: Google Analytics Integration

**What was chosen**: Add Google Analytics tracking via `_config.yml`

**Why chosen**:
- Minimal-mistakes has built-in GA support
- Requires only tracking ID in config
- No custom code or plugins needed
- Helps understand reader engagement

**Configuration**:
```yaml
# _config.yml
google_analytics_id: UA-XXXXX-X  # User provides
```

**Note**: User needs to provide their GA tracking ID during migration. If they don't have one, they can:
1. Create Google Analytics account
2. Set up property for zhiyisun.github.io
3. Get tracking ID (format: UA-XXXXX-X or G-XXXXXXXXXX for GA4)
4. Add to `_config.yml`

---

## Decision: 404 Error Page

**What was chosen**: Use minimal-mistakes default 404 layout

**Why chosen**:
- Minimal-mistakes includes well-designed 404 page
- No custom file needed
- Consistent with theme styling
- GitHub Pages automatically serves 404.html

**Implementation**:
- Option A: Create `404.html` with minimal-mistakes front matter
- Option B: Rely on theme's built-in 404 handling

**Chosen**: Option A - Create simple `404.html` file:
```yaml
---
layout: default
title: Page Not Found
permalink: /404.html
---
<p>Sorry, but the page you were trying to view does not exist.</p>
```

---

## Decision: Git History Treatment

**What was chosen**: Preserve all existing commits, remove lanyon remote only

**Why chosen**:
- Preserves user's contribution history
- Removes fork association with poole/lanyon
- No history rewriting (safer, reversible)
- Clean repository without losing work history

**Git Commands**:
```bash
# Remove old remote
git remote remove origin

# Add new origin (if not already set)
git remote add origin git@github.com:zhiyisun/zhiyisun.github.io.git

# Verify
git remote -v
```

**Note**: This does NOT rewrite history. All commits remain intact. Only the remote URL changes.

---

## Migration Steps Summary

1. **Backup**: Create branch (already done: `001-migrate-minimal-mistakes`)
2. **Create Gemfile**: Add minimal-mistakes-jekyll, jekyll-feed dependencies
3. **Install dependencies**: `bundle install`
4. **Update _config.yml**: 
   - Add `theme: minimal-mistakes-jekyll`
   - Add `minimal_mistakes_skin: [selected]`
   - Restructure social links to `author.links`
   - Add `google_analytics_id` (user provides)
5. **Remove old theme directories**: `_layouts/`, `_includes/`, `_sass/`, `css/`
6. **Update post front matter**: Change `layout: post` to `layout: single` for all 17 posts
7. **Update about.md**: Change layout to `single`
8. **Update index.html**: Ensure compatible with minimal-mistakes
9. **Create 404.html**: Simple 404 page with theme layout
10. **Skin preview**: User previews all 9 skins locally, selects preferred
11. **Test locally**: `bundle exec jekyll serve`
12. **Verify**: Check all posts, about page, RSS feed, 404 page
13. **Git cleanup**: Remove lanyon remote, verify origin
14. **Commit and push**: Deploy to GitHub Pages

---

## Rollback Procedure

If migration fails:
1. Stay on current branch (do not merge)
2. Return to `master` branch
3. Blog continues working with lanyon theme
4. Investigate issues on migration branch

---

## References

- Minimal Mistakes Documentation: https://mmistakes.github.io/minimal-mistakes/docs/quick-start-guide/
- Minimal Mistakes GitHub: https://github.com/mmistakes/minimal-mistakes
- Minimal Mistakes Skins: https://mmistakes.github.io/minimal-mistakes/docs/configuration/#site-skin
- GitHub Pages Dependencies: https://pages.github.com/versions/
- Jekyll Theme Migration: https://jekyllrb.com/docs/themes/
- Google Analytics Setup: https://support.google.com/analytics/answer/1008080
