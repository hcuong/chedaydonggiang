# Logo & Branding Refresh — Design

## Context

The site ("Chè Dây Đông Giang") currently uses `src/assets/logo.png` (1024×1024,
opaque white background) as its logo mark — a dark green "D"-shaped icon
containing a single cream leaf. A new, more refined mark has been provided at
`public/doja_logo.png` (830×835, transparent background) — a "D"-shaped icon
containing two cream leaves and a stem. Its green/cream tones match the
existing palette (`--forest-deep: #2F4032`, `--cream: #F9F6F0`).

This is a visual refresh only: brand name ("Chè Dây Đông Giang"), copy, and
color palette remain unchanged.

## Current logo usage

| Location | File | Usage |
|---|---|---|
| `src/components/SiteHeader.astro` | `.brand img` | height 38px, no filter, on `rgba(245,241,232,.95)` bg |
| `src/components/SiteFooter.astro` | `.footer-brand img` | height 50px, `filter: brightness(0) invert(1); opacity:.85`, on `--forest-deep` bg |
| `src/components/SiteFooterSimple.astro` | `.footer-mark img` | height 48px, no filter, on `--forest-deep` bg |
| `src/components/sections/SkuSection.astro` | `.sku-image img` | height 80px, decorative (`alt=""`), on cream/beige/gold gradient bg |
| `src/layouts/BaseLayout.astro` | favicon | `getImage({ src: logoImg, width: 64, height: 64, format: 'png' })` → `<link rel="icon" type="image/png">` |

All five locations import the same file: `import logoSrc from '.../assets/logo.png'`.

## Part 1: Logo asset swap

1. **Replace `src/assets/logo.png` in place** with the image content of
   `public/doja_logo.png` (same filename/path). All five existing imports keep
   working unchanged — this is a pure asset swap with no import/code changes
   in the consuming files.
2. **Remove `public/doja_logo.png`** once its content has been migrated into
   `src/assets/logo.png` (avoids a redundant duplicate file in the repo).
3. **Update `SiteFooterSimple.astro`'s `.footer-mark img` CSS** to add
   `filter: brightness(0) invert(1); opacity: .85;`, matching the treatment
   already used by `.footer-brand img` in the full footer. This is necessary
   because:
   - `.footer-mark img` currently has no filter and sits on the same dark
     `--forest-deep` (#2F4032) background as the full footer.
   - The new logo's dark green "D" body is close in tone to `--forest-deep`,
     so without a filter it would blend into the background, leaving mostly
     the cream leaves visible.
   - Applying the same white-silhouette filter as the full footer keeps both
     footer variants visually consistent and ensures good contrast.

### Net visual effect per location

- **Header** (38px): transparent background blends naturally with the
  near-white header background — improvement over the old opaque white
  square. No code change.
- **SKU cards** (80px): transparent background blends naturally with the
  cream/beige/gold gradient — improvement over the old opaque white square.
  No code change.
- **Full footer** (50px): the existing `brightness(0) invert(1)` filter
  previously turned the *entire* opaque 1024×1024 old logo into a solid white
  square (since `brightness(0)` ignores alpha and the old logo had no alpha
  channel) — a latent display bug. With the new transparent-background logo,
  this filter will now correctly render a white "D + leaves" silhouette. No
  code change needed; this is fixed as a side effect of the asset swap.
- **Simple footer** (48px): gains the same white-silhouette filter (see Part
  1, step 3) for contrast and consistency with the full footer.

## Part 2: Favicon refresh

1. **`BaseLayout.astro`**: no code change. Its build-time
   `getImage({ src: logoImg, width: 64, height: 64, format: 'png' })` call
   automatically uses the new logo once `src/assets/logo.png` is replaced,
   producing a favicon with transparent corners around the "D" mark.
2. **`public/favicon.ico`**: regenerate as a multi-resolution ICO (16/32/48px)
   from the new logo using ImageMagick, replacing the current file. This is
   the file browsers request as a fallback (`/favicon.ico`) regardless of
   `<link rel="icon">` tags.
3. **`public/favicon.svg`**: delete. It is an unrelated hand-drawn abstract
   plant-icon design, not referenced in `<head>`, and visually inconsistent
   with the refreshed branding.

## Out of scope

- Brand name, site copy, and color palette are unchanged.
- No Open Graph / social meta images exist currently; none are added as part
  of this change.
- No changes to image dimensions/aspect ratios in layout CSS — the new logo
  (830×835) is close enough to square (like the old 1024×1024) that existing
  height-based sizing (38–80px) requires no adjustment.

## Testing

- `astro dev`: visually verify the logo renders correctly (transparent
  background, correct silhouette colors) in the header, both footer variants,
  and SKU cards.
- `astro build`: verify the build-time favicon generation succeeds and the
  generated favicon reflects the new logo.
- Verify `public/favicon.ico` opens correctly and shows the new mark at small
  sizes.
