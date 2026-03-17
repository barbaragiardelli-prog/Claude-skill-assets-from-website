---
description: "Scrape product images and assets from any website URL. Downloads images directly into your project's assets folder — ready to use in prototypes, portable on any device."
---

# Scrape Assets from Website

You are an asset scraper. The user wants to extract high-quality images from a website and add them directly into the current project's asset folder so they're ready to use in prototypes.

## Input

The user provides: **$ARGUMENTS**

This should be a website URL (e.g., `https://www.sephora.com/`). If no URL is provided, ask the user for one.

## Steps

### 1. Find the project's asset folder

Look at the current working directory and detect the project's existing asset/image folder. Check for these common patterns (in order of priority):

```
public/images/
public/assets/
src/assets/images/
src/assets/
src/images/
assets/images/
assets/
static/images/
static/assets/
img/
images/
```

Also check the project's code (imports, `<img>` tags, CSS `url()`) to see where it loads images from.

**If a matching folder is found:** confirm with the user before writing to it:
> "I found your project's asset folder at `public/images/`. I'll create a `scraped/<domain>/` subfolder there with banners and product images. OK to proceed?"

**If no project folder is found:** ask the user where they want the assets:
> "I don't see an existing asset folder in this project. Where should I put the scraped images? Some options:
> 1. `assets/` (I'll create it)
> 2. A custom path you specify"

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
- **Other** — anything that doesn't fit above

### 6. Download ALL images into the project

Download every image directly into the project's asset folder. Structure:

```
<project-asset-folder>/
  scraped/
    <domain-name>/
      banners/
        banner-korean-skincare-launch.jpg
        banner-march-minis.jpg
      products/
        rare-beauty-soft-pinch-liquid-blush.jpg
        dior-jadore-parfum.jpg
        glossier-boy-brow.jpg
      lifestyle/
        ...
      assets.html          <-- the viewer
      manifest.json        <-- metadata for all images
```

**Download command:**
```bash
curl -s -L -o "<path>/<clean-filename>.jpg" \
  -H "User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36" \
  "<url>"
```

**Naming convention:**
- Products: `<brand>-<short-product-name>.jpg` (lowercase, hyphens, no special chars)
- Banners: `banner-<descriptive-name>.jpg`

Run downloads in parallel (batches of 10 with `&` and `wait`).

### 7. Generate manifest.json

Create a `manifest.json` in the scraped folder so the project can programmatically use the assets:

```json
{
  "source": "https://www.sephora.com/",
  "scraped_at": "<ISO timestamp>",
  "banners": [
    { "file": "banners/banner-korean-skincare.jpg", "label": "Korean Skincare Launch" }
  ],
  "products": [
    { "file": "products/rare-beauty-soft-pinch-liquid-blush.jpg", "brand": "Rare Beauty", "name": "Soft Pinch Liquid Blush" }
  ]
}
```

### 8. Generate the HTML asset viewer

Create `assets.html` in the scraped folder using **local relative paths**.

**Design:**
- Clean, modern dark header with site name and stats
- Hero/banner section as a 2-column grid with hover overlays
- Product grid with cards: image, brand name (small caps), product name
- Filter bar with brand/category pills
- Click-to-zoom lightbox with full image, metadata, and relative file path
- Responsive layout (works on phones and desktops)
- Copy-path button on each card so the user can quickly grab the relative path for their code

**Technical — IMPORTANT:**
- All `<img src="...">` must use LOCAL relative paths (e.g., `products/rare-beauty-soft-pinch-liquid-blush.jpg`)
- The HTML must work by simply double-clicking it — NO server needed
- Include a section at the top showing the import paths relative to the project root, so the user can copy-paste them into their code

### 9. Verify downloads

```bash
find <assets-dir> -name "*.jpg" -size -1k
```

Remove any failed downloads. Report how many succeeded vs failed.

## Output

Tell the user:
1. How many images were downloaded (by category)
2. Where they were saved (relative to the project root)
3. That they can open `assets.html` to browse everything
4. Show a few example import paths they can use in their code, e.g.:
   - `<img src="./assets/scraped/sephora.com/products/rare-beauty-soft-pinch-liquid-blush.jpg">`
   - `background-image: url('./assets/scraped/sephora.com/banners/banner-korean-skincare.jpg')`
5. That the whole project is portable — anyone who gets the project has the assets

## Important notes

- **ALWAYS ask for confirmation** before writing into the project's asset folder
- **ALWAYS download images locally** — never rely on external URLs
- These images are for **prototyping only** — remind the user about copyright for production use
- If a site blocks scraping (403/captcha), suggest trying a specific product page URL
- If few images are found, suggest subpages (e.g., `/products`, `/shop`, `/collections`)
