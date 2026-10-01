# Operator guide

[Documentation](README.md) / Operate

Use this guide to prepare, run, resume, verify, and export a bounded
acquisition. Commands run from the repository root and use placeholder case
paths. Use only material you are authorized to acquire and retain.

TOD-DL remains a portfolio project with open validation work. Start with
queue validation. Review the [current limits][open-work] before source
contact. Public descriptor acquisition remains unavailable. URL queues use
native transfer; the complete native acceptance matrix and operator pilot
remain unverified. The [cutover contract][cutover] owns current resume and
compatibility rules.

## Requirements

You need Python 3 and the pinned downloader dependencies. Tor routes also
need `torsocks` and a local Tor service. The Tor SocksPort must use
`IsolateSOCKSAuth`; the controller checks it through the ControlPort.
The native engine does not require `aria2c`. The Tor control cookie default
is owned by `--tor-control-cookie` in [`tod_dl.py`](../src/tod_dl.py).

Create the downloader environment with these commands:

```bash
python3 -m venv .venv
. .venv/bin/activate
python3 -m pip install -r requirements-downloader.txt
```

Install the monitor dependencies in the same environment for attached
`--control` launches:

```bash
python3 -m pip install -r requirements-monitor.txt
```

## Prepare the case and context

Run the commands from the repository root. Paths such as `/case` and
`/secure` are placeholders for operator-managed directories. Create the
case tree outside the repository. Keep state and destination on one
filesystem. Keep signing keys outside the case tree.

Before a live version 2 run, prepare an operator-owned context declaration.
The [acquisition context contract][context] defines its format, component
artifacts, validation report, validation evidence, and opaque custody
references. Each referenced file must exist and pass validation. The
controller requires `--context-declaration` for each session, including
resume. The examples use `/secure/context-declaration.json`.

The declaration's case ID must match the run. Its example version strings
are illustrative; record the actual build and validation evidence. The
controller retains the declared artifacts and measured session context. It
does not retain the declaration file itself.

[context]: ../specs/SPEC-acquisition-context.md
[cutover]: ../specs/SPEC-native-engine-cutover.md

## Prepare a queue

Each queue file contains one URL per line. Blank lines, comment lines,
duplicate URLs, unsafe paths, URLs with credentials, and URLs with query
values are ignored or rejected.

A URL has the form `http://HOST/COLLECTION/PATH` or
`https://HOST/COLLECTION/PATH`. The `/data/` segment is
optional. The final path is `COLLECTION/PATH`, so a URL with `/data/` keeps
`data` in the final path. A URL with only a collection, or with `ALL_FILES` as
the file, is rejected. A URL is also rejected if a decoded segment makes the
path absolute (for example `/%2Fetc/passwd` or `//passwd`), contains `..`, or
contains a NUL byte. Such a path could leave the destination directory.

A line can carry optional `key=value` tokens after the URL, separated by
whitespace, in any order:

- `size=<token>`: a rough, human-readable size estimate, for example `1.8M`
  or `43K`. The downloader does not parse or validate this value beyond
  rejecting an empty one; it stores the token as given.
- `sha256=<hex>`: the expected 64-character hex SHA-256 digest. If the
  downloaded file's digest does not match, the downloader moves the file to
  a review candidate instead of promoting it.
  A later queue that gives an existing item a different `size=` or `sha256=`
  value replaces the stored value.
- `generation=<id>`: an opaque source-generation identifier that the queue
  producer sets, for example the inventory snapshot hash. The downloader
  stores it per item. If a later run gives an `unavailable` item a different
  identifier, the item's daily recheck runs early. The early recheck counts as
  that day's recheck. Unless a later run gives another differing identifier,
  the next recheck waits a full day after it.
- `route=<tor|direct|proxy:NAME>`: the network route this item must use; see
  [`src/routing.py`](../src/routing.py) and
  [the multi-route acquisition
  specification](../specs/SPEC-acquisition-multi-route.md).
  A line without this token keeps the historical behavior: the item fetches
  through the verified Tor route, exactly as before this token existed. An
  `.onion` host always fetches through Tor and ignores a `route=direct` or
  `route=proxy:NAME` token on its line. The route is resolved and stored once,
  at import time, and does not change for that item afterward.

