# Logo & Branding Refresh Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Swap the site's logo mark for the new `doja_logo.png` design everywhere it's used, and refresh the favicon files to match.

**Architecture:** This is a static-asset swap plus one CSS tweak — no application logic changes. The new logo replaces `src/assets/logo.png` in place (same path), so all five existing `<Image>` imports keep working unchanged. One CSS rule (`.footer-mark img` in the global stylesheet) gets a filter added so the new mark renders correctly on the dark footer background. The favicon is regenerated from the new source image.

**Tech Stack:** Astro 6 (`astro:assets` Image component), plain CSS (`src/styles/global.css`), ImageMagick (`magick` CLI) for favicon generation.

---

## Context for the engineer

Spec: `docs/superpowers/specs/2026-06-14-logo-branding-refresh-design.md`

Current state:
- `src/assets/logo.png` — 1024×1024, RGB **no alpha** (opaque white background), single-leaf "D" mark. Imported by 5 files via `import logoSrc from '.../assets/logo.png'`.
- `public/doja_logo.png` — 830×835, RGBA **with alpha** (transparent background), two-leaf "D" mark with stem. This is the new logo to adopt.
- `public/favicon.ico` — currently a 32×32 PNG-with-.ico-extension (not a real multi-res ICO).
- `public/favicon.svg` — unrelated abstract plant-icon design, not referenced anywhere in `<head>`.

Files that import the logo (no changes needed to any of these — they keep importing `../assets/logo.png` / `../../assets/logo.png`):
- `src/components/SiteHeader.astro`
- `src/components/SiteFooter.astro`
- `src/components/SiteFooterSimple.astro`
- `src/components/sections/SkuSection.astro`
- `src/layouts/BaseLayout.astro` (generates the favicon PNG at build time)

---

### Task 1: Replace the logo source asset

**Files:**
- Modify (binary): `src/assets/logo.png`
- Delete: `public/doja_logo.png`

- [ ] **Step 1: Confirm current file properties before changing anything**

Run:
```bash
sips -g pixelWidth -g pixelHeight -g hasAlpha src/assets/logo.png public/doja_logo.png
```
Expected output:
```
/Users/.../src/assets/logo.png
  pixelWidth: 1024
  pixelHeight: 1024
  hasAlpha: no
/Users/.../public/doja_logo.png
  pixelWidth: 830
  pixelHeight: 835
  hasAlpha: yes
```

- [ ] **Step 2: Overwrite `src/assets/logo.png` with the new logo's content**

Run:
```bash
cp public/doja_logo.png src/assets/logo.png
```

- [ ] **Step 3: Remove the now-redundant source file**

Run:
```bash
rm public/doja_logo.png
```

- [ ] **Step 4: Verify the replacement took effect**

Run:
```bash
sips -g pixelWidth -g pixelHeight -g hasAlpha src/assets/logo.png
file public/doja_logo.png
```
Expected: `src/assets/logo.png` now reports `pixelWidth: 830`, `pixelHeight: 835`, `hasAlpha: yes`. The `file` command on `public/doja_logo.png` should report `No such file or directory`.

- [ ] **Step 5: Commit**

`public/doja_logo.png` is currently untracked (per `git status`), so its
removal needs no `git add` — only the modified `src/assets/logo.png` is
staged.

```bash
git add src/assets/logo.png
git commit -m "$(cat <<'EOF'
Replace logo mark with new doja_logo design

Same path/filename so existing imports across header, footers,
SKU cards, and favicon generation pick up the new mark unchanged.
EOF
)"
```

---

### Task 2: Update the simple-footer mark filter

**Files:**
- Modify: `src/styles/global.css:1861-1864`

