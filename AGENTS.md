# AGENTS.md

This file applies to the entire repository.

## Project purpose

This repository is a maintained fork of [`fananimi/pyzk`](https://github.com/fananimi/pyzk), an unofficial Python library for communicating with ZKTeco-compatible standalone attendance and biometric terminals.

Treat this repository as a standalone reusable library. Do not add details, URLs, architecture notes, issue references, credentials, or private implementation information from downstream applications that consume this package.

## Engineering priorities

When priorities conflict, prefer:

1. protocol/data correctness;
2. device and API compatibility;
3. operator and biometric-data safety;
4. clear failure behaviour;
5. maintainability;
6. convenience.

Never manufacture plausible data to hide malformed device responses.

## Upstream relationship

Original upstream:

- https://github.com/fananimi/pyzk

Before implementing protocol or device-specific behaviour:

1. inspect the current fork code;
2. check relevant local issues;
3. check upstream issues, pull requests, and merged commits for prior art;
4. preserve attribution/provenance for ported fixes;
5. add regression coverage before or with the behaviour change.

Do not blindly merge an upstream change just because it exists. Review correctness, device assumptions, safety, and compatibility first.

Preserve the existing GPL-2.0 licensing, copyright notices, and Git history.

## Repository map

Important areas include:

- `zk/base.py` — protocol transport, device commands, packet handling, user/attendance/template/device operations.
- `zk/const.py` — protocol constants, commands, flags, event values, and sizes.
- `zk/exception.py` — public exception types.
- `zk/user.py` — user representation.
- `zk/finger.py` — fingerprint-template representation.
- `zk/attendance.py` — attendance representation.
- `example/` — usage examples, including destructive examples that must not be treated as automated tests.
- `docs/` — library/protocol/device documentation.
- `test_machine.py` and similar scripts — hardware/manual diagnostics; do not assume they are safe CI tests.

Before changing a protocol function, trace its command constants, packet structure, response parsing, public API callers, and cleanup behaviour.

## Device capability rules

ZKTeco-compatible terminals vary by model, platform, firmware generation, fingerprint algorithm, and packet format.

Use these states explicitly when introducing capability-aware behaviour:

- `supported` — documented or proven for the relevant device/profile;
- `unsupported` — known not to work or not applicable;
- `unknown` — not proven safely.

A method existing in `ZK` does not prove a connected device supports it.

For write or destructive operations, `unknown` must not be silently treated as `supported`.

Prefer isolated device profiles, parser variants, or capability checks over large model-name condition chains scattered through `zk/base.py`.

## Real-device safety

Real-hardware testing is opt-in and should start read-only.

Do not run state-changing or destructive commands against physical hardware unless the task explicitly requires that exact operation and the operator has intentionally authorized it.

High-risk operations include, but are not limited to:

- `clear_data()`;
- `clear_attendance()`;
- `delete_user()`;
- `delete_user_template()`;
- `save_user_template()` / high-rate template upload;
- `set_user()`;
- `set_time()`;
- `restart()`;
- `poweroff()`;
- relay/door unlock operations;
- network/configuration writes;
- remote fingerprint enrollment.

Do not treat `disable_device()` as a guaranteed physical safety lock. Some devices acknowledge the command but continue allowing local activity.

Always restore device state and disconnect cleanly through `try/finally` or equivalent cleanup where appropriate.

## Biometric, credential, and personal-data handling

Never commit or expose:

- real fingerprint templates;
- fingerprint images;
- communication/Comm Keys or device passwords;
- real user passwords;
- identifiable user datasets;
- identifiable attendance datasets;
- sensitive device network configuration that is not required for a sanitized compatibility report.

Use synthetic fingerprint/template bytes and sanitized packet fixtures in tests.

Secrets must not appear in:

- exception strings;
- debug/verbose output;
- `repr`/`str` output;
- CLI inspection output;
- test snapshots;
- GitHub issue/PR artifacts.

## Protocol and parser changes

For binary protocol changes:

- verify endianness, signedness, width, offsets, packet lengths, and terminators;
- validate ranges before packing values;
- consume exactly the documented/detected record length;
- handle truncated/unknown packets explicitly;
- retain safe raw diagnostic context where useful without dumping secrets or biometric payloads;
- do not silently reinterpret unknown formats as a known layout;
- preserve existing public semantics for already supported packet formats unless a breaking change is explicitly approved.

Malformed attendance timestamps must not be silently clamped into a different plausible timestamp. Surface invalid data clearly and preserve valid records where possible.

## Public API compatibility

This is a library used by downstream consumers.

Before changing a public method, argument, return type, exception, field, CLI flag, or machine-readable output:

- identify existing behaviour;
- prefer backwards-compatible additions;
- add deprecation paths for unavoidable breaking changes;
- document the change;
- include regression tests for the old supported behaviour and new behaviour.

Do not casually rename the `zk` import package or existing public classes/functions.

## Testing

Automated tests must be hardware-independent by default.

- Use mocks, synthetic binary fixtures, and deterministic responses.
- CI must never scan a LAN or connect to a physical attendance terminal.
- Keep physical-device tests/manual probes clearly opt-in and separate from normal CI.
- Test protocol boundary values, malformed packets, unsupported responses, cleanup after exceptions, and secret redaction where relevant.
- Local validation should be scoped to the functionality/files changed by the current task; hosted CI may run the repository's full configured regression suite.
- Do not weaken assertions merely to make a device-specific patch pass.

When adding CI, prefer a small supported Python-version matrix and clean-environment installation checks over unrelated tooling churn.

## CLI and inspection tooling

For diagnostic/inspection commands:

- default to read-only behaviour;
- provide useful human-readable output;
- provide stable machine-readable JSON where appropriate;
- use meaningful exit codes;
- report `unsupported` and `unknown` explicitly;
- never print device passwords/Comm Keys;
- keep destructive actions separate from inspection commands;
- require deliberate invocation for state-changing actions.

## Packaging and releases

Do not publish over or imply ownership of the original upstream PyPI project without explicit authorization.

Maintained releases should be reproducible and pin-able by tag or exact commit. Preserve GPL metadata/license files in distributions.

Changes to packaging/versioning should not be mixed with unrelated protocol behaviour unless the issue explicitly requires both.

## Documentation and compatibility evidence

When documenting a tested device, record only sanitized information needed for compatibility work, such as:

- device/model name;
- platform;
- firmware version;
- fingerprint algorithm/version;
- relevant packet/record sizes;
- transport result;
- tested library commit/version;
- per-capability result.

Separate proven facts from assumptions. Never publish credentials, real user records, real attendance records, or biometric templates.

## Specialist agent guidance

Project-adapted specialist guidance lives in `.github/agents/`.

Use the smallest relevant set:

- `.github/agents/engineering-codebase-onboarding-engineer.md` — read-only repository exploration, architecture/code-path tracing, and understanding inherited protocol behaviour before edits.
- `.github/agents/engineering-backend-architect.md` — public library API design, capability modelling, exception design, compatibility boundaries, and larger protocol architecture decisions.
- `.github/agents/engineering-code-reviewer.md` — PR/diff review for correctness, packet safety, backward compatibility, resource cleanup, tests, secrets, and destructive-operation risk.
- `.github/agents/engineering-developer-tooling-engineer.md` — inspection/diagnostic CLI design, packaging/release tooling, CI ergonomics, machine-readable output, and safe command interfaces.

These role files are advisory. **This `AGENTS.md`, issue acceptance criteria, repository code/contracts, and explicit task instructions take precedence over generic role guidance.**

Do not import generic agent requirements that do not fit a low-level Python hardware/protocol library (for example database, cloud, microservice, frontend, or UI requirements) unless the current task actually needs them.

## Scope discipline

Keep fixes minimal and issue-focused.

Do not combine unrelated refactors, dependency upgrades, formatting sweeps, API redesigns, packaging migrations, and device fixes into one change without a clear reason.

If a requested feature depends on uncertain device/protocol behaviour, stop at the safe boundary, document what remains unknown, and create/point to a focused compatibility task rather than guessing.