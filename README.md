# Scrape Assets from Website

A Claude Code skill that scrapes product images from any website and downloads them directly into your project's asset folder — ready to use in prototypes, portable on any device.

![Claude Code](https://img.shields.io/badge/Claude_Code-skill-blueviolet)

## What it does

Give it any website URL and it will:

1. **Fetch the page** with browser-like headers
2. **Extract all images** — product shots, hero banners, lifestyle photos
3. **Pair images with metadata** — product names, brand names from structured data
4. **Auto-detect your project's asset folder** (e.g., `public/images/`, `src/assets/`)
5. **Download everything locally** with clean filenames (e.g., `rare-beauty-soft-pinch-liquid-blush.jpg`)
6. **Generate a manifest.json** for programmatic use
7. **Generate an interactive HTML viewer** (`assets.html`) to browse, filter, and grab file paths

Everything is local — no external URLs, no server needed. Share your project and the assets come with it.

## Install

```bash
# Create the commands folder if it doesn't exist
mkdir -p ~/.claude/commands

# Download the skill
curl -o ~/.claude/commands/scrape-assets.md \
  https://raw.githubusercontent.com/barbaragiardelli-prog/Claude-skill-assets-from-website/main/scrape-assets.md
```

Or manually: download `scrape-assets.md` and drop it into `~/.claude/commands/`.

## Usage

In any Claude Code session, inside your project:

```
/scrape-assets https://www.sephora.com/
/scrape-assets https://www.nike.com/
/scrape-assets https://www.glossier.com/
/scrape-assets https://www.apple.com/shop
```

The skill will:
1. Find your project's asset folder
2. Ask for confirmation before writing
3. Download all images into `<your-asset-folder>/scraped/<domain>/`
4. Show you the import paths ready to paste into your code

## Project structure after scraping

```
your-project/
  assets/
    scraped/
      sephora.com/
        banners/
          banner-korean-skincare-launch.jpg
          banner-march-minis.jpg
        products/
          rare-beauty-soft-pinch-liquid-blush.jpg
          dior-jadore-parfum.jpg
          glossier-boy-brow.jpg
        assets.html       ← browse everything visually
        manifest.json     ← metadata for all images
```

## Using the assets in your code

```html
<img src="./assets/scraped/sephora.com/products/rare-beauty-soft-pinch-liquid-blush.jpg">
```

```css
background-image: url('./assets/scraped/sephora.com/banners/banner-korean-skincare.jpg');
```

## Notes

- Images are for **prototyping only** — respect copyright for production use
- Some sites may block scraping; try specific product page URLs if the homepage doesn't work
- Works best with e-commerce sites that have structured product data
