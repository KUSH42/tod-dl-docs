# TOD-DL documentation

[Repository](../README.md) / Documentation

![TOD-DL: resumable acquisition, durable state, verifiable custody][banner]

TOD-DL combines resumable acquisition, durable recovery, and signed custody
records. These guides explain how to use the software and how to evaluate
its engineering. URL queues use the native engine. Protected resume requires
an authenticated, reread prefix and strong ETag or trusted-checksum protection.
The complete native acceptance matrix and operator pilot remain unverified.

## Find your starting point

Choose a guide by the question you need to answer.

| Question | Guide |
| --- | --- |
| What problem does the project solve? | [Project brief][ref-1] |
| What engineering decisions can I review? | [Design and evidence][ref-2] |
| How do I run, resume, and verify an acquisition? | [Operator guide][ref-3] |
| Which component owns each responsibility? | [Architecture](ARCHITECTURE.md) |
| How do I work on the code? | [Development](DEVELOPMENT.md) |
| How do signed records establish a verifiable history? | [Audit rail][ref-4] |
| How do bytes move into a verified handover? | [Chain of custody][ref-5] |
| Which specification owns a behavior? | [Specification guide][ref-6] |
| How do I style websites and documents? | [Visual identity][ref-7] |

## Read at three depths

The documentation offers three levels without requiring a full spec read.

1. Read the [project brief](PORTFOLIO-OVERVIEW.md) for the problem, design
   choices, and validation evidence.
2. Follow the [architecture](ARCHITECTURE.md) or [custody walkthrough]
   (CHAIN-OF-CUSTODY.md) for component boundaries and failure behavior.
3. Open the relevant [specification](SPECIFICATIONS.md), source file, and
   test when you need the exact contract.

## Understand the evidence

Guides explain the implementation. Specifications own behavior contracts.
Their `Status:` lines state implementation progress. Dated reports record
what a particular validation run checked.

A passing historical report does not establish that the current checkout
passes. A partially implemented specification can describe both available
behavior and planned extensions. [Open work](../specs/OPEN-WORK.md) names
remaining gaps; the linked spec defines the requirement. The
[native cutover contract](../specs/SPEC-native-engine-cutover.md) owns current
resume, retirement, and compatibility rules. Historical aria2 reports describe
the earlier engine. Non-active version 2 schemas require their matching
retained build.

## Next steps

Start with the [project brief](PORTFOLIO-OVERVIEW.md), or open the
[operator guide](OPERATOR-GUIDE.md) when you have an authorized case to run.

[ref-1]: PORTFOLIO-OVERVIEW.md
[ref-2]: PORTFOLIO-OVERVIEW.md#engineering-decisions
[ref-3]: OPERATOR-GUIDE.md
[ref-4]: AUDIT-RAIL.md
[ref-5]: CHAIN-OF-CUSTODY.md
[ref-6]: SPECIFICATIONS.md
[ref-7]: VISUAL-IDENTITY.md

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

[banner]: assets/tod-dl-banner.svg