Example: `https://example.onion/data/case/file.bin size=1.8M
sha256=9f86d081884c7d659a2feaa0c55ad015a3bf4f1b2b0b822cd15d6c15b0f00a08`

Store queue files outside this source repository when they contain case data.

## Run an acquisition

Start with a dry run. The dry run reads queues and reports the selected final
paths. It does not start a native transfer or contact a source.

To validate a format 2 `requests.jsonl` file and its source policy, add
`--source-policy /case/source-policy.json` and `--artifact-root /case` to a
dry run. TOD-DL validates every request before it applies `--max-files`.
Descriptor transfer remains unavailable.

```bash
python3 src/tod_dl.py \
    --queue /case/urls_priority.txt \
    --destination /case/downloaded_files \
    --state /case/download-state \
    --dry-run --max-files 3
```

After an operator reviews the paths and storage, start a bounded run:

```bash
python3 src/tod_dl.py \
    --queue /case/urls_priority.txt \
    --destination /case/downloaded_files \
    --state /case/runs/RUN_ID --run-id RUN_ID \
    --case-id CASE_ID --artifact-root /case \
    --provenance-signing-key /secure/run-signing-key.pem \
    --context-declaration /secure/context-declaration.json \
    --workers 4 --max-files 5
```

A new run writes provenance version 2 and needs its own state directory, a
case ID, an artifact root, a context declaration, and a signing key
outside the case tree. See
"Version 2 provenance" below.

The destination and state directories must use the same filesystem. The
controller keeps a 10 GiB free-space reserve by default. It never replaces an
existing final. A collision creates a review item and preserves incoming bytes
under `download-state/redownload-candidates/`. A blocked promotion (an unsafe
destination path or a failed final link) also creates a review item. It keeps
the staged file in place and adds a hard link to it under
`redownload-candidates/`, with no extra disk use. Safe-path remediation
removes that link when it promotes the file. The controller hashes one
staged file at a time; a lagging hash holds its worker slot and pauses new
admission until the hash finishes.

Native telemetry reads counters from the controller staging sink. The
controller does not start an aria2 process or RPC server. `--engine native`
selects the default engine explicitly. Retired aria2 options, including
`--engine aria2`, `--aria2c`, `--aria2-rpc`, `--no-aria2-rpc`, and `--rpc-eval`,
fail with exit status 2.

The controller verifies the Tor isolation preflight only when at least one
selected item resolves to the `tor` route; an all-`direct` or all-`proxy`
selection starts without contacting a Tor control port. Native URL queues
support HTTP and HTTPS. A `proxy:NAME` route needs an HTTP proxy endpoint in
`--proxy-config FILE`; unsupported proxy configurations enter review. If a
queue's selected items resolve to more than one route kind (for example, an
`.onion` item mixed with a clearnet `route=direct` item), the controller refuses
to start until the operator passes `--allow-mixed-routes`, so a mixed selection
is never a silent surprise.

## Recover from low storage

The controller stops admission when free space at the destination and state
filesystem falls below the reserve. The reserve default is owned by
`--reserve-bytes` in [`tod_dl.py`](../src/tod_dl.py). The controller pauses
admission and keeps control available. Local storage failures consume no
network retry budget.

Free space without deleting retained evidence. Then confirm
`resume_admission` in the console, or restart with the same run inputs.
Recovery preserves uncertain suffix bytes and restores authenticated prefixes
before admission resumes. If repair fails, admission remains paused.

A signing failure also pauses admission and stops native writers. Repair the
signing problem before confirming `resume_admission`. The controller must
stop all writers and finish authenticated-prefix recovery before transfer
restarts. See [live signing repair][cutover].

## Review a candidate

An item enters `review_required` when TOD-DL cannot safely promote it
automatically: a checksum mismatch, a name that exceeds the filesystem limit
(`ENAMETOOLONG`), an unsafe path, or a rejected continuation response.
A short body can retry from a protected authenticated prefix. Changed or
unprotected remote representations enter review; Last-Modified alone does
not authorize native continuation. Use `--status` to list `review_required`
items and read each item's `review_code` and `last_error`.

Three row-scoped actions apply to a `review_required` item. Each is durable and
confirmed before the controller applies them:

