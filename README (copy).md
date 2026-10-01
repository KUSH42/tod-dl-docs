# Specification artifacts

`build-html-pdf.js` owns the HTML and PDF presentation for this renderer.
It reads sorted `SPEC*.md` files and writes `output.html` and
`combined_output.pdf`. The PDF includes bookmarks and linked contents with
page numbers taken from the rendered PDF destinations.

## Select a design

Change one string near the top of `build-html-pdf.js`, then rebuild:

```js
const DESIGN = 'signal';
```

Choose one of these names. `signal` is the default.

| Name | Design |
| --- | --- |
| `signal` | Vermilion artwork, oversized type, and an editorial grid. |
| `atlas` | Midnight blue, cyan diagrams, and technical typography. |
| `folio` | Warm paper, serif titles, and engraved linework. |
| `copper` | The preserved ink-and-copper design from the first revision. |

The selection applies to the cover, contents, specification pages, syntax
colors, and diagrams. `artifact-designs.js` owns the palettes, cover artwork,
and design-specific styles. The build script owns shared layout and page
geometry. An unknown design name fails the build.

## Build

Use the Node version required by `package.json`. Install the locked packages
from this directory. Puppeteer installs its matching browser during setup.

```bash
npm ci
```

Pass the specification directory and an output directory outside the source
repository. Existing artifacts with the same names are replaced after a
successful render.

```bash
node build-html-pdf.js ../../specs /tmp/tod-dl-artifacts
```

Without arguments, the script reads and writes the current directory. The
saved HTML embeds syntax colors and rendered Mermaid diagrams. Its contents
numbers refer to the companion PDF, including the cover and contents pages.
Implementation status comes from each source specification.

## Preview and check

Run the renderer tests. Every design uses the same synthetic specifications
and temporary paths. The tests check page numbers, heading links, cover fit,
offline diagrams, and mobile width. No downloader process or source request
runs.

```bash
npm test
```

Use Poppler's `pdftoppm` to make PNG previews from the actual PDF pages.
Install Poppler separately if the command is unavailable.

```bash
pdftoppm -f 1 -l 2 -scale-to 1600 -png \
  /tmp/tod-dl-artifacts/combined_output.pdf /tmp/tod-dl-artifacts/preview
```

The existing GIF scripts can consume the same artifact names. Run those
scripts from the output directory with their existing prerequisites.

## Preserve print geometry

`build-html-pdf.js` owns the page geometry and print options. Keep its
Puppeteer margins at zero and its header/footer insertion through CSS page
margin boxes. The separate CSS page margins must also remain unchanged.

The contents reserve a fixed column for page numbers. The build renders the
PDF, fills the numbers, and renders again. It writes artifacts only after the
page destinations stabilize. A missing destination or unstable pagination
fails the build.

`../../specs/build-pdf.js` is a separate copy of the older renderer. Use
`build-html-pdf.js` for this design; the older copy does not share its styles.
