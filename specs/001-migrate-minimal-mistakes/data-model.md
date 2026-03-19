# Data Model: Blog Content Structure

**Created**: 2026-03-19
**Updated**: 2026-03-19 (post-clarification)
**Feature**: [spec.md](spec.md) | [plan.md](plan.md)

## Overview

This document describes the content structure for the blog migration. Since this is a static blog, "data model" refers to the structure of content files (front matter and content format).

---

## Post Structure

### File Location
`_posts/YYYY-MM-DD-post-slug.md`

### Front Matter Fields

| Field | Type | Required | Description | Example |
|-------|------|----------|-------------|---------|
| `layout` | string | Yes | Must be `single` for minimal-mistakes | `layout: single` |
| `title` | string | Yes | Post title | `title: "Linux Kernel development with QEMU"` |
| `date` | datetime | Yes | Publication date (can be in filename or front matter) | `date: 2015-04-08 10:00:00` |
| `categories` | array | No | Post categories for navigation | `categories: [Linux, QEMU]` |
| `tags` | array | No | Post tags for filtering | `tags: [ARM, kernel, debugging]` |
| `excerpt` | string | No | Custom excerpt for post listings | `excerpt: "Guide to kernel development..."` |
| `header.overlay_image` | string | No | Featured image for post | `header.overlay_image: /assets/images/post-image.jpg` |
| `toc` | boolean | No | Show table of contents | `toc: true` |
| `toc_label` | string | No | Custom TOC title | `toc_label: "On this page"` |

### Content Body

- **Format**: Markdown (kramdown parser)
- **Supported elements**:
  - Headings (`##`, `###`, etc.)
  - Code blocks with syntax highlighting (```language)
  - Images with alt text
  - Links (internal and external)
  - Lists (ordered and unordered)
  - Blockquotes
  - Tables
  - HTML (when needed for special formatting)

### Example Post

```yaml
---
layout: single
title: "Linux Kernel development with QEMU"
date: 2015-04-08 10:00:00
categories: [Linux, QEMU]
tags: [ARM, kernel, debugging]
toc: true
toc_label: "Contents"
---

Post content in Markdown format...

## Section Heading

Code example:

```c
#include <stdio.h>

int main() {
    return 0;
}
```
```

---

## Page Structure (About, etc.)

### File Location
Root directory: `about.md`, `filename.md`

### Front Matter Fields

| Field | Type | Required | Description | Example |
|-------|------|----------|-------------|---------|
| `layout` | string | Yes | Page layout type | `layout: single` |
| `title` | string | Yes | Page title | `title: About` |
| `permalink` | string | No | Custom URL path | `permalink: /about/` |
| `excerpt` | string | No | Page description | `excerpt: "About the author"` |

### Example Page

```yaml
---
layout: single
title: About
permalink: /about/
---

Page content in Markdown format...
```

---

## 404 Error Page Structure

### File Location
Root directory: `404.html`

### Front Matter Fields

| Field | Type | Required | Description | Example |
|-------|------|----------|-------------|---------|
| `layout` | string | Yes | Must be `default` for 404 | `layout: default` |
| `title` | string | Yes | Page title | `title: Page Not Found` |
| `permalink` | string | Yes | Must be `/404.html` | `permalink: /404.html` |

### Example 404 Page

```yaml
---
layout: default
title: Page Not Found
permalink: /404.html
---

<p>Sorry, but the page you were trying to view does not exist.</p>
```

---

## Site Configuration Structure

### File Location
`_config.yml`

### Required Fields for Minimal-Mistakes

```yaml
# Site settings
title: Zhiyi's Blog
email: zhiyisun@gmail.com
description: >
  I am Zhiyi Sun. I like Open Source Software(OSS), especially Linux...
baseurl: ""
url: "http://zhiyisun.github.io"

# Theme settings
theme: minimal-mistakes-jekyll
minimal_mistakes_skin: [air|aqua|blush|plum|sun|sea|mint|lava|dusk]

# Author settings
author:
  name: Zhiyi Sun
  links:
    - type: twitter
      url: https://twitter.com/zhiyisun
    - type: github
      url: https://github.com/zhiyisun

# Build settings
markdown: kramdown

# Google Analytics
google_analytics_id: UA-XXXXX-X  # User provides
```

### Optional Fields

```yaml
# Navigation
navigation:
  - title: Home
    url: /
  - title: About
    url: /about/
  - title: Posts
    url: /posts/

# Footer
footer:
  links:
    - type: twitter
      url: https://twitter.com/zhiyisun
    - type: github
      url: https://github.com/zhiyisun

# Posts settings (defaults for all posts)
defaults:
  - scope:
      path: ""
      type: posts
    values:
      layout: single
      author_profile: true
      read_time: true
      comments: false
      share: true
      related: true
```

---

## Analytics Configuration Structure

### File Location
`_config.yml` (same file as site configuration)

### Google Analytics Fields

| Field | Type | Required | Description | Example |
|-------|------|----------|-------------|---------|
| `google_analytics_id` | string | No (but recommended) | GA tracking ID | `UA-123456-1` or `G-XXXXXXXXXX` |

### Notes

- **Universal Analytics (UA)**: Format `UA-XXXXX-Y` (being phased out)
- **Google Analytics 4 (GA4)**: Format `G-XXXXXXXXXX` (current standard)
- Minimal-mistakes supports both formats
- If user doesn't have GA, they can skip this field and add later

---

## Validation Rules

### Post Validation
- MUST have `layout` field set to `single`
- MUST have `title` field
- MUST have valid date (in filename or front matter)
- MUST be in `_posts/` directory
- Filename MUST follow `YYYY-MM-DD-slug.md` pattern

### Page Validation
- MUST have `layout` field
- MUST have `title` field (for most pages)
- MUST be valid Markdown

### 404 Page Validation
- MUST have `layout: default`
- MUST have `permalink: /404.html`
- MUST be named `404.html`

### Configuration Validation
- MUST specify `theme: minimal-mistakes-jekyll`
- MUST have `title` and `url` fields
- MUST have valid `author` structure if using author links
- MUST have `google_analytics_id` if analytics tracking required

---

## Content Migration Checklist

For each post:
- [ ] Front matter `layout` changed from `post` to `single`
- [ ] Date preserved correctly
- [ ] Title preserved correctly
- [ ] Content body unchanged
- [ ] Code blocks render correctly
- [ ] Images display correctly
- [ ] Links work correctly

For configuration:
- [ ] Theme specified correctly
- [ ] Social links migrated to `author.links` format
- [ ] Site title and description preserved
- [ ] Build settings preserved (markdown: kramdown)
- [ ] Google Analytics ID added (if provided)
- [ ] Skin selected and configured

For pages:
- [ ] About page layout updated to `single`
- [ ] 404 page created with correct front matter
