# Development guide

[Documentation](README.md) / Develop

[banner]: docs/assets/tod-dl-banner.svg

Start with the behavior contract and its owning component. The
[architecture guide](ARCHITECTURE.md) maps responsibilities; the
[specification guide](SPECIFICATIONS.md) groups the contracts by subject.

## Prepare the environment

Run these commands from the repository root. CI uses Python 3.12; use that
version when reproducing its environment. Dependency files pin the Python
packages used by the downloader and monitor.

```bash
python3 -m venv .venv
. .venv/bin/activate
python3 -m pip install -r requirements-downloader.txt \
    -r requirements-monitor.txt
```

Read [repository instructions](../AGENTS.md) before changing code. Keep
runtime data in temporary or separate case directories.

## Choose a code-reading path

Follow the behavior from its entry point to the relevant tests.

| Area | Source | Tests |
| --- | --- | --- |
| Acquisition and recovery | [Controller][ref-1] | [Behavior][ref-2] |
| Interrupted finalization | [Storage][ref-3] | [Recovery][ref-4] |
| Canonical signed bytes | [JCS][ref-5] | [Canonicalization][ref-6] |
| Inventory lineage | [Rederivation][ref-7] | [Lineage][ref-8] |
| Handover | [Exporter][ref-9] | [Export][ref-10] |
| External timestamps | [Verifier][ref-11] | [Acceptance][ref-12] |
| Descriptor transport | [Adapter][ref-13] | [Transport][ref-14] |
| Attached console and resume | [CLI][ref-15] | [Lifecycle][ref-16] |

Patch helpers in their owning module when writing tests. For example, patch
Tor helpers in `downloader.tor` and hash helpers in `downloader.util`.
The CLI module is not their owner.

## Run focused checks

Run the modules that cover the changed behavior. For example, when changing
handover export:

```bash
python3 -m unittest tests.test_export_handover -v
```

For deterministic canonicalization checks:

```bash
python3 -m unittest tests.test_jcs -v
```

Add focused coverage for recovery, finalization, control, or scheduling
changes. Test names must state why the behavior matters. Use synthetic
bytes, fixed clocks, and temporary paths. Pin `TZ` when testing displayed
time. Follow [AGENTS.md](../AGENTS.md) for the full testing policy.

Run the relevant syntax and whitespace checks before handing off changes:

```bash
python3 -m py_compile src/*.py
bash -n ./run.sh
git diff --check
```

## Understand CI scope

The [CI workflow](../.github/workflows/ci.yml) runs the default suite,
Python and Bash syntax checks, and a secret scan. The suite command is:

```bash
python3 -m unittest discover -s tests -p 'test_*.py' -v
```

The repository reserves local full-suite runs for an explicit request.
During development, use focused modules and leave full-suite validation to
CI. Report skipped tests and environment restrictions with the result.

## Exercise the local fixture harness

The fixture self-test uses a loopback server and synthetic bytes. It starts
no acquisition engine or source request. Choose a new output directory:

```bash
python3 src/evaluation/acquisition_evaluation.py \
    --self-test --output /tmp/tod-dl-evaluation
```

The output directory must be empty. This self-test validates infrastructure;
it does not repeat the engine selection evaluation. Read the
[evaluation contract](../specs/SPEC-acquisition-tool-evaluation.md) before
running engine scenarios.

## Maintain the documentation

Keep explanations in `docs/`, behavior contracts in `specs/SPEC-*.md`, and
dated validation evidence in `specs/reports/`. Link to the owner of a value
or rule instead of copying schema digests, defaults, or status inventories.

Update operator instructions when public behavior changes. Label planned
features explicitly. Keep historical results tied to their report dates.
Use [the publication proposal](SPECIFICATIONS.md#publication-proposal)
when preparing HTML or PDF editions.

## Next steps

Choose a behavior in [open work](../specs/OPEN-WORK.md), then read its spec,
source owner, and tests before proposing an implementation.

[banner]: docs/assets/tod-dl-banner.png
[ref-1]: ../src/downloader/core.py
[ref-2]: ../tests/test_tod_dl.py
[ref-3]: ../src/downloader/storage.py
[ref-4]: ../tests/test_v2_finalization_recovery.py
[ref-5]: ../src/provenance/jcs.py
[ref-6]: ../tests/test_jcs.py
[ref-7]: ../src/provenance/rederive.py
[ref-8]: ../tests/test_inventory_rederivation.py
[ref-9]: ../src/provenance/export_handover.py
[ref-10]: ../tests/test_export_handover.py
[ref-11]: ../src/provenance/timestamp_verify.py
[ref-12]: ../tests/test_timestamp_acceptance.py
[ref-13]: ../src/downloader/descriptor_transport.py
[ref-14]: ../tests/test_descriptor_transport_acceptance.py
[ref-15]: ../src/tod_dl.py
[ref-16]: ../tests/test_control_launch_and_resume_script.py
