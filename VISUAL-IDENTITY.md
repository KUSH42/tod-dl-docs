# TOD-DL visual identity: Signal

[Repository](../VISUAL-IDENTITY.md) / Visual Identity

![TOD-DL: resumable acquisition, durable state, verifiable custody][banner]

Signal defines the TOD-DL identity for websites, documentation, and readable
HTML/PDF artifacts. Warm paper, dark type, a vermilion field, and the capture
mark connect these formats. Use the same identity at each reading density.

## Scope and sources

This guide owns the visual rules for future product materials. It does not
change acquisition behavior, specification status, or console requirements.
The [console specification](../specs/SPEC-console-visual-style.md) owns the
terminal interface.

The supplied Signal motion study and desktop image establish the identity.
The following assets have distinct roles:

- [Banner](assets/tod-dl-banner.svg): reusable static identity and canonical
  capture-mark geometry, in the `capture-mark` group.
- [Signal motion source](assets/acquisition-motion-signal.html): screen
  layout and animation choreography.
- `.github/scripts/build-html-pdf.js` in the source repository: specification
  renderer with the complete static identity panel embedded for offline use.

The reference establishes the palette, typography, and motion values below.
Minimum sizes, clear space, and reading widths are recommendations derived
from the reference. They are not measurements from an existing acceptance
test. Font appearance depends on the installed Arial or Helvetica font.

## Identity principles

The identity combines a technical drawing with a strong editorial hierarchy.
Use these principles together:

- Put the product name and a concrete statement before decorative detail.
- Use open frames, square corners, thin rules, and directional arrows.
- Reserve the vermilion field for identity panels and major introductions.
- Use paper for reading surfaces and dark ink for most text.
- Use numbered stages, figure labels, and aligned metadata for structure.
- Explain acquisition, preservation, and verification with bounded diagrams.

Avoid gradients, shadows, rounded cards, glass effects, and stock security
symbols. Do not add shields, locks, or badges that imply verified safety.
The capture mark and the type hierarchy provide the product identity.

## Color

Use the motion study's semantic names in new stylesheets. This table defines
palette roles; existing asset values remain in their source files.

| Name | Reference value | Role |
| --- | --- | --- |
| Paper | `#f2f0e9` | Main reading background |
| Ink | `#191a19` | Text, capture mark, and primary rules |
| Muted | `#585952` | Supporting text and metadata |
| Line | `#bebbb1` | Quiet dividers and construction guides |
| Accent | `#b52c18` | Active stage, terminal dot, and emphasis |
| Signal | `#e64b32` | Large identity field behind dark type |
| Panel | `#e9e6de` | Diagram and figure surface |

Pair ink with paper, panel, or signal. Pair muted and accent text with paper.
Do not put small white text on signal. Do not use signal as body text on
paper. Keep accent text off the signal field.

Color must accompany a label, position, shape, or state attribute. An active
stage needs more than a red heading. Use a visible marker and an accessible
state such as `aria-pressed` for interactive stage selectors.

The artifact currently reverses the names `--ink` and `--paper`: its ink
variable contains the light background. Preserve rendered colors when editing
that artifact. Use the semantic names in this guide for new work.

## Typography

Use `Arial, Helvetica, sans-serif` for display and reading text. The label
class `.mono` in the motion reference also uses this sans-serif family.
Its name does not mean that metadata uses a monospace font.

| Role | Reference treatment |
| --- | --- |
| Stacked wordmark | Weight 900; line height 0.85; tracking -0.085em |
| Website headline | Weight 800; line height 0.96; tracking -0.065em |
| Section headline | Weight 800; line height 1.05; tracking -0.055em |
| Stage title | Weight 800; 32px; tracking -1.3px |
| Website introduction | 18px; line height 1.5 |
| Explanation | 14px; line height 1.7 |
| Diagram or stage copy | 13px; line height 1.6 |
| Technical label | 10px; weight 700; uppercase; tracking 1.3px |

The desktop headline uses `clamp(62px, 7.7vw, 110px)`. At narrow widths, the
reference uses `clamp(58px, 13.5vw, 85px)`. These sizes belong to short display
headlines. Use ordinary heading sizes for long technical titles.

Recommend 16–18px body text for new web documentation. The specification
artifact uses 13px body text with a 1.65 line height for dense print reading.
Use monospace for code, commands, paths, and digests. Preserve complete values
when users need to copy or verify them. Use tabular numerals for aligned counts.

## Logo system

The identity has a compact wordmark, a display wordmark, and a capture mark.
Use each form for its intended space.

The compact wordmark reads `TOD–DL`, followed by a separate accent dot. Use it
in navigation, document headers, and small identity areas. Accessible text and
plain-text references use `TOD-DL`.

The display wordmark stacks `TOD` above `DL`. Keep the tight spacing and heavy
weight. On narrow screens, the motion reference places the letters on one line.
Do not add a second arrow beside `DL` when the capture mark is present.

