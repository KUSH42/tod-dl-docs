# System architecture

[Documentation](README.md) / Architecture

TOD-DL separates acquisition authority, operator interaction, and independent
verification. SQLite owns recovery state. Signed artifacts provide the
exportable history. The console observes published state and requests
confirmed actions.

## Component boundaries

The diagram shows the public URL-queue path. Descriptor components have a
separate internal path; their public release gate remains closed.

```mermaid
flowchart TB
    Q[Queue and retained inputs] --> C[Acquisition controller]
    C --> DB[(SQLite recovery state)]
    C --> W[aria2 transfer workers]
    W --> S[Staging]
    S --> V[Validation and promotion]
    V --> F[Final files]
    C --> P[Signed provenance]
    V --> P
    C --> T[Published telemetry]
    T --> M[Textual monitor]
    M --> I[Read-only inspection]
    M --> X[Confirmed local commands]
    X --> C
    P --> R[Independent verifier]
    Q --> R
    F --> R
    K[External public key and index digest] --> R
    R --> H[Verified handover export]
```

Telemetry is an observation surface. Inspection serves detailed queries.
Neither gives the monitor direct access to acquisition database writes.

## Ownership map

Start with the owning component when tracing a behavior.

| Concern | Source | Contract |
| --- | --- | --- |
| CLI and lifecycle | [CLI][ref-1], [core][ref-2] | [Acquisition][reliable] |
| Transfer | [Admit][ref-3], [transfer][ref-4] | [Flow][reliable] |
| Promotion | [State][ref-5], [storage][ref-6] | [Recovery][recovery] |
| Console and endpoints | [UI][ref-7], [IPC][ref-8] | [Console][console] |
| Signed records | [Provenance][ref-9] | [Custody][custody] |
| Inventory | [Parse][ref-10], [lineage][ref-11] | [Inputs][inventory] |
| Transport | [Routes][ref-12], [inputs][ref-13] | [HTTP][transport] |
| Local evaluation | [Harness][ref-14] | [Evaluation][evaluation] |

The `Downloader` class composes mixins by concern. Read the CLI, the owning
mixin, and its immediate callers together when changing behavior.

## The promotion boundary

Promotion bridges SQLite and the filesystem. The controller records durable
intent before creating the final file. It records durable provenance before
committing completion. An exclusive hard link prevents replacement of an
existing final path.

Recovery checks interrupted promotion against the retained bytes and state.
Uncertainty enters review. The [custody walkthrough](CHAIN-OF-CUSTODY.md)
shows the write order and resulting records.

## The trust boundary

Signed records and recovery state serve different purposes. SQLite enables
continuation after interruption. Signed checkpoints and indexes support
review outside the running controller.

The verifier uses an external public key and trusted run-index digest.
Keeping both beside mutable records would let a replacement record set
supply its own apparent trust. See the [audit guide](AUDIT-RAIL.md).

## The transport boundary

The legacy URL-queue path uses aria2. Route selection resolves Tor, direct,
or configured proxy behavior, subject to version-specific admission guards.
An onion host requires Tor; a required Tor route has no direct fallback.

Descriptor components use item identities and a separate streaming adapter.
Do not infer public support from an adapter test. `run_locked()` in
[`core.py`](../src/downloader/core.py) rejects format 2 descriptor acquisition.
The [transport contract][transport] and [open work](../specs/OPEN-WORK.md)
define the remaining integration work.

## The observation boundary

The controller publishes telemetry. The console can inspect a run and send
confirmed control requests through authenticated local endpoints. The
controller records accepted decisions and applies their effects.

Tor renewal affects future streams. It does not change an active transfer
or prove that Tor selected a different route. Stop actions and durable item
exclusion have distinct effects described in the [operator guide][ref-15].

## Next steps

Use the [development guide](DEVELOPMENT.md) to select focused tests.
Use the [specification guide](SPECIFICATIONS.md) to trace an exact contract.

[reliable]: ../specs/SPEC-reliable-acquisition.md
[recovery]: ../specs/SPEC-acquisition-fault-recovery.md
[console]: ../specs/SPEC-console-ui.md
[custody]: ../specs/SPEC-run-custody.md
[inventory]: ../specs/SPEC-inventory-snapshot-manifest.md
[transport]: ../specs/SPEC-http-transport.md
[evaluation]: ../specs/SPEC-acquisition-evaluation-infrastructure.md

[ref-1]: ../src/tod_dl.py
[ref-2]: ../src/downloader/core.py
[ref-3]: ../src/downloader/admission.py
[ref-4]: ../src/downloader/transfer.py
[ref-5]: ../src/downloader/state.py
[ref-6]: ../src/downloader/storage.py
[ref-7]: ../src/console/
[ref-8]: ../src/ipc/
[ref-9]: ../src/provenance/
[ref-10]: ../src/inventory.py
[ref-11]: ../src/provenance/lineage.py
[ref-12]: ../src/routing.py
[ref-13]: ../src/downloader/request_descriptors.py
[ref-14]: ../src/evaluation/
[ref-15]: OPERATOR-GUIDE.md#monitor-and-control-a-run
