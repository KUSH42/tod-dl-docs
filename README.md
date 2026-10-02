# TOD-DL

[Repository](https://github.com/KUSH42/tod-dl-docs) / Documentation

![TOD-DL: Acquire. Preserve. Verify.][banner]

**Bounded native acquisition with durable recovery and signed provenance.**

TOD-DL stands for *Tor Onion Dump Downloader*. The project acquires a bounded
set of files from unreliable sources, including Tor onion services. It
preserves existing final files, recovers interrupted work, and records signed
provenance for independent verification.

This public repository presents the project through engineering guides,
visual demonstrations, and selected specification artifacts. The source code
and complete specification set remain private. The material here explains
project design and scope; it does not provide an installable downloader.

[Project brief][brief] · [Architecture][architecture] ·
[Operator guide][operate] · [Specification demo][demo-pdf]

## The engineering problem

A successful HTTP response does not finish an acquisition. The controller
must also handle a changed remote file, a full disk, a process crash, and an
existing destination. A reviewer needs to distinguish retained bytes,
recorded observations, and verified claims.

TOD-DL addresses these conditions through explicit boundaries:

- **Bounded scope.** A run keeps its selected set across retries and resume.
- **Durable recovery.** SQLite records state transitions and promotion intent.
  Recovery reconciles interrupted work with the filesystem.
- **Evidence preservation.** Exclusive final-file creation protects existing
  files. Conflicts retain incoming bytes for review.
- **Verifiable history.** Signed records connect sessions, outcomes, retained
  inputs, and final-file digests.
- **Separate observation and control.** The console reads published telemetry.
  Confirmed commands reach the acquisition controller through a local socket.
- **Independent review.** Read-only verification checks retained artifacts
  against external trust material.

The [architecture guide][architecture] explains component boundaries. The
[chain of custody guide][custody] follows acquired bytes through finalization
and verification.

## Explore the project

Choose the guide that answers your question.

| Goal | Read |
| --- | --- |
| Understand the problem and project scope | [Project brief][brief] |
| Review engineering tradeoffs | [Design decisions][decisions] |
| Follow acquisition and recovery | [Operator guide][operate] |
| Understand component responsibilities | [Architecture][architecture] |
| Assess signed records and trust limits | [Audit rail][audit] |
| Follow bytes into a verified handover | [Chain of custody][custody] |
| Understand the development approach | [Development guide][develop] |
| Explore the contract structure | [Specification guide][specs] |
| Review the visual system | [Visual identity][identity] |

The guides include references to private source files, tests, specifications,
and reports. Those references describe the engineering context; their targets
are not included in this public repository.

## View the demonstrations

The visual demonstrations use synthetic content. They illustrate the design
and do not establish source authenticity or acquisition acceptance.

![Synthetic points form a file and linked provenance records.][motion]

Open the [interactive acquisition illustration][motion-page] or the
[static illustration][motion-still]. Download the interactive HTML and open
it in a browser to use its controls.

The specification demo contains **five specifications**, rather than the
complete private set. It presents the document design, linked contents,
rendered diagrams, and PDF navigation.

- [Download the five-specification PDF][demo-pdf].
- [Download the companion HTML][demo-html] and open it in a browser.
- [View the project presentation HTML][presentation] by downloading the file
  and opening it in a browser.

## Current scope and evidence

TOD-DL remains a portfolio project under active development. URL queues use
the native transfer engine. Protected continuation requires authenticated,
reread prefix bytes and a strong ETag or trusted expected checksum. Weak or
missing ETags alone cannot protect continuation.

The operator guide records successful fresh Tor acquisitions with signed
record and final-file verification. Interrupted source resume, the complete
native acceptance matrix, and the operator pilot remain unverified.
Descriptor acquisition remains blocked in the public CLI.

Specifications define behavior contracts. Their status lines distinguish
implemented behavior from planned work. Dated reports establish evidence
for their stated scope; historical results do not certify a current build.
The [project brief][brief] explains validation evidence and project limits.
The public demo does not publish the complete acceptance evidence.

## About this repository

This repository contains public documentation and presentation artifacts.
The [operator guide][operate] explains workflows for the separate downloader.
The [development guide][develop] describes work in the private source
repository. Commands and source paths in those guides require that checkout.

The local artifact builder supports full and limited document builds. Run
`node build-html-pdf.js --max-specs 5` to select up to five specifications in
filename order. The builder defaults to `./build/` and writes
`output_<design>.html` and `combined_output_<design>.pdf`. Its current design
is `folio`, selected by `DESIGN` in `build-html-pdf.js`. A build requires the
private specification inputs and the dependencies in `package.json`.
Generated files in `build/` are ignored by Git. The published five-spec demo
linked above is a separate artifact and uses the Signal design.

Start with the [project brief][brief], then follow the guide for the
engineering question you want to review.

[banner]: assets/tod-dl-banner.svg
[brief]: PORTFOLIO-OVERVIEW.md
[architecture]: ARCHITECTURE.md
[operate]: OPERATOR-GUIDE.md
[decisions]: PORTFOLIO-OVERVIEW.md#engineering-decisions
[develop]: DEVELOPMENT.md
[audit]: AUDIT-RAIL.md
[custody]: CHAIN-OF-CUSTODY.md
[specs]: SPECIFICATIONS.md
[identity]: VISUAL-IDENTITY.md
[motion]: assets/acquisition-motion.gif
[motion-page]: assets/acquisition-motion-signal.html
[motion-still]: assets/acquisition-motion.png
[demo-pdf]: tor-dl-tech-specs.pdf
[demo-html]: tor-dl-tech-specs.html
[presentation]: tor-dl-web.html