**Why:** `.footer-mark img` (used by `src/components/SiteFooterSimple.astro`) currently has no filter and sits on the dark `--forest-deep` (#2F4032) footer background. The new logo's dark green "D" body is close in tone to that background and would blend into it, leaving mostly the cream leaves visible. The full footer already solves this for `.footer-brand img` with `filter: brightness(0) invert(1); opacity: 0.85;` — apply the same treatment here for a white silhouette and visual consistency between the two footer variants.

- [ ] **Step 1: Edit the CSS rule**

Current content at `src/styles/global.css:1861-1864`:
```css
.footer-mark img {
  height: 48px;
  width: auto;
}
```

Change to:
```css
.footer-mark img {
  height: 48px;
  width: auto;
  filter: brightness(0) invert(1);
  opacity: 0.85;
}
```

- [ ] **Step 2: Commit**

```bash
git add src/styles/global.css
git commit -m "$(cat <<'EOF'
Render footer-simple logo as white silhouette on dark background

Matches the treatment already used by the full footer's
.footer-brand img, needed because the new logo's dark green body
is close in tone to the --forest-deep footer background.
EOF
)"
```

---

### Task 3: Regenerate the favicon and remove the unused SVG

**Files:**
- Modify (binary): `public/favicon.ico`
- Delete: `public/favicon.svg`

**Why:** `public/favicon.ico` is the file browsers request as a fallback (`/favicon.ico`) regardless of the `<link rel="icon">` tag in `BaseLayout.astro`, so it should reflect the new mark too. It's currently just a 32×32 PNG renamed to `.ico`; regenerate it as a proper multi-resolution ICO from the new logo. `public/favicon.svg` is an unrelated, unreferenced design and gets removed.

- [ ] **Step 1: Generate a multi-resolution favicon.ico from the new logo**

Run:
```bash
magick src/assets/logo.png -define icon:auto-resize=48,32,16 public/favicon.ico
```

- [ ] **Step 2: Verify the new favicon.ico**

Run:
```bash
magick identify public/favicon.ico
```
Expected output: three lines, one per resolution, e.g.
```
public/favicon.ico[0] ICO 48x48 48x48+0+0 8-bit sRGB ... B 0.000u 0:00.000
public/favicon.ico[1] ICO 32x32 32x32+0+0 8-bit sRGB ... B 0.000u 0:00.000
public/favicon.ico[2] ICO 16x16 16x16+0+0 8-bit sRGB ... B 0.000u 0:00.000
```

- [ ] **Step 3: Remove the unused favicon.svg**

Run:
```bash
rm public/favicon.svg
```

- [ ] **Step 4: Commit**

```bash
git add public/favicon.ico public/favicon.svg
git commit -m "$(cat <<'EOF'
Regenerate favicon.ico from new logo, drop unused favicon.svg

favicon.ico is the browser fallback at /favicon.ico and was just a
32x32 PNG renamed to .ico; now a proper multi-res ICO built from the
new logo. favicon.svg was an unrelated, unreferenced design.
EOF
)"
```

---

### Task 4: Build and visually verify

**Files:** none (verification only)

- [ ] **Step 1: Run a production build**

Run:
```bash
npm run build
```
Expected: build succeeds with no errors. This exercises `BaseLayout.astro`'s
`getImage({ src: logoImg, width: 64, height: 64, format: 'png' })` call against
the new logo and confirms `astro:assets` can process the new PNG.

- [ ] **Step 2: Confirm the generated favicon and logo assets look correct**

Run:
```bash
find dist/_astro -iname "*logo*"
```
Expected: at least one generated PNG file (the favicon derived from the new logo).

- [ ] **Step 3: Start the dev server and visually inspect in a browser**

Run:
```bash
npm run dev
```
Open `http://localhost:4321/` (or the port Astro reports) in a browser and check:
- Header (top-left): new two-leaf "D" mark, ~38px tall, no visible white box around it.
- SKU section cards: new mark at ~80px, blends with the cream/beige/gold gradient card background (no white box).
- Full footer (bottom of homepage, dark green background): mark renders as a white silhouette (~50px).
- If `SiteFooterSimple` is used on any page (check `src/pages/*.astro` for which layout/footer each page uses), confirm its mark also renders as a white silhouette (~48px) on the dark background.
- Browser tab favicon shows the new mark (may require a hard refresh / cache clear).

- [ ] **Step 4: Fix any issues found, then commit if changes were made**

If everything looks correct, no further action needed — Task 4 is verification-only. If you made any fixes, stage and commit them with a descriptive message following the same format as the commits above.

---

## Final check

- [ ] `git status` is clean (or only contains intentional changes from Task 4 fixes).
- [ ] `git log --oneline -4` shows the three commits from Tasks 1–3 (plus any Task 4 fix commit).