- `exclude_item` durably moves the item to `excluded`. It leaves every other
  item's state, selected set, and queue rank unchanged, and it never fires
  automatically from any outage, retry, validation, or promotion path. In
  the monitor's Queue tab (`src/monitor.py --control`), select the item and
  press `x`, then confirm.
- `resume_new_generation` applies only to a `changed_remote_representation`
  or `no_reliable_version_protection` review item. It clears the review code
  and returns the item to `queued` under a new staging generation, so the
  next attempt starts a fresh download instead of resuming stale bytes. In
  the monitor's Queue tab (`src/monitor.py --control`), select the item and
  press `N`, then confirm. The key is offered only for a row in the
  `review_required` bucket; the controller still rejects an item with any
  other review code.
- `retry_access_denied` applies only to an `access_denied` review item (an
  HTTP 401 or 403). It returns the item to `queued` and keeps its staged
  bytes, attempts, and staging generation. The origin pause ends when no
  other `access_denied` item of that origin remains. A repeated 401 or 403
  pauses the origin again. In the monitor's Queue tab (`src/monitor.py
  --control`), select the item and press `A`, then confirm. The key is
  offered only for a row in the `review_required` bucket; the controller
  still rejects an item with any other review code.

## Resume a run

Use the same run ID, queue files, and `--max-files` value to resume a stopped
run. The downloader binds the run ID to queue file hashes and the selection
value. A retry cannot add a later queue item to the selected set.

```bash
python3 src/tod_dl.py \
    --queue /case/urls_priority.txt \
    --destination /case/downloaded_files \
    --state /case/runs/RUN_ID --run-id RUN_ID \
    --case-id CASE_ID --artifact-root /case \
    --provenance-signing-key /secure/run-signing-key.pem \
    --context-declaration /secure/context-declaration.json \
    --max-files 5 --retry-now
```

Always pass `--run-id` to resume. Without it, the controller names a new run,
and a version 2 state directory refuses a second run.

Native continuation requires checkpoint-authenticated range receipts, a
successful prefix reread, and a strong ETag or trusted expected checksum.
Weak or missing ETags alone cannot protect a continuation. Recovery preserves
unsigned suffix bytes before installing the authenticated prefix.

The controller requests the next range only after those checks pass. It
checks the response head before appending bytes. Ignored ranges, changed
protection, invalid ranges, and other rejected heads preserve the prefix and
enter review. A source response does not guarantee resumability.

A network retry keeps the staging generation. A confirmed replacement from
review starts a new generation and cannot use the old receipt coverage.
`--retry-now` ends an eligible network retry wait; it does not bypass review
or repair checks. Retry timing is owned by
[`downloader/constants.py`](../src/downloader/constants.py).

The active build refuses archived aria2 runs and incompatible version 2 schema
pins before runtime writes or source contact. Resume or verify those runs
with their matching retained build. Keep archived records and partials
unchanged; do not edit an engine field or schema pin to force compatibility.

Use `--status` to read persisted state without starting transfers.

A fresh run writes `resume-RUN_ID.sh` (mode `0700`) in the current directory,
or in `--resume-script-dir` when you pass it. Run that script to resume the
same run, with a console attached, as a single command. Pass
`--no-resume-script` to skip writing it. `--resume-script-dir` must not lie
under `--state` or `--destination`.

## Monitor and control a run

Pass `--control` to `tod_dl.py` to start or resume a run with an interactive
console attached as a child process, in one command:

```bash
python3 src/tod_dl.py \
    --queue /case/urls_priority.txt \
    --destination /case/downloaded_files \
    --state /case/runs/RUN_ID --run-id RUN_ID \
    --case-id CASE_ID --artifact-root /case \
    --provenance-signing-key /secure/run-signing-key.pem \
    --context-declaration /secure/context-declaration.json \
    --max-files 5 --control
```

`--control` requires an interactive terminal on standard input, output, and
error. If the console exits, the run continues headless and prints the
`monitor.py --control` command line to reattach. `--control` is incompatible
with `--status`, `--retry-now`, and `--dry-run`.

To attach a console to an already-running or already-finished run instead,
run the monitor directly. The monitor reads a published telemetry snapshot.
It does not open the acquisition database or write final provenance.

```bash
python3 src/monitor.py --state /case/runs/RUN_ID \
    --run-id RUN_ID --control
