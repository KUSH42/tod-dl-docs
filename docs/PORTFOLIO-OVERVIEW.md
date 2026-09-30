# TOD-DL project brief

[Documentation](README.md) / Project brief

![TOD-DL: resumable acquisition, durable state, verifiable custody][banner]

**Systems engineering for acquisition that can fail halfway through.**

TOD-DL is a forensic acquisition portfolio project. It combines transfer
orchestration, durable state, filesystem safety, cryptographic records, and
an operator console. The useful review question is how these components
preserve their guarantees when a transfer, process, or host fails.

## From fragments to proof

This motion study follows three ideas: acquire bytes, check integrity, and
trace the recorded history. The animation uses synthetic points. It does
not show a live acquisition or establish source authenticity.

![Points assemble into a file, then form linked provenance records.][motion]

The animation plays once. Open the [static image][motion-still] for a still
view, or download the [interactive page][motion-page] and open it in a
browser. The page provides stage selection, pause, and keyboard controls.
It starts paused when your system requests reduced motion.

The sequence illustrates the design. The [custody walkthrough]
(CHAIN-OF-CUSTODY.md) explains the actual ordering and recovery rules.

## The problem

An unreliable source can interrupt a transfer or change a file during
resume. A host can lose power between creating a final file and recording
completion. An operator can inherit a directory that already contains files.

The system must preserve bytes, record uncertainty, and leave enough
information for an independent check. The controller therefore separates
selection, transfer, validation, finalization, and verification.

## Engineering decisions

Each decision below connects a failure mode to code and test evidence.

### Make filesystem promotion recoverable

A database transaction cannot atomically include a filesystem operation.
TOD-DL records promotion intent, creates the final exclusively, records
provenance, and commits completion. Recovery reconciles interrupted steps.

Review [storage](../src/downloader/storage.py), the
[custody walkthrough](CHAIN-OF-CUSTODY.md), and
[finalization recovery tests](../tests/test_v2_finalization_recovery.py).
The tradeoff is explicit recovery logic and a shared filesystem for state
and destination.

### Authenticate a recoverable prefix of history

A process can die before signing a closing summary. Version 2 records use
signed checkpoints and run indexes to authenticate retained history across
sessions. A verifier reports an unsigned tail separately.

Review the [provenance writer](../src/provenance/__init__.py) and
[run custody contract](../specs/SPEC-run-custody.md). Rollback detection still
needs an index digest retained outside the case directory.

### Keep the console outside acquisition authority

The Textual monitor consumes telemetry and inspection responses. Confirmed
commands go through a separate local endpoint; the controller owns changes.
This separation makes observation possible without giving the UI database
or final-file write authority.

Review [IPC components](../src/ipc/), the
[telemetry projection contract](../specs/SPEC-telemetry-state-projection.md),
and [control outbox tests](../tests/test_control_decision_outbox.py).

### Reproduce the inputs behind a selected set

Inventory processing retains snapshots, manifests, and queue lineage.
Verification can rederive outputs from retained inputs instead of trusting
a manifest because its file digest matches.

Review the [inventory tool](../src/inventory.py),
[lineage implementation](../src/provenance/lineage.py), and
[rederivation tests](../tests/test_inventory_rederivation.py).

### Test the boundaries before exposing new transport

Request descriptors separate resource identity from the resolved request.
The descriptor adapters have local HTTP and HTTPS fixtures, but the public
CLI still refuses descriptor acquisition. The release gate makes that
integration boundary visible.

Review the [transport matrix][transport-report],
[adapter tests](../tests/test_descriptor_transport_acceptance.py), and
[open transport work](../specs/OPEN-WORK.md#version-2-http-transport).

## Validation evidence

These are historical validation records, not fresh test results. Each
report states its method, date, and limits.

- **Engine selection, September 20, 2026.** E01–E14 passed with local
  fixtures on the reported tree. Read the [engine report][engine-report].
- **Transport, September 28, 2026.** The 194 cited tests passed. Header-case
  coverage remains open; descriptor resume is not covered. Read the
  [transport matrix][transport-report].
- **External timestamps, September 24, 2026.** X01–X08 passed with a local
  authority and synthetic certificates. Read the [acceptance report]
  [timestamp-report].
- **Continuous integration.** The [CI workflow][ref-1] defines unit tests,
  syntax checks, and secret scanning. Inspect the actual run for its result.

These reports demonstrate how the project evaluates behavior. Production
validation remains open in the [work register](../specs/OPEN-WORK.md).

## Skills demonstrated

The repository provides concrete material for a technical interview or code
review across several areas.

- **Systems design:** state machines, crash recovery, storage ordering, and
  resource admission.
- **Security engineering:** path checks, no-overwrite creation, secret
  redaction, canonical records, and explicit trust anchors.
- **Verification:** fault injection, synthetic transport fixtures, artifact
  rederivation, and read-only validation.
- **Interface design:** a terminal console with inspection, review, and
  confirmed control actions.
- **Technical communication:** linked contracts, acceptance criteria, dated
  results, and documented implementation gaps.

The [development guide](DEVELOPMENT.md) provides focused commands and a
source tour for each area.

## Scope and limits

TOD-DL remains a portfolio project, not a production-certified acquisition
system. Signatures establish a relationship to a trusted key. They do not
establish source authenticity or a person's identity.

A complete run accounts for every selected item; individual outcomes can
include review, exclusion, or failure. External timestamps apply to signed
artifacts and do not establish the time of every acquisition event.

Sealed-sidecar export, re-encryption, preservation profiles, and parts of
transport integration remain open. The [specification guide](SPECIFICATIONS.md)
links the relevant contracts without treating planned behavior as available.

## Next steps

Read the [architecture](ARCHITECTURE.md) for component boundaries, or follow
one file through the [chain of custody](CHAIN-OF-CUSTODY.md).
For use permissions, read the [license](../LICENSE).

[engine-report]: ../specs/reports/acquisition-tool-evaluation-2026-09-20e.md
[transport-report]:
  ../specs/reports/descriptor-transport-t01-t10-matrix-2026-09-28.md
[timestamp-report]:
  ../specs/reports/external-timestamping-acceptance-2026-09-24.md

[banner]: assets/tod-dl-banner.png
[ref-1]: ../.github/workflows/ci.yml
[motion]: assets/acquisition-motion.gif
[motion-still]: assets/acquisition-motion.png
[motion-page]: assets/acquisition-motion.html
