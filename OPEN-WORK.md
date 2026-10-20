# Open development work

This register lists current work that lacks a completion record. The linked
specifications define requirements. Dated reports define completed evidence.
This file does not authorize a source request, an unattended run, or a
production deployment.

For the owner of a topic, see the [specification index](INDEX.md).

## Operational validation

The earlier aria2 configuration passed local E01 through E14 scenarios.
Queue A passed its limited source pilot and resume checks. That evidence does
not validate the current native engine. The following operator-run evidence
remains before unattended production use:

- Run queues B and C with approved case inputs and separate state paths.
- Run one bounded production acquisition with an explicit `--max-files` value.
- Check each base URL against a known working URL before a run. A wrong base
  URL can return HTTP 404 for every selected item.

See [SPEC-reliable-acquisition.md](SPEC-reliable-acquisition.md) and
[SPEC-acquisition-tool-evaluation.md](SPEC-acquisition-tool-evaluation.md).

## Non-forensic download acceptance

The unsigned URL-queue controller and explicit ETag override are available
under [the non-forensic contract](SPEC-non-forensic-downloads.md). The
[implementation report](reports/non-forensic-downloads-2026-10-03.md) records
focused synthetic fixture evidence. Complete isolated-route acceptance,
private-writer process kills, concurrent controls, resource exhaustion, and
scheduling matrices remain open. CI must run the full suite. The specification
remains partially implemented until its acceptance requirements are complete.
Native cutover acceptance and the operator pilot remain open.

## Worker-limit verification

Live fixed and automatic worker limits are implemented. The
[implementation report](reports/dynamic-worker-limits-2026-10-03.md) records
focused fixture checks. CI must run the full suite. Native acceptance and
the operator pilot remain open under the native cutover contract.

Descriptor acquisition keeps its existing rollout gates. Isolated descriptor
children publish payload counters at completion; feedback windows hold when
required fresh samples are missing. Automatic parallel ranges are partially
implemented under
[the specification](SPEC-automatic-parallel-ranges.md). Connection admission,
private segment storage, authenticated assembly, and bounded-gap restart have
focused fixture evidence. Complete the specification's fault, scheduling,
route, resource, and lifecycle matrices. CI, Tor acceptance, and the operator
pilot remain open. See the
[implementation report](reports/automatic-parallel-ranges-2026-10-03.md).
Equal-second restart ordering, repeated assembly and preservation recovery,
private-writer process kills, and multi-child controls have focused fixture
evidence in the implementation report. Startup and the shared earlier-event
reader follow signed run-index session order. Integrated loopback worker-limit
increases and decreases have focused evidence in the implementation report.
Complete the remaining eligibility, integrated rechecks, queue fairness,
route acceptance, and resource checks before a parallel implementation
completion claim.

## Descriptor console details

