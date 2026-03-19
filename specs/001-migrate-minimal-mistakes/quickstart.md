# Quickstart: Local Development Setup

**Created**: 2026-03-19
**Updated**: 2026-03-19 (post-clarification)
**Feature**: [spec.md](spec.md) | [plan.md](plan.md)

## Purpose

This guide helps you set up a local development environment to preview the blog with the minimal-mistakes theme before deploying to GitHub Pages. Includes steps for previewing all 9 theme skins.

---

## Prerequisites

### Required Software

1. **Ruby** (version 2.7 or higher)
   - Check: `ruby --version`
   - Install: Use [rbenv](https://github.com/rbenv/rbenv) or [RVM](https://rvm.io/)

2. **Bundler** (Ruby dependency manager)
   - Check: `bundle --version`
   - Install: `gem install bundler`

3. **Git**
   - Check: `git --version`
   - Install: [git-scm.com](https://git-scm.com/)

---

## Setup Steps

### Step 1: Navigate to Repository

```bash
cd /path/to/zhiyisun.github.io
```

### Step 2: Create Gemfile

Create a file named `Gemfile` in the repository root:

```ruby
source "https://rubygems.org"

gem "minimal-mistakes-jekyll"
gem "jekyll"
gem "kramdown"
gem "jekyll-feed"
```

### Step 3: Install Dependencies

```bash
bundle install
```

This installs:
- `minimal-mistakes-jekyll` theme
- `jekyll` static site generator
- `kramdown` Markdown parser
- `jekyll-feed` for RSS feed
- All transitive dependencies

### Step 4: Update Configuration

Update `_config.yml` with minimal-mistakes configuration:

```yaml
# Theme
theme: minimal-mistakes-jekyll
minimal_mistakes_skin: default  # Start with default for preview

# Author
author:
  name: Zhiyi Sun
  links:
    - type: twitter
      url: https://twitter.com/zhiyisun
    - type: github
      url: https://github.com/zhiyisun

# Google Analytics (optional, add your ID)
google_analytics_id: UA-XXXXX-X  # Replace with your ID
```

### Step 5: Start Local Server

```bash
bundle exec jekyll serve
```

You should see output like:

```text
Configuration file: /path/to/zhiyisun.github.io/_config.yml
            Source: /path/to/zhiyisun.github.io
       Destination: /path/to/zhiyisun.github.io/_site
 Incremental build: disabled. Enable with --incremental
      Generating... 
       Jekyll Feed: Generating feed for posts
        ...done in 2.5 seconds.

 Auto-regeneration: enabled for '/path/to/zhiyisun.github.io'
    Server address: http://127.0.0.1:4000/
  Server running... press ctrl-c to stop.
```

### Step 6: Preview in Browser

Open your browser and navigate to: **http://127.0.0.1:4000/**

---

## Theme Skin Preview

One of the key steps in this migration is selecting your preferred theme skin. Minimal-mistakes includes 9 built-in skins.

### Available Skins

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

### Preview Process

1. **Start the server** (if not already running):
   ```bash
   bundle exec jekyll serve
   ```

2. **Open `_config.yml`** in your editor

3. **Find the `minimal_mistakes_skin` line**:
   ```yaml
   minimal_mistakes_skin: default
   ```

4. **Change the skin value** to each option:
   ```yaml
   minimal_mistakes_skin: air
   ```

5. **Refresh your browser** - Jekyll auto-regenerates

6. **Browse the site** - Check homepage, about page, and a few posts

7. **Repeat** for each skin you want to preview

8. **Note your favorites** - Keep a list of preferred skins

9. **Final selection** - Once decided, keep that skin value

### Quick Preview Script (Optional)

For faster previewing, you can use this bash loop:

```bash
for skin in air aqua blush plum sun sea mint lava dusk; do
  echo "Previewing skin: $skin"
  sed -i "s/minimal_mistakes_skin:.*/minimal_mistakes_skin: $skin/" _config.yml
  echo "Press Enter to continue to next skin..."
  read
done
```

**Note**: This modifies `_config.yml` in place. Remember your preferred skin!

---

## Google Analytics Setup (Optional)

If you want to enable analytics tracking:

### Step 1: Create Google Analytics Account

1. Go to [analytics.google.com](https://analytics.google.com)
2. Sign in with Google account
3. Create new property for `zhiyisun.github.io`
4. Get your tracking ID

### Step 2: Add to Configuration

Edit `_config.yml`:

```yaml
# For Universal Analytics (older format)
google_analytics_id: UA-123456-1

# OR for Google Analytics 4 (newer format)
google_analytics_id: G-XXXXXXXXXX
```

### Step 3: Verify

After deploying to GitHub Pages:
1. Visit your blog
2. Check GA Real-Time reports
3. You should see your visit tracked

---

## Common Tasks

### Preview Specific Post

Navigate to: `http://127.0.0.1:4000/year/month/day/post-title/`

Example: `http://127.0.0.1:4000/2015/04/08/linux-kernel-development-with-qemu/`

### Preview About Page

Navigate to: **http://127.0.0.1:4000/about/**

### Preview 404 Page

Navigate to: **http://127.0.0.1:4000/nonexistent-page**

### Test Mobile Layout

Use browser DevTools:
1. Open DevTools (F12 or Cmd+Option+I)
2. Click device toggle (Ctrl+Cmd+M or mobile icon)
3. Select device (iPhone, iPad, Android, etc.)
4. Verify layout at different screen sizes

### Check for Build Errors

Watch the terminal output while `jekyll serve` is running. Any errors will appear immediately:

```text
jekyll 3.9.0 | Error:  Invalid front matter in _posts/2015-04-08-post.md
```

### Stop Server

Press `Ctrl+C` in the terminal.

---

## Troubleshooting

### Error: "Could not find gems 'minimal-mistakes-jekyll'"

**Solution**: Run `bundle install` again. Ensure Gemfile exists.

### Error: "Address already in use"

**Solution**: Port 4000 is in use. Either:
- Stop the other process using port 4000
- Or run on different port: `bundle exec jekyll serve --port 4001`

### Error: "Invalid post front matter"

**Solution**: Check the post file for YAML syntax errors. Ensure front matter is wrapped with `---`:

```yaml
---
layout: single
title: "Post Title"
---
```

### Styles not loading

**Solution**: 
1. Hard refresh browser (Ctrl+Shift+R or Cmd+Shift+R)
2. Clear browser cache
3. Restart `jekyll serve`

### Posts not showing

**Solution**: 
1. Check post filename format: `YYYY-MM-DD-slug.md`
2. Check front matter has `layout: single`
3. Check date is not in the future

### Google Analytics not tracking

**Solution**:
1. Verify tracking ID is correct format
2. Check `_config.yml` syntax (proper indentation)
3. GA may take 24-48 hours to show data
4. Use Real-Time reports for immediate verification

---

## Deployment Preview

Before pushing to GitHub:

1. **Build production version**:
   ```bash
   bundle exec jekyll build
   ```

2. **Check generated files**:
   ```bash
   ls -la _site/
   ```

3. **Verify all posts exist**:
   ```bash
   ls _site/posts/
   ```

4. **Test RSS feed**:
   Navigate to: `http://127.0.0.1:4000/feed.xml`

5. **Test 404 page**:
   Navigate to: `http://127.0.0.1:4000/nonexistent`

---

## Next Steps

After local testing is complete:

1. **Finalize skin selection**: Set `minimal_mistakes_skin` in `_config.yml`
2. **Add GA tracking ID** (if using analytics)
3. **Commit changes**: `git add . && git commit -m "Migrate to minimal-mistakes theme"`
4. **Push to branch**: `git push origin 001-migrate-minimal-mistakes`
5. **Create pull request** (if using PR workflow)
6. **Merge to master** to trigger GitHub Pages deployment

---

## References

- [Jekyll Documentation](https://jekyllrb.com/docs/)
- [Minimal Mistakes Documentation](https://mmistakes.github.io/minimal-mistakes/docs/quick-start-guide/)
- [Minimal Mistakes Skins](https://mmistakes.github.io/minimal-mistakes/docs/configuration/#site-skin)
- [GitHub Pages Setup](https://pages.github.com/)
- [Google Analytics Setup](https://support.google.com/analytics/answer/1008080)
