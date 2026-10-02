# Specification guide

[Documentation](README.md) / Specifications

Use the specifications as an engineering reference library. Start with a
system concern, then follow its contract to source, tests, and dated reports.
For an introduction, read the [project brief](PORTFOLIO-OVERVIEW.md).

## Read a contract and its evidence together

Each spec owns a behavior contract and carries a `Status:` line. Read that
line and the relevant section before treating a requirement as available.
A partially implemented spec can contain completed behavior and planned
extensions.

[Open work](../specs/OPEN-WORK.md) identifies remaining gaps.
[Reports](../specs/reports/) record results for a dated scope. Neither replaces
the owning specification. This guide groups links without copying mutable
status labels.

## Acquisition and recovery

These contracts define selection, transfer, storage, and interruption handling.

- [Native transfer](../specs/SPEC-native-transfer-engine.md): native worker
  boundaries and release 1 requirements.
- [Native cutover](../specs/SPEC-native-engine-cutover.md): protected resume,
  engine retirement, compatibility, and the current acceptance gate.
- [Reliable acquisition](../specs/SPEC-reliable-acquisition.md): selection,
  retries, resource admission, validation, and finalization.
- [Fault recovery](../specs/SPEC-acquisition-fault-recovery.md): required
  failure cases and assertions.
- [HTTP transport](../specs/SPEC-http-transport.md): descriptors, route policy,
  redirects, HTTPS, and representation checks.
- [Multiple routes](../specs/SPEC-acquisition-multi-route.md): Tor, direct,
  and configured proxy acquisition.
- [Tor recovery](../specs/SPEC-tor-circuit-recovery.md): isolation and circuit
  renewal boundaries.
- [Attached control launch](../specs/SPEC-control-launch.md): console startup
  and lifecycle.
- [Resume scripts](../specs/SPEC-resume-script.md): generated run continuation.

## Provenance, trust, and handover

These contracts define recorded claims and the evidence required to check them.

- [Acquisition provenance](../specs/SPEC-acquisition-provenance.md): the
  original signed session records.
- [Run custody](../specs/SPEC-run-custody.md): version 2 checkpoints, run
  indexes, completeness, lineage, and handover.
- [Acquisition context](../specs/SPEC-acquisition-context.md): session
  environment, validation evidence, and custody references.
- [Verification hardening](../specs/SPEC-manifest-verification-hardening.md):
  parsing, canonicalization, schema checks, and safe reads.
- [Manifest evolution](../specs/SPEC-manifest-v2-evolution.md): schema
  ownership, revisions, and extensions.
- [External timestamping](../specs/SPEC-external-timestamping.md): submission,
  retained evidence, trust, and release policy.
- [Response-body integrity](../specs/SPEC-response-body-integrity.md): the
  received-byte and final-file coverage, with remaining profile work.
- [Sealed-sidecar handover](../specs/SPEC-sealed-sidecar-handover.md): the
  planned export inclusion policy.
- [Multiple sealed recipients](../specs/SPEC-sealed-multi-recipient.md):
  encrypted request evidence for several recipients and acceptance gaps.
- [Authorized re-encryption](../specs/SPEC-sealed-re-encryption.md): the
  planned derived evidence sets for later recipients.
- [Forensic compliance](../specs/SPEC-forensic-compliance.md): assessment
  requirements and profile definitions.
- [Evidence preservation](../specs/SPEC-evidence-preservation.md): qualified
  time and archive preservation requirements.

Compliance and preservation documents describe requirements. They do not
establish a certification for the current software.

## Inventory and discovery

These contracts connect listings, manifests, and selected request inputs.

- [Inventory manifests](../specs/SPEC-inventory-snapshot-manifest.md): local
  parsing, accepted snapshots, manifests, diffs, and queue export.
- [Discovery](../specs/SPEC-inventory-discovery.md): configured listing fetch
  and remaining discovery work.
- [Request mapping](../specs/SPEC-inventory-request-mapping.md): inventory
  rows mapped to version 2 descriptors.
- [Target-directory rescan](../specs/SPEC-target-dir-rescan.md): planned
  reconciliation of local files against inventory.

## Console and control

Start with the console contract, then read the view or endpoint you need.

- [Console UI](../specs/SPEC-console-ui.md): application layout and lifecycle.
- [Visual style](../specs/SPEC-console-visual-style.md): labels, emphasis,
  colors, and interaction presentation.