The descriptor projection and views are owned by
[inspection](SPEC-console-inspection.md#descriptor-detail-projection),
[item details](SPEC-console-item-details.md#descriptor-item-details), and
[worker details][descriptor-workers].
Implemented on October 3, 2026: the durable-field projection, attempt and
generation binding for runtime samples, attempt validation history, unknown
worker slots, and the descriptor item and worker view models. Focused tests
are `tests/test_descriptor_inspection.py` and
`tests/test_descriptor_console_details.py`.

Implemented on October 4, 2026: `list_queue` fills `review_code` and
`retry_at` (only in `retry_wait`) for descriptor rows. `get_item` reports
retained item bytes from the generation's promotion intent and committed
completion bytes only when the item is `complete`.
`tests/test_descriptor_console_screens.py` covers selection on return to Queue
and the queue retry deadline.

Implemented on October 4, 2026: `descriptor_downloads.candidate_path` and
`descriptor_download_attempts.validated_at` use the guarded `ALTER TABLE`
migration in `open_descriptor_state`. `get_item` reads them. They are SQLite
state columns, not signed record fields, so the manifest evolution contract
does not apply. `validated_at` uses `now()`, which writes RFC 3339 with `Z`.

Implemented on October 4, 2026: descriptor queue rows fill `last_change_at`
from the newest valid transition or attempt end. A Textual screen test covers
the worker details screen with a bound descriptor assignment. Descriptor
queue rows and attempt history report the recorded `error_category`.

Implemented on October 4, 2026: the controller supplies descriptor control
advice and the request projection. Tests cover both over the inspection
socket and in the item details screen. A run with a live controller and
monitor remains unverified.

Existing generic console tests do not establish these descriptor
requirements. Public acquisition, real Tor/proxy checks, and complete
terminal acceptance remain unverified.

## Version 2 HTTP transport

Phase 1 transport controls are complete. Version 2 source-policy validation,
request descriptors, and inventory request mapping are implemented. Public
descriptor acquisition remains blocked. Controller startup uses the item-keyed
session behind the rollout refusal. The following work remains:

- Finish r5. `--assurance-profile FILE` deviates from
  SPEC-forensic-compliance.md, decided 2026-09-29: the option is optional (the
  spec makes it required for every r5 run, but no `forensic-baseline`
  artifact is defined); there is no `--profile-root`, installation record,
  register, or release evidence check; the verifier accepts the
  `assurance_profile` input role at most once and checks the artifact values,
  but does not require it.
- Close the descriptor body-integrity rollout gates in
  [SPEC-response-body-integrity.md](SPEC-response-body-integrity.md) and
  [SPEC-http-transport.md](SPEC-http-transport.md). Publish the adapter
  capability matrix and all required T01 through T10, T11, T14, T16, and
  RI14 through RI17 results before enabling the affected features. No adapter
  requests a range, so only fixtures exercise the 206 binding rules.
  Controller wiring is complete; see the
  [controller integration report][descriptor-controller].
- Close the proxy and Tor test gaps. No test reaches an outside proxy or Tor.
  Most tests run the real worker child under a stand-in wrapper that only
  drops `-i`. One test runs the real `torsocks` against a loopback SOCKS5 stub.
- Complete native acceptance under
  [SPEC-native-engine-cutover.md](SPEC-native-engine-cutover.md). Record a
  dated report for E01 through E14, the engineering targets, N01 through N15,
  R01 through R07, and removal cases N16 through N18. The complete matrix,
  including socket-based cases, remains unverified.
  Use the [selective acceptance collection](../acceptance/README.md) for
  preserved collectors and named reruns. The
  [collector preservation report][native-collector-preservation] records
  verified selections, missing replay inputs, and archived-only scripts.
  Rebuild the missing synthetic R05 inputs and provide a portable selection
  for the earlier anchored E verifier.
  Complete the remaining N/R and removal matrices, including complete
  archived-case verification. Use the
  [native resume acceptance report][native-resume-acceptance] to identify
  coverage gaps, and check the later
  [native resume fixes report](reports/native-resume-fixes-2026-10-01.md) and
  [native fixture acceptance report][native-fixture-acceptance].
  The [single-build E matrix][native-single-build] records passing E01 through
  E14 assertions on unchanged `812168c`, including both large-file resume
  modes and the million-row resource check. Complete N/R and removal
  acceptance remains open. The
  [live telemetry and controls report][native-live-controls] adds fresh
  direct-hop byte equality, exact engine identities, timed controls, and four
  anchored closed runs. Connection-establishment timeouts, resumed and Tor
  counters, aggregate telemetry, and the remaining route and fault matrices
  still need local fixture evidence.
  The [current-build continuation report][native-continuation]
  adds local SOCKS5 forwarding, controller matrices, and complete synthetic
  earlier-schema verification. Four synthetic control runs also pass closed-run
  verification after restart. Tor circuit forwarding, resumed and Tor byte
  equality, and remaining fault variants remain open. Synthetic archived-script
  working copies resume with explicit and default native selection. Historical
  operator scripts and the complete removal matrix remain unverified.
- Complete the interrupted native operator pilot under the queue and
  route restrictions in
  [SPEC-native-transfer-engine.md](SPEC-native-transfer-engine.md). Use the
  exact host and file scope established by a successful bounded acquisition.
  Record Range requests and observed responses as the cutover specification
  requires. A pilot without interruption provides no source-resume evidence.
  Read the [source evidence and next-agent handover][native-evidence]
  before selecting the pilot. The
  [bounded pilot report](reports/native-operator-pilot-2026-10-01.md) records
  six failures with no source bytes. Interrupted resume remains unverified.
- Close the HTTP onion rollout gate. Validation and internal body acquisition
  use signed origins and CA trust checks under the active schema. Public
  descriptor acquisition
  remains blocked. See the
  [T18 report](reports/http-onion-descriptors-t18-2026-10-01.md)
  for local checks and gaps.
- Implement the sealed-sidecar export policy and `require-include` behavior
  in SPEC-sealed-sidecar-handover.md (`r6`). Then implement the derived-set
  re-encryption tool in SPEC-sealed-re-encryption.md. It depends on format 3.

See [SPEC-http-transport.md](SPEC-http-transport.md) and
[PLAN-manifest-custody-remediation.md](PLAN-manifest-custody-remediation.md).

## Inventory and rescan work

Inventory parsing, manifests, diffs, queue export, and configured listing fetch
are implemented. The remaining work is:

- Add scheduled listing refresh only after basic acquisition is reliable.
- Add directory crawling only after basic acquisition is reliable.
- Run a real listing-pair check for a candidate manifest from `diff --policy`.
  Candidate-manifest items have no expected checksum.
- Define a source of checksum sidecars before relying on
  `manifest --checksums FILE` for checksum verification.
- Implement target-directory rescan, including a manual command, debounced
  watcher, dated report, and candidate queue for missing or mismatched files.

The parser cannot represent a file name that contains a newline. Import also
rejects listings above `MAX_FILES` (2,000,000). These are documented limits,
not implementation defects.

See [SPEC-inventory-discovery.md](SPEC-inventory-discovery.md),
[SPEC-inventory-snapshot-manifest.md](SPEC-inventory-snapshot-manifest.md), and
[SPEC-target-dir-rescan.md](SPEC-target-dir-rescan.md).

## Schema follow-ups

The September 27, 2026 schema review added the inventory `-v1-r2` schemas
and the provenance v2 `r3` revision. These items remain open:

- Close the free-text error slots in a later v2 revision. The `local_failure`
  `reason_code` and `request_observation.error_code` accept any string. The
  descriptor adapter writes only `descriptor_transport_error`.
  SPEC-reliable-acquisition.md defers the full categorized-error value set.
  A closed enum needs the complete list of values the writer produces.
- Decide whether a v2 relative path may contain control characters. The
  path pattern excludes only NUL, `\`, and `/` as the last character.
- Consider `if`/`then` discriminators instead of large `oneOf` blocks for
  shorter validator errors, and add `examples`. Both change the schema
  digest, so batch them with the next required revision.
- Require local block storage for the SQLite state and staging directories,
  and refuse NFS or SMB mounts at startup. SQLite WAL mode and exclusive
  locks assume local POSIX locking. No such check exists. The planned
  [storage preflight](SPEC-local-storage-preflight.md) owns the contract.
- Consider a global bandwidth cap for concurrent native transfers.
- Event times keep `time_assurance: "host_clock"`. Independent time evidence
  comes from RFC 3161 checkpoint timestamps; see
  [SPEC-external-timestamping.md](SPEC-external-timestamping.md).

## Speed history and visualization

Planned on October 4, 2026. Item 1 is implemented in
`src/downloader/speed_history.py` (recorder, store, ring) and
`src/downloader/speed_runner.py` (recorder thread, declaration, events), with
tests in `tests/test_speed_history.py` and `tests/test_speed_wiring.py`. Open
for item 1: no test runs a full acquisition-mode controller with history, so
the signed-record check there is a provenance import check only. An
admission pause that the storage reserve causes records no event. Items 2 and
3 have no code.
The
[design concept](reports/speed-visualization-concept-2026-10-04.md) records
the reasons. Implement in this order, each with its own test plan:

1. Controller recorder, history file, and retention under
   [SPEC-speed-history.md](SPEC-speed-history.md).
2. Inspection operations, reference runs, and the offline report under
   [SPEC-speed-history-access.md](SPEC-speed-history-access.md).
3. Console views under
   [SPEC-console-speed-visualization.md](SPEC-console-speed-visualization.md),
   the charts section of
   [SPEC-console-visual-style.md](SPEC-console-visual-style.md), and the
   speed-view keys in [SPEC-console-keymap.md](SPEC-console-keymap.md).

The "Decisions" sections of the three speed specs record the operator
decisions. The bin width and the 30-sample minimum need review against
real onion throughput data.

## Remaining specification work

Python CI coverage reporting is partially implemented. Complete hosted CI
acceptance for forced artifact upload failure under
[SPEC-ci-python-coverage.md](SPEC-ci-python-coverage.md).

The following specifications remain planned or partially implemented. Each
specification is the source of its detailed acceptance criteria:

- Console UI, inspection, item details, keymap, visual style, controller
  controls, and download telemetry.
- Manifest version 2 evolution, verification hardening, and run custody:
  session context, body-byte coverage, preservation profiles, and remaining
  verifier and custody requirements.
- Response-body integrity assessment and preservation-profile support.
- Forensic-compliance and evidence-preservation profiles. The project has no
  approved legal or organization-specific compliance profile.

See [SPEC-console-ui.md](SPEC-console-ui.md),
[SPEC-download-telemetry.md](SPEC-download-telemetry.md),
[SPEC-manifest-v2-evolution.md](SPEC-manifest-v2-evolution.md),
[SPEC-manifest-verification-hardening.md][hardening],
[SPEC-run-custody.md](SPEC-run-custody.md),
[SPEC-response-body-integrity.md](SPEC-response-body-integrity.md),
[SPEC-forensic-compliance.md](SPEC-forensic-compliance.md), and
[SPEC-evidence-preservation.md](SPEC-evidence-preservation.md).

## Status reference

The repository records the current status in each `SPEC-*.md` file. Do not add
completed fixes, test history, or report summaries here. Add an item only when
an approved specification requirement lacks its required evidence.

[hardening]: SPEC-manifest-verification-hardening.md

[native-evidence]:
  reports/native-acquisition-handover-2026-10-01.md

[native-resume-acceptance]:
  reports/native-resume-acceptance-2026-10-01.md
[descriptor-controller]:
  reports/descriptor-controller-integration-2026-10-01.md
[native-continuation]:
  reports/native-acceptance-continuation-2026-10-02.md
[native-fixture-acceptance]:
  reports/native-fixture-acceptance-2026-10-02.md
[native-single-build]:
  reports/native-single-build-e-matrix-2026-10-03.md

[native-live-controls]:
  reports/native-live-telemetry-controls-2026-10-03.md

[native-collector-preservation]:
  reports/native-collector-preservation-2026-10-03.md

[descriptor-workers]:
  SPEC-console-worker-details.md#descriptor-workers-and-unknown-assignment