```

With `--control`, every action below needs confirmation before the
controller runs it.

- Press `r` to make selected retryable items eligible now.
- Press `t` to request fresh Tor circuits for future streams. This does not
  change active transfers or prove a new route. The controller applies the
  configured rate limit, which defaults to 60 seconds.
- Press `p` to pause admission. Active transfers keep running; the
  controller admits no new transfer until you resume.
- Press `u` to resume admission after any required storage or signing repair.
- Press `d` to drain and stop the run. Admission stops now. Active
  transfers finish to a durable state, then the run exits. You cannot undo
  this action.
- Press `k` to checkpoint and stop the run. The controller checkpoints and
  terminates active transfers immediately, then the run exits. You cannot
  undo this action.

The controller records every request it accepts, with its outcome and a
durable state revision.

## Verify provenance

The controller writes signed record sets below
`<state>/provenance/<run-id>/<session-id>/`. Keep the trusted Ed25519
public key or its fingerprint outside the record directory. To derive the
public key from a signing key, run
`openssl pkey -in /secure/run-signing-key.pem -pubout`.

Run the verifier to check the signature, event chain, final paths, byte counts,
and SHA-256 digests. The verifier does not modify downloaded files. The
verifier always needs `--public-key`. Add `--expected-fingerprint` to pin that
key to a fingerprint you keep elsewhere. A fingerprint alone fails, because it
cannot verify the signature.

```bash
python3 src/provenance/verify_provenance.py \
    /case/runs/RUN_ID/provenance/RUN_ID/SESSION_ID \
    --destination /case/downloaded_files \
    --public-key /secure/TRUSTED_PUBLIC_KEY.pem
```

### Version 2 provenance

New native runs write provenance version 2. The active build resumes only
compatible native runs. These rules apply to a version 2 run:

- Give each new version 2 run its own `--state` directory. The controller
  refuses a state directory that holds another run. It also refuses to run
  a version 1 controller against a version 2 state directory.
- A new run needs `--case-id ID`: an opaque label without personal data,
  made of 1 to 128 letters, digits, `.`, `_`, or `-`. A resume reads the
  stored ID, and a different `--case-id` refuses the start.
- Every session needs `--artifact-root DIR`. Each queue file must lie below
  it, and the records store queue paths relative to it.
- To bind an inventory-derived queue to its inventory, name each input with
  an option. All paths must lie below `--artifact-root`. Give
  `--inventory-snapshot DIR` (an accepted snapshot directory),
  `--inventory-policy FILE`, and `--inventory-manifest DIR` together. Add
  `--inventory-checksums FILE` when the manifest used one. The controller
  requires `--inventory-old-snapshot DIR` for a candidate manifest. Use the
  accepted old snapshot named by the diff. Its raw digest must match the
  candidate header. Do not use this option for a regular manifest. The
  controller reads `QUEUE.provenance.json` for each queue. It refuses the
  start if a lineage check fails, and it records the inputs in the run. A
  resume must give the same options again.
- Every session needs `--provenance-signing-key` outside the state and
  destination directories.
- A new run writes `provenance-schema.json` in its state directory. Keep
  this file with the state. A resume refuses a missing or changed schema
  digest before it changes recovery state. An older version 2 run without
  the file needs its original compatible build for resume.
- The [installed schema registry][schemas] owns revision digests. Keep the
  schema captured with the run; do not substitute a newer schema file.
  Final-file records include SHA-256 and SHA-512 content digests. The
  verifier checks each declared final-file digest. Non-active version 2
  schemas require their matching retained verifier build; see the
  [cutover contract][cutover].
- Every session needs `--context-declaration`. See [case preparation][case].
- To timestamp each session's final signed run-index revision, give
  `--timestamp-policy FILE` below `--artifact-root`, plus
  `--service-config FILE` and `--trust-config FILE` outside the case.
  Add `--validation-material DIR` to retain local validation evidence.
  Give the same policy file on resume. An absent or `off` policy sends no
  request. The controller prints each submission status. A failed or
  pending submission does not provide external time assurance.
- Native URL queues support HTTP, HTTPS, Tor, direct routes, and configured
  HTTP proxies. Descriptor acquisition remains blocked independently of URL
  queue support. Native acceptance and the operator pilot remain unverified.
- Each run has its own state directory, so a `generation=` change from a
  later run cannot start an early recheck of an `unavailable` item.
- When every selected item has a terminal outcome, the controller closes
  the run. A later start with the same `--run-id` then ends at once. Add
  `--reopen-closed-run` to start a new session of the closed run.
- For key rotation, create the new run with `--prior-run-index`,
  `--prior-trusted-index-sha256`, and `--prior-public-key`, which name the
  prior run's index, its trusted digest, and its public key.

To check a complete version 2 run, give the run directory, the SHA-256 of
the run-index revision that you kept outside the case tree, the trusted
public key, and the `--artifact-root` directory that the run used:

```bash
python3 src/provenance/verify_provenance.py \
    /case/runs/RUN_ID/provenance/RUN_ID --scope run \
    --destination /case/downloaded_files \
    --artifact-root /case \
    --public-key /secure/TRUSTED_PUBLIC_KEY.pem \
    --trusted-index-sha256 TRUSTED_INDEX_SHA256