- [Controller controls](../specs/SPEC-controller-control-ui.md): authority,
  authentication, confirmation, and command behavior.
- [Queue](../specs/SPEC-console-queue.md): selection, search, pagination,
  and item actions.
- [Errors and review](../specs/SPEC-console-errors-review.md): issue review.
- [Item details](../specs/SPEC-console-item-details.md): one item's state.
- [Worker table](../specs/SPEC-console-worker-table.md): active worker rows.
- [Worker details](../specs/SPEC-console-worker-details.md): transfer detail.
- [Detail layout](../specs/SPEC-console-detail-layout.md): detail panels.
- [Keymap](../specs/SPEC-console-keymap.md): keyboard interaction.
- [Activity log](../specs/SPEC-console-activity-log.md): event presentation.
- [Inspection](../specs/SPEC-console-inspection.md): read-only detail queries.
- [Download telemetry](../specs/SPEC-download-telemetry.md): published progress.
- [State projection](../specs/SPEC-telemetry-state-projection.md): committed
  controller state projected for observation.

## Evaluation

These contracts define how transfer engines are tested against local fixtures.

- [Tool evaluation](../specs/SPEC-acquisition-tool-evaluation.md): scenarios
  and engine-selection criteria.
- [Evaluation infrastructure][ref-1]:
  fixture generation, failure injection, and measurement.

The [project brief](PORTFOLIO-OVERVIEW.md#validation-evidence) links selected
reports with dates and limits. Browse [all reports](../specs/reports/) when
reviewing a specific acceptance gate. Native cutover requires its complete
matrix and interrupted-resume operator pilot. Historical aria2 evaluation
does not establish native acceptance.

The [October 1 native handover][native-evidence] links the failed
source pilot, successful full acquisitions, and diagnostic retry. Read those
reports together; each report states its source scope and verification limits.

## Publication proposal

The tracked Markdown guides are the current reading interface. The following
proposal describes a future HTML and PDF publication path; no site build or
deployment is included in this documentation change.

### One source, three reading formats

Keep `SPEC-*.md` as the contract source. Render editions from an explicit
Git revision so readers can connect an artifact to its source.

| Format | Reader need | Proposed presentation |
| --- | --- | --- |
| Markdown | Review changes | Grouped guide and source history |
| HTML | Search and browse | Topic navigation, status, and diagrams |
| PDF | Share a snapshot | Subject volumes and source revision |

Use the same TOD-DL identity, restrained color palette, and architecture
language across the three formats. Keep the prose readable without color.
Render diagrams with text labels and retain code blocks as selectable text.

### Existing local artifacts

The inspected workspace contains `specs/build-pdf.js`, HTML/PDF outputs,
styles, and preview scripts. Those files are ignored by Git. They therefore
cannot currently be treated as a reproducible publishing toolchain for a
fresh clone.

The inspected builder concatenates specs alphabetically and renders HTML
and PDF through Puppeteer. Its cover uses `AegisFetch` and the blanket status
`Approved / Active`; it loads diagram and highlighting resources from CDNs.
Those labels do not reflect this repository's identity or per-spec statuses.
The builder also treats a Mermaid render timeout as a non-fatal condition.
A publication build must distinguish a document without diagrams from a
failed diagram render.

The next implementation can reuse the rendering approach after addressing
these gaps. Preserve local experiments until a replacement is verified.

### Proposed publication gate

A published edition must make its source, scope, and render result reviewable.

1. Track one builder and its exact dependency versions, including a lockfile.
2. Bundle rendering assets locally so builds do not depend on mutable CDNs.
3. Read titles and status text from the source specs. Do not infer approval.
4. Group specs by subject and include a complete reference edition.
5. Record the source commit and UTC build time in each edition.
6. Validate internal links, anchors, diagrams, and code-block rendering.
7. Check narrow-screen HTML, keyboard navigation, contrast, and printed PDF.
8. Publish verified artifacts with the matching source revision.

Keep generated PDFs, preview frames, and videos in release artifacts when
that pipeline exists. Source specs and their history remain the authority.

## Next steps

Use [architecture](ARCHITECTURE.md) to find an implementation owner.
Use [open work](../specs/OPEN-WORK.md) to identify the next unfinished contract.

[ref-1]: ../specs/SPEC-acquisition-evaluation-infrastructure.md

[native-evidence]:
  ../specs/reports/native-acquisition-handover-2026-10-01.md
