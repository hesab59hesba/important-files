# Ready Recipe: Turn Any Logo into HTML and Export PNGs

A reusable, brand-neutral workflow for recreating any logo as a standalone HTML
preview and exporting tightly cropped transparent and white-background PNG
versions. It covers logos made from text and CSS, SVG artwork, raster images,
or a combination of these.

Use this only for logos and assets you own or have permission to reuse.

## What you will make

1. A standalone HTML file that renders the logo without the rest of the site.
2. A transparent PNG with only the logo artwork.
3. A white-background PNG with the same artwork and crop.

Keep the HTML as the editable source of truth. Export both PNGs from that same
HTML so their artwork, proportions, and dimensions stay consistent.

## 1. Inspect the original logo

Open the source at the size where the logo normally appears. This could be a
website, a design file, a supplied image, or a brand guide. Record:

- The exact text, capitalization, punctuation, and symbol.
- The visual order of symbol and wordmark.
- The logo's direction (`ltr` or `rtl`) and its position in the site.
- Font family, weight, size, line height, and letter spacing.
- Text colors, gradients, borders, radii, shadows, and spacing.
- Any image, SVG, icon font, or other asset used for the mark.
- Responsive changes at narrow screen widths.

For a website, use the browser inspector's **Computed** styles to find final
values. Follow CSS custom properties to their definitions, and check inherited
styles on the page, header, and links. A logo can look different when copied
alone if it depended on those inherited styles. For a supplied asset, use its
intrinsic proportions and transparency as the source of truth.

If the website uses an existing SVG or image file for the logo, use that
original asset whenever possible instead of redrawing it. If it is text plus a
separate mark, reproduce both using their actual assets, font, and CSS values.
Do not approximate the font or symbol if a faithful source is available.

## 2. Make a standalone HTML preview

Create a small HTML file with all required styles and dependencies included.
For a website logo, copy only the logo-related declarations and the
tokens/assets/fonts they rely on; do not copy the whole site's CSS bundle.
For an existing SVG or image, embed or reference that asset and preserve its
aspect ratio. For a text-based logo, use the real font and text rather than
substituting an approximate font or drawing. Include any separate symbol or
wordmark parts in their original order.

This neutral structure is a starting point for any logo. Replace the
placeholder content and styles with that logo's own artwork, dimensions,
spacing, and typography. For image/SVG-only artwork, the wrapper may contain a
single `<img>` instead of separate mark and word elements.

```html
<!DOCTYPE html>
<html lang="en" dir="ltr">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Logo preview</title>
  <!-- Load the original font here if the logo uses a web font. -->
  <style>
    * {
      box-sizing: border-box;
    }

    html,
    body {
      min-height: 100%;
      margin: 0;
    }

    body {
      min-height: 100vh;
      display: grid;
      place-items: center;
      background: #eeeeee; /* Preview canvas only; choose any preview color. */
    }

    #logo {
      display: inline-flex;
      align-items: center;
      gap: 1rem; /* Replace with this logo's measured spacing. */
      /* Add this logo's own colors, font, and other visual styling. */
      text-decoration: none;
    }

    #logo .mark {
      display: grid;
      place-items: center;
      /* Use the original mark asset and dimensions, if there is a separate mark. */
    }

    #logo .wordmark {
      /* Use the original font, size, weight, color, and direction, if needed. */
    }
  </style>
</head>
<body>
  <div id="logo" role="img" aria-label="Describe the logo">
    <span class="mark" aria-hidden="true">Replace with mark or asset</span>
    <span class="wordmark">Replace with exact wordmark</span>
  </div>
</body>
</html>
```

The values and sample text above are placeholders, not universal logo
settings. For an existing image or SVG, use markup such as
`<img src="logo.svg" alt="Describe the logo">` inside `#logo` instead of
rebuilding the artwork. Keep the file with its referenced assets, or inline
the SVG so the HTML is self-contained. Set document language and direction
appropriately; do not force RTL or LTR unless that is how the original logo is
composed. If it is not a website logo, use its supplied vector/image, font
files, or brand guide as the reference rather than inventing missing details.

Open the standalone file in a browser and compare it side by side with the
source. For a website, check at the same viewport size and wait for external
fonts to load. Avoid a blank online sandbox unless you also include all
required fonts, assets, and styles; it will not inherit the original site's
design system.

## 3. Choose and lock the export size

Decide what output size the logo needs before exporting. For a faithful
one-to-one capture, render at CSS scale `1` / device scale factor `1`, then
capture the logo element's bounds. If a specific pixel width is required,
choose it deliberately and scale the rendered logo proportionally. Use the
same chosen scale for both background variants.

For repeatable output, keep the browser viewport, device scale factor, font
loading, responsive breakpoint, and HTML/CSS fixed between exports. Record the
final PNG dimensions. The logo should be cropped to its own bounds, not the
full browser window; a little intentional transparent padding is acceptable
only if you choose it consistently.

## 4. Export the transparent PNG

Use a browser automation tool such as Playwright. Start from the rendered
standalone page, wait for fonts, and screenshot the logo element itself. The
example below assumes a browser automation page variable named `page`, the
logo wrapper `#logo`, and an output path you choose:

```js
await page.evaluate(() => document.fonts.ready);

const logo = page.locator("#logo");
await logo.screenshot({
  path: "path/to/logo-transparent.png",
  omitBackground: true,
  animations: "disabled"
});
```

The screenshot is of the logo element only, not the preview page. Make the
wrapper's background transparent for this export. Preserve intended
backgrounds that are part of the artwork itself, such as a colored icon tile.

## 5. Export the white-background PNG

Use the same page, font, viewport, scale, and crop. Temporarily make the logo
element's rectangular export area white, then capture exactly its measured
bounds. Rounding the crop outward to whole pixels avoids clipping antialiased
edges:

```js
await page.evaluate(() => document.fonts.ready);

const logo = page.locator("#logo");
await logo.evaluate(element => {
  element.style.backgroundColor = "#ffffff";
});

const bounds = await logo.boundingBox();
const x = Math.floor(bounds.x);
const y = Math.floor(bounds.y);
const clip = {
  x,
  y,
  width: Math.ceil(bounds.x + bounds.width) - x,
  height: Math.ceil(bounds.y + bounds.height) - y
};

await page.screenshot({
  path: "path/to/logo-white.png",
  clip,
  animations: "disabled"
});
```

If the logo already has a background that must remain part of its artwork,
include that in both exports. The white version should have white in the
surrounding crop area, without changing the logo's own colors.

## 6. Verify both files

Before using the exports, confirm:

- Both PNGs show the same logo at the same scale and orientation.
- Their pixel dimensions match.
- Neither export includes the browser page, header, selection handles, or
  unwanted whitespace.
- No text, symbol, shadow, or antialiased edge is clipped.
- The transparent version has alpha transparency outside the artwork.
- The white version is opaque white (`#ffffff`) outside the artwork.
- The intended font loaded; a fallback font can change the wordmark's width.

If the font did not load, check the network and font URL, wait for
`document.fonts.ready`, and export again. If the crop differs, compare the
element bounds and browser scale before manually changing the logo CSS.

## 7. Keep the deliverables organized

Save the HTML source and exports together, using names that clearly distinguish
background variants:

```text
brand-logo/
  logo.html
  brand-logo-transparent.png
  brand-logo-white.png
```

When the source logo changes, update and compare the standalone HTML first,
then regenerate both PNGs from the same finished page. Do not edit one PNG
independently of the other; that can make the variants inconsistent.
