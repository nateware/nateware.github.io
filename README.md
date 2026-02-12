# nateware.github.io

A personal blog hosted on GitHub Pages, built with Jekyll. Home to technical articles, leadership insights, and photography.

## About

This is a Jekyll-based blog hosted at [nateware.github.io](https://nateware.github.io), deployed automatically to GitHub Pages from the main branch. The site features a collection of technical posts, management principles, and thoughts on software engineering.

## Features

- **Jekyll 4.x** with Minima theme
- **GitHub Pages** deployment (automatic builds)
- **Random rotating background images** - 165 photography samples from personal collection
- **Content delivery network (CDN)** - assets served from Google Cloud Storage for performance
- **Responsive design** with semi-transparent content overlays for readability
- **Jekyll Feed plugin** for RSS/Atom feed support

## Setup & Development

### Prerequisites

- Ruby 3.4+
- Bundler
- Git

### Local Development

1. Clone the repository:
```bash
git clone https://github.com/nateware/nateware.github.io.git
cd nateware.github.io
```

2. Install dependencies:
```bash
bundle install
```

3. Start the local development server:
```bash
bundle exec jekyll serve
```

The site will be available at `http://localhost:4000`

### Creating Posts

Posts go in the `_posts/` directory with filename format: `YYYY-MM-DD-title.md`

Example:
```markdown
---
layout: post
title: "My Post Title"
date: 2024-02-11 12:00:00 -0800
categories: technology
tags: jekyll github pages
---

Your post content here...
```

The site is configured with `permalink: /:year/:month/:day/:title/` to maintain URL compatibility with the original WordPress blog structure.

## Special Configuration

### asset_cdn Setting

The blog uses a custom `asset_cdn` configuration in `_config.yml` to serve assets from Google Cloud Storage:

```yaml
asset_cdn: https://storage.googleapis.com/nateware-blog
```

When this is set, all asset paths (images, CSS, JavaScript) are prefixed with the CDN URL in the generated HTML. This is particularly useful for serving large image files without overloading the GitHub Pages infrastructure.

**How it works:**
- In `_config.yml`, set `asset_cdn: https://your-cdn-url`
- In templates (`_includes/head.html`), assets reference `{{ site.asset_cdn }}{{ asset_path }}`
- During local development, leave `asset_cdn` empty or unset to serve assets directly from the repo

## Random Background Images

The blog features a dynamic random background image that changes with each page load or navigation.

### Under the Hood

1. **Jekyll Plugin** (`_plugins/background_images.rb`) - Scans `assets/backgrounds/` directory and generates a JavaScript array of all image paths during build time
2. **Generated JavaScript** (`assets/js/backgrounds.js`) - Contains the array of 165 background image paths
3. **Client-side Selection** (`_includes/head.html`) - JavaScript picks a random image on page load and applies it to the body
4. **CSS Styling** (`assets/css/background.css`) - Makes the background cover the full viewport while maintaining aspect ratio

### Adding New Background Images

When you add new background images to the `assets/backgrounds/` directory, you need to regenerate the JavaScript array:

```bash
rake backgrounds
```

This Rake task scans the backgrounds directory and regenerates `assets/js/backgrounds.js` with the new images.

**Important:** After adding new images, if you're using Google Cloud Storage CDN, also run:

```bash
rake sync_assets
```

This syncs all assets to the GCS bucket. **Don't forget this step** - otherwise new background images won't be available to the CDN!

### Background Image Requirements

- **Format:** JPEG files (`.jpeg` or `.jpg` extension)
- **Location:** `assets/backgrounds/`
- **Size:** Large images are fine - they're only downloaded when randomly selected
- **Aspect ratio:** Any ratio works - CSS uses `background-size: cover` which maintains aspect ratio

## Rake Tasks

### `rake backgrounds`

Regenerates the background images JavaScript array. Run this whenever you:
- Add new images to `assets/backgrounds/`
- Remove old background images
- Want to refresh the list

```bash
rake backgrounds
```

**Output:**
```
Generating background images JavaScript...
Backgrounds: Generated assets/js/backgrounds.js with 165 images
Don't forget to run `rake sync_assets` to upload assets to the CDN!
```

### `rake sync_assets`

Syncs the entire `assets/` directory to Google Cloud Storage. This is needed for CDN deployment.

```bash
rake sync_assets
```

Prerequisites:
- `gcloud` CLI tool installed and configured
- Permission to write to the GCS bucket (`nateware-blog`)

**Typical workflow:**
```bash
# Add or update background images
cp new-photo.jpeg assets/backgrounds/

# Regenerate the backgrounds.js file
rake backgrounds

# Sync all assets to CDN
rake sync_assets

# Commit and push (GitHub Pages handles the Jekyll build)
git add assets/
git commit -m "Add new background images"
git push
```

## Deployment

### GitHub Pages (Automatic)

The `main` branch is automatically built and deployed by GitHub Pages. Just push your changes:

```bash
git add .
git commit -m "Add new post"
git push origin main
```

GitHub Pages will:
1. Detect changes to the repo
2. Run Jekyll build (built-in plugins only)
3. Deploy the `_site/` directory to the live site

The custom plugin (`_plugins/background_images.rb`) won't run on GitHub Pages due to security restrictions, but the generated `assets/js/backgrounds.js` file will be deployed as a static asset.

### Manual Build (Optional)

If you want to rebuild locally before pushing:

```bash
bundle exec jekyll clean
bundle exec jekyll build
```

The output will be in `_site/` directory.

## Theme Customization

The blog uses the [Minima](https://github.com/jekyll/minima) Jekyll theme with the following customizations:

### Files Overriding Minima

- `_layouts/post.html` - Custom post layout with Like button integration
- `_includes/head.html` - Extended to include background CSS/JS and CDN support
- `_includes/fixedheader.css` - Custom header styling (referenced in head.html)
- `assets/css/background.css` - Background image styling

To make further customizations to the theme, copy the file from the minima gem to your project root and modify it.

## File Structure

```
nateware.github.io/
├── _includes/              # Template includes (theme overrides)
│   ├── head.html          # Minima override - adds CDN & background JS
│   └── fixedheader.css    # Header styling
├── _layouts/              # Layout templates (theme overrides)
│   └── post.html          # Custom post layout
├── _plugins/              # Custom Jekyll plugins
│   └── background_images.rb
├── _posts/                # Blog posts
├── assets/
│   ├── backgrounds/       # 165 JPEG background images
│   ├── css/
│   │   └── background.css # Background image styling
│   ├── images/            # Misc images
│   ├── js/
│   │   └── backgrounds.js # Auto-generated by rake backgrounds
│   └── main.css           # Main stylesheet (from minima)
├── _config.yml            # Jekyll configuration
├── Gemfile               # Ruby dependencies
├── Rakefile              # Rake tasks (backgrounds, sync_assets)
└── README.md             # This file
```

## Troubleshooting

### Background Images Not Updating

If you added new images but they're not appearing:

1. Run `rake backgrounds` to regenerate the JavaScript array
2. Run `bundle exec jekyll clean && bundle exec jekyll build` to rebuild locally
3. Verify `assets/js/backgrounds.js` contains the new images
4. If using CDN, run `rake sync_assets` to push to cloud storage

### Assets Not Loading from CDN

Check the `asset_cdn` setting in `_config.yml`:

```bash
grep asset_cdn _config.yml
```

If CDN URLs aren't working:
1. Verify the GCS bucket URL is correct
2. Check bucket permissions (should be public or authenticated)
3. Run `rake sync_assets` to ensure files are present in the bucket
4. Test direct URL access: `https://storage.googleapis.com/nateware-blog/assets/css/main.css`

### Local Server Not Picking Up Changes

Sometimes the Jekyll incremental build cache gets stale:

```bash
bundle exec jekyll clean
bundle exec jekyll serve
```

Or use the `--no-incremental` flag:

```bash
bundle exec jekyll serve --no-incremental
```

## License

Personal blog content. See LICENSE file for details.

## Contact

- GitHub: [@nateware](https://github.com/nateware)
- LinkedIn: [natewiger](https://www.linkedin.com/in/natewiger)