```

The verifier re-hashes each retained input, such as a queue file, below
`--artifact-root`. Without that option, the inputs stay unchecked and the
result is `INCOMPLETE`. The report states the lineage mode. A manual-queue
run reports inventory lineage as unavailable. For an inventory run, the
verifier applies the seven lineage checks to the retained inputs and prints
the snapshot origin, `fetched` or `local_import`. It then rebuilds the parse
report, manifest, queues, and both item registries from those inputs and
fails the run if any differs. The report says `inventory lineage verified`
when they all agree. For a candidate manifest, pass `--old-snapshot DIR` to
the run verifier. It recomputes the candidate path set from both listings.
The old listing must match the digest in the signed candidate header. Without
that listing, candidate lineage fails verification.

For a run that records a prior run, also pass the three `--prior-*`
options. `OK` means that the anchored revision is closed and complete.
`INCOMPLETE` means that no record failed a check, but some required
evidence is missing, for example an attempt that only an unsigned tail
holds. `FAIL` means that a record failed a check. Both exit nonzero. The
report also lists unsigned session tails, which it does not verify, and
records marked `redacted`, which cannot support source reconstruction.

### Sealed request evidence

This section describes implemented descriptor components. Public descriptor
acquisition remains blocked; these flags do not enable it. See the
[transport contract][transport] for the release gate.

By default, terminal records redact runtime query values and credential header
values. To retain exact descriptor requests for an authorized operator, pass an
X25519 public PEM key with `--sealed-request-evidence-recipient-key`. The
controller encrypts the resolved URL and configured headers before source
contact. It stores the encrypted sidecar under
`STATE/sealed-request-evidence/RUN-ID` unless you set
`--sealed-request-evidence-dir`.

The signed terminal record contains only the sidecar digest, size, recipient
key fingerprint, and attempt ID. Handover export omits sealed sidecars.
The repeatable `--sealed-request-evidence-recipient-keys` option selects
format 3 multi-recipient sealing. It is mutually exclusive with the singular
option. The [multi-recipient contract][sealed] lists acceptance gaps,
including the missing dry-run stop-reason check. Keep
the recipient private key outside the case directory. The normal run verifier
can check sidecars with `--sealed-evidence-dir DIR`. It must verify the signed
event before an authorized operator decrypts a sidecar.

### Handover export

To give a closed run to a recipient, export it. The run must verify as
complete against a trusted run-index hash and public key. The signing key
must match that public key, and it must not lie in the export. The export
holds the run records, the retained inputs below `inputs/`, and the final
files listed in `final-files.json`. It holds no private key.
For a candidate-manifest run, add `--old-snapshot DIR` to `create` and
`resume`. The exporter copies `DIR/raw.txt` into the handover. Its digest
must match the candidate header. The handover verifier reads that copy.

```bash
python3 src/provenance/export_handover.py create \
    --run-dir /case/runs/RUN_ID/provenance/RUN_ID \
    --destination /case/downloaded_files --artifact-root /case \
    --trusted-index-sha256 TRUSTED_INDEX_SHA256 \
    --public-key /secure/TRUSTED_PUBLIC_KEY.pem \
    --signing-key /secure/run-signing-key.pem \
    --export-path /case/handover/RUN_ID
