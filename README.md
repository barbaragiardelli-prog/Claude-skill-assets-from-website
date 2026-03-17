# Scrape Assets from Website

A Claude Code skill that scrapes product images and assets from any website URL. Creates a local asset repository with an interactive HTML viewer for browsing, filtering, and copying image URLs — perfect for prototyping.

![Claude Code](https://img.shields.io/badge/Claude_Code-skill-blueviolet)

## What it does

Give it any website URL and it will:

1. **Fetch the page** with browser-like headers
2. **Extract all images** — product shots, hero banners, lifestyle photos, logos
3. **Pair images with metadata** — product names, brand names from structured data
4. **Categorize** them (banners vs products vs lifestyle)
5. **Generate an interactive HTML viewer** with:
   - Grid layout with brand/product labels
   - Filter by brand
   - Click-to-zoom lightbox
   - One-click copy URL for each image
6. **Optionally download** all images locally into your project

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

In any Claude Code session:

```
/scrape-assets https://www.sephora.com/
/scrape-assets https://www.nike.com/
/scrape-assets https://www.glossier.com/
/scrape-assets https://www.apple.com/shop
```

## Example output

The skill generates an interactive HTML viewer like this:

- Product cards with brand name, product name, and high-res image
- Hero/banner images in a separate section
- Filter pills to narrow by brand
- Lightbox for full-size preview
- Copy-to-clipboard for each image URL

## Notes

- Images are for **prototyping only** — respect copyright for production use
- Some sites may block scraping; try specific product page URLs if the homepage doesn't work
- Works best with e-commerce sites that have structured product data