The capture mark contains three open frames and a southeast arrow. Its source
coordinate system is 400 × 400. The outer, middle, and inner frame strokes are
24, 18, and 12 units. The arrow stroke is 28 units. Frames have square caps and
miter joins; the arrow has butt caps. Preserve path coordinates and stroke
ratios from the banner's `capture-mark` group. Scale the whole group uniformly.

Recommend a square canvas of at least 48px for the isolated mark. Below that
size, use the compact wordmark until a separate small-size mark is designed
and checked. Recommend clear space equal to one arrow stroke around visible
geometry. The full source canvas provides more space than that minimum.

Render the mark in ink on paper or signal. A monochrome print version can use
black on white. Construction lines and registration crosses are optional
illustration detail; omit them in a small logo. Preserve the open frames.
Do not rotate, stretch, round, outline, or close the mark's frames.

## Layout and spacing

Use a left-aligned page with generous margins and horizontal rules. The screen
reference has a 1440px maximum shell width and 6% side padding.

The identity panel divides into a wordmark and a capture-mark area. The
reference uses a 0.85:1.15 column ratio, a 30px gap, and 30px × 36px padding.
The introduction uses a 1.4:1 ratio. The diagram and stage selector use 1.2:1.
Use these ratios as starting points, then preserve the reading order.

Use 16, 24, 32, and 48px spacing for new component layouts. Use 1px quiet
rules, 2–3px section rules, and a 6px masthead rule. Keep square corners.
Align captions, figure rules, and body text to the same column edges.

At 900px, reduce gaps and display sizes. At 640px, stack the identity,
introduction, process, and explanatory columns. Keep the diagram square.
Recommend a 60–75 character reading width for web documentation. Do not force
long prose to span the full display grid.

## Components and illustrations

Components use typography and rules to show hierarchy. Keep these treatments
consistent across a website and documentation.

- Masthead: compact wordmark at left; navigation and edition label at right.
- Introduction: uppercase context label; large headline; short explanation.
- Figure: tinted panel; numbered caption; thin top and bottom rules.
- Stage selector: number, bold title, question, and short explanation.
- Text action: underlined label with a directional arrow and visible focus.
- Motion control: square border with a text alternative for the symbol.
- Footer: thin rule, series label, and scope statement.

Use synthetic points, open frames, arrows, and linked records for conceptual
illustrations. Label synthetic data directly. Treat guide lines as secondary
information. Use consistent line weights within each diagram.

The identity line is “Acquire / Preserve / Verify.” The motion study labels
its explanatory stages “Acquire / Verify / Trace.” These have different roles:
the first states product principles; the second follows the illustration.
Do not change operational stage names to match a decorative sequence.

## Motion and accessibility

Motion explains sequence. The capture mark draws its frames, then its arrow,
over 4.8 seconds. The separate process study spans 18 seconds. Reference
choreography remains owned by the motion HTML; reuse that implementation
instead of reconstructing timings from screenshots.

Provide pause and replay controls. Pause motion when the page is hidden or the
illustration leaves view. Respect `prefers-reduced-motion`; show a complete
static mark on initial load when reduced motion is selected. Keep meaningful
content available without JavaScript. Print exports always use static artwork.

Provide a descriptive accessible name for an isolated logo. Mark repeated
artwork as decorative when nearby text already names the product. Use semantic
headings, visible keyboard focus, and text labels for control icons. The
reference uses a 3px focus outline and 44px control dimensions.

Recommend at least 4.5:1 contrast for normal text and 3:1 for large text and
meaningful control boundaries. These are implementation checks, not a claim
that every existing reference component has passed an accessibility audit.

## Apply the identity to each format

A website can use the full identity field and motion study. Documentation can
use a compact masthead, restrained accent, numbered figures, and readable
prose. A specification cover can use the stacked wordmark and static capture
mark while keeping dates, scope, and status in separate metadata.

Keep specification Markdown focused on requirements. Apply visual styling in
the HTML/PDF renderer. Do not add presentation markup to every specification.
Preserve page numbering, internal links, selectable code, table headers, and
safe page breaks. Do not let a large heading obscure a requirement.

The source repository's `.github/scripts/artifact-designs.js` reads the
complete identity panel from `docs/assets/tod-dl-banner.svg`. The HTML/PDF
builder embeds the panel for offline use. Its tests compare the emitted cover
with the canonical banner. Build instructions belong to
`.github/scripts/README.md` in the source repository.

Use the banner as an image for ordinary documents. Inline the canonical group
only when styling or a self-contained export requires it. Do not derive a
production logo from a screenshot. Regenerate the Signal PDF
from its HTML whenever the embedded mark changes.

## Review new materials

Check the final rendered format before publishing it.

1. Check the logo geometry against the banner.
2. Check color roles, type hierarchy, alignment, and rule weights.
3. Check narrow screens, keyboard focus, contrast, and reduced motion.
4. Check printed pages, links, diagrams, code wrapping, and table breaks.
5. Check that illustrations use synthetic data and label their scope.
6. Check that text separates byte integrity, record authentication, and source
   authenticity. Do not imply completed acceptance from a brand illustration.

   [banner]: assets/tod-dl-banner.svg