```

`create` prints a private stage path, `RUN_ID.staging-UUID`. If `create`
stops early, run `resume` with the same arguments and `--staged-export`
instead of `--export-path`. `release` verifies the whole stage again, then
renames it to the export path. It never replaces an existing path. The
exporter holds the run's timestamp lock while it copies run submissions.
`resume` can copy submissions added after `create` and keeps handover
submissions already in the stage. A copied file that changed blocks `resume`.

```bash
python3 src/provenance/export_handover.py release \
    --staged-export /case/handover/RUN_ID.staging-UUID \
    --export-path /case/handover/RUN_ID \
    --trusted-index-sha256 TRUSTED_INDEX_SHA256 \
    --public-key /secure/TRUSTED_PUBLIC_KEY.pem
```

`release` holds the same lock through optional handover submission, final
verification, and rename. Add `--service-config /secure/tsa-service.json` and
`--trust-config /secure/tsa-trust.json` to submit the handover before release.
Add `--validation-material DIR` when local certificate and revocation files
must accompany the submission. With an absent or `off` run policy, an explicit
`--late-policy /secure/late-timestamp-policy.json` permits a handover-only
submission. Without `--service-config`, release makes no submission; use
`--trust-config` alone to validate an existing response. A
`required_for_handover` policy blocks release until both the referenced run
index and handover artifact have valid tokens. Every mode blocks release on
local timestamp sidecar errors. A blocked stage remains private.

The recipient verifies the export with the trusted hash and key, which must
come from outside the export. The verifier reads the finals and inputs from
the export, so it rejects `--destination` and `--artifact-root`.

```bash
python3 src/provenance/verify_provenance.py /received/RUN_ID \
    --scope handover --public-key /secure/TRUSTED_PUBLIC_KEY.pem \
    --trusted-index-sha256 TRUSTED_INDEX_SHA256
```

If a run or handover includes a bound timestamp response, add
`--trust-config /secure/tsa-trust.json`. The trust file and its DER root
certificates must stay outside the case and export. The verifier reads the
retained request, response, and validation material without network access.
Add `--require-valid-timestamps` to require a valid token for the run index
and, in handover scope, the handover artifact. A run policy with
`required_for_handover` requires both tokens in handover scope even without
this option. A late handover policy requires only the handover token. Missing
trust for a submitted token gives exit 2; a failed required check or local
sidecar integrity check gives exit 1. The report lists each included attempt.
It cannot prove that no attempt was removed before export.

To submit an intermediate signed run-index revision under an enabled
timestamp policy, provide retained inputs and external service and trust
configuration. The client checks the signed target before it sends a request.

```bash
python3 src/provenance/timestamp_client.py run-index \
    --run-dir /case/runs/RUN_ID/provenance/RUN_ID \
    --artifact-root /case \
    --target /case/runs/RUN_ID/provenance/RUN_ID/run-index-1.json \
    --public-key /secure/TRUSTED_PUBLIC_KEY.pem \
    --service-config /secure/tsa-service.json \
    --trust-config /secure/tsa-trust.json
```

For a private handover stage with no enabled run policy, an operator can
submit its signed handover artifact with an explicit late policy. The client
retains the exact policy bytes beside the request. Use the same command
without `--late-policy` when the run has an enabled policy.

```bash
python3 src/provenance/timestamp_client.py handover \
    --staged-export /case/handover/RUN_ID.staging-UUID \
    --public-key /secure/TRUSTED_PUBLIC_KEY.pem \
    --trusted-index-sha256 TRUSTED_INDEX_SHA256 \
    --service-config /secure/tsa-service.json \
    --trust-config /secure/tsa-trust.json \
    --late-policy /secure/late-timestamp-policy.json
```

Both commands accept `--validation-material DIR` for local certificate and
revocation files. A submission returns 0 only for a valid token. The exporter
and client use one per-run lock. Release reports each target status and states
that later submission history is unverified.

A recipient receipt is a separate signed file outside the export. Add
`--receipt FILE` and `--recipient-public-key FILE` to check it. If the
handover records no recipient fingerprint, also give
`--trusted-recipient-fingerprint`. Without a trusted receipt, the report says
`recipient acceptance unconfirmed`.

## Build a queue from an inventory listing

`src/inventory.py` reads a local `ls -R`-style listing, writes a reproducible
manifest, and exports a queue. It reads local files only and never replaces an
existing file. Use a case directory for all paths.

```bash
python3 src/inventory.py snapshot --input /case/ALL_FILES1 \
    --store /case/inventory
