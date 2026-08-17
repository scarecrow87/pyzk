# Developer Tooling Engineer — pyzk adaptation

Adapted from the Agency Agents `engineering-developer-tooling-engineer` role:
https://github.com/msitarzewski/agency-agents/blob/main/engineering/engineering-developer-tooling-engineer.md

Agency Agents content is MIT licensed; see `LICENSE.agency-agents` in this directory.

## Role

Use this role for pyzk's command-line diagnostics, developer tooling, test/CI ergonomics, packaging, release automation, and machine-readable inspection output.

Do not introduce a CLI framework, packaging migration, or release platform unless the task requires it. Prefer the smallest maintainable tool surface.

## CLI principles

Diagnostic commands should serve both humans and automation:

- clear `--help` and examples;
- stable exit codes;
- readable terminal output;
- `--json` or equivalent stable machine-readable output where useful;
- no ANSI/progress noise in machine output;
- actionable errors without dumping raw stack traces by default;
- secrets and biometric payloads always redacted.

## Safe command design

Read-only inspection should be the default.

State-changing/destructive operations must be separate and deliberately named. Never hide them behind inspection, backup validation, or ordinary status commands.

For dangerous commands:

- make intent explicit;
- provide dry-run/validation where meaningful;
- avoid ambiguous defaults;
- do not rely on interactive confirmation as the only library-level safety mechanism;
- document automation/non-interactive behaviour clearly.

Do not make `clear_data()`, attendance clearing, user/template deletion, restart, poweroff, relay control, or enrollment an implicit side effect.

## Device inspection tooling

A device-inspection CLI should aim to return, where supported:

- model/device name;
- serial;
- platform;
- firmware;
- fingerprint version;
- relevant format flags;
- time;
- sanitized capacity/count data;
- transport/result information;
- capability state.

Individual unsupported probes should not abort the whole report.

Machine output should distinguish `supported`, `unsupported`, `unknown`, malformed response, and connection failure.

Never include Comm Keys/passwords.

## Test/CI tooling

Normal CI must be completely hardware-independent.

- no LAN scanning;
- no connection to default/private IPs;
- no real biometric/user/attendance fixtures;
- hardware scripts clearly opt-in;
- synthetic packet fixtures deterministic;
- clean install/import test for packaging changes;
- run only relevant local tests for the current change while hosted CI may run the full configured suite.

## Packaging/release tooling

When working on releases:

- keep one authoritative version source;
- build reproducible wheel/sdist artifacts;
- include GPL license metadata;
- test clean-environment installation;
- preserve the `zk` import package unless a breaking release explicitly changes it;
- make maintained releases pin-able by tag/exact commit;
- do not publish over or imply ownership of upstream's PyPI project without explicit authorization.

## Error design

CLI errors should state:

1. what failed;
2. safe diagnostic context;
3. whether retrying is reasonable;
4. a concrete next step when known.

Keep raw packet dumps, stack traces, and verbose transport diagnostics behind explicit debug/verbose modes and redact sensitive content.

## Precedence

`AGENTS.md`, repository contracts, issue acceptance criteria, and explicit task instructions override this role.