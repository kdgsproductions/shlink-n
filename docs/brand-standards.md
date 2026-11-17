# Mikkle brand standards

## Approved identity

1. Product name: **Mikkle** (M-I-K-K-L-E).
2. Tagline: **Long Story Short**; the seal uses uppercase **LONG STORY SHORT**.
3. The owner's uploaded logo image supersedes the earlier SVG package and generated preview. It shows a green/gold circular seal, widely spaced arc lettering, two side dots, a larger black QR pattern, and a gold browser shape with a diagonal right edge. Do not substitute the rejected rendering or infer replacement artwork.
4. The owner-uploaded logo is available unchanged at `public/assets/brand/mikkle-logo.png` ([GitHub asset](https://github.com/kdgsproductions/shlink-n/blob/develop/public/assets/brand/mikkle-logo.png)). This PNG is the approved logo asset; the rejected SVG/preview is excluded.
5. Product QR generation remains governed by the roadmap; the brand mark is not a replacement for generated product QR codes.

## Typography

1. Montserrat is the approved default family, self-hosted in regular 400, semibold 600, and bold 700 weights.
2. App fallback stack: `'Montserrat', system-ui, sans-serif`.
3. Declare each face once in `public/assets/css/base.css`. Both current HTML error templates load that stylesheet. Future app layouts must load it too.
4. Fonts are downloaded from the [Google Fonts Montserrat source repository](https://github.com/googlefonts/Montserrat/tree/master/fonts/ttf) as static TTF and converted locally to WOFF2 with owner approval. No font CDN/API is used.
5. Preserve `public/assets/fonts/OFL.txt` verbatim. Source blob hashes and output checksums are recorded in `public/assets/fonts/sources.json`.
6. Any later approved editable SVG lettering must specify Montserrat. Renaming a font family alone does not embed a font; verify standalone asset rendering before delivery.

## Palette

1. Green: `#14532D`, primary brand color from the supplied brand package.
2. Gold: `#D4AF37`, trims, lettering, and accents.
3. Ink: `#111111`, body text and the abstract QR mark.
4. White: `#FFFFFF`, field and gaps.
5. Red: `#C62828`, destructive/critical UI accent; absent from the logo.
6. Preserve the uploaded logo image unchanged when received; do not recolor its pixels to force these UI tokens.

## Usage

1. Do not stretch, rotate, or alter the owner's approved logo image.
2. Verify contrast before using palette combinations for functional UI text.
3. Monochrome variants and a mark-only/favicon package are not included in this typography commit.

## Verification status

1. WOFF2 signatures, expected weights, source blob hashes, output checksums, and retained glyph counts were checked.
2. Chromium checks passed for both current HTML error templates: Montserrat is the computed body font and all three local weights load. No external requests, HTTP failures, or browser script errors occurred.
3. Whitespace checks passed for authored changes. OFL.txt is preserved verbatim, including upstream whitespace.
4. Backend PHPUnit suites were not run for this asset/template typography change; this verification does not establish the application baseline or complete Phase 0.