python3 src/inventory.py activate --store /case/inventory \
    --snapshot SNAPSHOT_PREFIX
python3 src/inventory.py manifest \
    --snapshot /case/inventory/snapshots/SNAPSHOT_NAME \
    --policy /case/policy.json --base-url https://HOST/COLLECTION \
    --output /case/manifests/m1
python3 src/inventory.py queue --manifest /case/manifests/m1 \
    --output /case/urls_1.txt
```

After you import a newer listing, compare it with the old one:

```bash
python3 src/inventory.py diff --old /case/inventory/snapshots/OLD_SNAPSHOT \
    --new /case/inventory/snapshots/NEW_SNAPSHOT --output /case/diffs/d1
```

`report.md` in the output directory is the dated report. A removed path does
not authorize local deletion. Review every `ambiguous` item.

The `manifest` command also accepts `--checksums FILE`. The file is an
operator's `sha256sum`-format list. The manifest records which listed paths
matched a digest and which did not.

To get a candidate manifest of the new and size-changed paths, add the policy
and base URL. Then export a queue from it:

```bash
python3 src/inventory.py diff --old /case/inventory/snapshots/OLD_SNAPSHOT \
    --new /case/inventory/snapshots/NEW_SNAPSHOT --output /case/diffs/d2 \
    --policy /case/policy.json --base-url https://HOST/COLLECTION
python3 src/inventory.py queue --manifest /case/diffs/d2/candidate-manifest \
    --output /case/urls_new.txt
```

The candidate manifest changes no queue and no running job.

A rejected snapshot lands in `rejected/` and cannot become a baseline. Read
its `parse-report.json` before you use `--max-issues`. The policy file assigns
each item to `priority`, `deferred`, or `rejected`; the specification shows
the format. `queue --disposition deferred` exports the deferred items. The
`--generation` option adds a `generation=` token to each line; use it only
when you want the downloader to run early rechecks. Read
[the specification](../specs/SPEC-inventory-snapshot-manifest.md) for the
runbook.

## Discovery network fetch

`src/inventory_fetch.py` fetches a configured source's inventory listing
over the network and files it through the same acceptance path as
`inventory.py snapshot --input`, so a fetched snapshot is validated
identically to a locally supplied one. Each source resolves its own route:

- An `.onion` host always fetches through the verified Tor route; it
  cannot set a proxy or an override.
- A clearnet host fetches directly by default, or through a configured
  proxy, or through the verified Tor route if the source sets
  `require_tor: true`. A `require_tor` source never falls back to a
  direct or proxy route after a Tor failure.

Describe sources as a JSON list:

```json
[
  {
    "name": "case1",
    "base_url": "https://source.example/case1/ALL_FILES",
    "require_tor": false,
    "headers": {"Accept": "text/plain"},
    "credential_header": "Authorization",
    "credential_env": "CASE1_TOKEN"
  }
]
```

A credential header's value is read from the named environment variable at
fetch time only; the source list and every stored snapshot record only the
header's name, never its value.

```bash
export CASE1_TOKEN=...
python3 src/inventory_fetch.py fetch --sources /case/sources.json \
    --store /case/inventory
```

Each snapshot's `snapshot.json` records the route that served it
(`tor`, `direct`, or `proxy:<name>`), the final URL and redirect chain,
response header names in received order, and permitted outgoing metadata.
Response values outside the safe list are redacted. The
[transport contract][transport] defines the redaction policy. See
[the discovery specification](../specs/SPEC-inventory-discovery.md).

## Next steps

Follow the [custody walkthrough](CHAIN-OF-CUSTODY.md) when reviewing a run.
Use the [audit guide](AUDIT-RAIL.md) to interpret verification results.

[schemas]: ../src/provenance/schemas/registry.json
[open-work]: ../specs/OPEN-WORK.md
[case]: #prepare-the-case-and-context
[transport]: ../specs/SPEC-http-transport.md
[sealed]: ../specs/SPEC-sealed-multi-recipient.md
