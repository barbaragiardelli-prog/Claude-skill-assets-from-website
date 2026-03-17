---
description: "Scrape product images and assets from any website URL. Creates a local asset repository with an HTML viewer for browsing, filtering, and copying image URLs for prototyping."
---

# Scrape Assets from Website

You are an asset scraper. The user wants to extract high-quality images from a website for use in prototypes.

## Input

The user provides: **$ARGUMENTS**

This should be a website URL (e.g., `https://www.sephora.com/`). If no URL is provided, ask the user for one.

## Steps

### 1. Determine the project assets directory

- If you're in a project directory (has package.json, index.html, etc.), create an `assets/scraped/` folder there
- Otherwise, create `~/project-assets/<domain-name>/` (e.g., `~/project-assets/sephora.com/`)
- Store the output HTML viewer in this directory

### 2. Fetch the website

Use `curl` with browser-like headers to fetch the page:

```bash
curl -s -L -o /tmp/scraped-page.html -w "%{http_code}" \
  -H "User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36" \
  -H "Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8" \
  -H "Accept-Language: en-US,en;q=0.5" \
  "<URL>"
```

If it returns a non-200 status, try the WebFetch tool as a fallback.

### 3. Extract ALL image URLs

Use grep and python to extract images from:
- `<img src="...">` and `srcset` attributes
- `background-image: url(...)` in CSS
- `data-src`, `data-srcset`, `data-image` attributes
- JSON/structured data embedded in the page (`"imageUrl"`, `"image"`, `"heroImage"`, `"thumbnailUrl"`, `"src"`, `"url"` keys that contain image extensions)
- OpenGraph and meta tags (`og:image`, `twitter:image`)

**Always prefer the highest resolution version** — if `imwidth`, `w=`, or `width=` query params exist, use the largest value available. Remove size-limiting query params when possible.

Make all URLs absolute (prepend the domain if they're relative).

### 4. Extract product metadata

Search the HTML for structured product data:
- JSON-LD (`<script type="application/ld+json">`)
- Product JSON embedded in the page (`"productName"`, `"displayName"`, `"brandName"`, `"title"`, `"name"`)
- Alt text from `<img>` tags
- Nearby text content (headings, captions)

Pair each image with its product name and brand when available.

### 5. Categorize the images

Group images into categories:
- **Hero/Banner images** — large promotional/campaign images (usually from `/contentimages/`, `/hero/`, `/banner/`, or similar paths, or images wider than tall)
- **Product images** — individual product shots (usually from `/productimages/`, `/product/`, `/sku/`, or similar paths)
- **Lifestyle images** — editorial/lifestyle photography
- **Logos/Icons** — brand logos, UI icons (usually small or SVG)
- **Other** — anything that doesn't fit above

### 6. Generate the HTML asset viewer

Create a self-contained HTML file with:

**Design:**
- Clean, modern dark header with site name and stats
- Hero/banner section displayed as a 2-column grid with hover overlays
- Product grid with cards showing: image, brand name (small caps), product name
- Filter bar with brand/category pills
- Click-to-zoom lightbox with full image, metadata, and URL
- Copy-URL button on each card (clipboard icon, visible on hover)
- "Download All" button that lists all URLs for easy bulk download
- Responsive layout

**Technical:**
- All image URLs referenced directly (no downloading needed for the viewer)
- Product data embedded as a JavaScript array
- Brand filtering with active state
- Lightbox with Escape-to-close and click-outside-to-close
- Toast notification on URL copy

### 7. Optionally download images locally

After showing the viewer, ask the user if they want to download all images locally into the assets directory. If yes:

```bash
# Create subdirectories
mkdir -p <assets-dir>/banners <assets-dir>/products

# Download with curl, preserving filenames
curl -s -L -o <assets-dir>/products/<filename>.jpg "<url>"
```

### 8. Serve and open

Start a local HTTP server and tell the user the URL:

```bash
python3 -m http.server <port> --directory <assets-dir> &
```

Use a port in the 8800-8899 range. Check if the port is available first.

## Output

Tell the user:
1. How many images were found (by category)
2. The URL to open the viewer (e.g., `http://localhost:8850/assets.html`)
3. Where the files are saved
4. Ask if they want to download the images locally for offline use

## Important notes

- These images are for **prototyping only** — remind the user about copyright if they plan to use them in production
- If a site blocks scraping (403/captcha), suggest the user try a specific product page URL instead of the homepage
- Some sites lazy-load images via JavaScript — extract what's available from the HTML source and note if coverage seems low
- If the page has very few images in the HTML, suggest trying subpages (e.g., `/products`, `/shop`, `/collections`)
