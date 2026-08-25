# Native command and exception reference

During the normal flow, execute the command returned in Runtime `continuation`. This file only explains returned fields and rejected input. `continuation.disposition` says whether to continue, wait for the user, resolve a blocker, or finish. Use a follow-up command containing `--confirmed` only after explicit user confirmation.

Treat CLI help as authoritative for command signatures and current arguments:

```text
comet native --help
comet native <command> --help
comet native <group> <command> --help
```

## Next action returned by the Runtime

- `disposition`: whether to continue, wait for the user, resolve a blocker, or finish.
- `commandArgs` / `commandAlternatives`: complete command arguments returned by the Runtime.
- `inputOptions`: fields and a JSON template for this command.
- `workspace` / `preparation`: the actual working directory and change-creation result.
- `stateVersion` / `loop`: the current state version and execution progress.
- `acceptance` / `children` / `readyChildren` / `nextPageArgs`: the acceptance summary, Supervisor Change child projections, currently startable children, and command for the next page.
- `workspaceFinishResult` / `recoveryArgs`: the post-Archive workspace result and recovery command.

Angle brackets in a template mark values to fill in. `await-user` means wait for the user's decision before advancing. With `commandAlternatives`, execute the matching complete arguments and preserve `--expected-state-version` and `--expected-action`. If that binding is stale, reread the latest `continuation`.

## Fill Build and Verify input

Copy `inputOptions.template` into a temporary system JSON file, replace only the requested values, run `continuation.commandArgs`, then delete the file.

- `builder-handoff`: preserve `validation_mode: model`, then submit the implementation summary, addressed acceptance IDs, development checks, and known limitations. Runtime enters Verify performed by the current model.
- `verification-result`: after the current model completes its semantic review, submit the validation summary, the per-item acceptance verdicts (each `acceptance` entry provides `id`, `result`, and a traceable `reason`), the Runtime check plan, and known limitations. Each check provides `id`, `name`, `executable`, `argv`, `cwdRef`, `timeoutMs`, and `repeatable`; use an empty array only when no command check applies and explain that in the summary.

Runtime executes and records these checks. A missing acceptance item, any non-passing acceptance verdict, or any non-passing check prevents Archive. When everything passes, Runtime records a `self-validated` result and awaits the user's decision; after the user accepts, use `--accept-result` to enter Archive.

An existing change may still be waiting for an independent Verifier response. Runtime keeps the old inputs and recovery actions only to finish or retreat such attempts. New Verify rounds do not start an independent Verifier.

## Diagnostics

Run read-only `doctor` first. Execute a repair command only when `doctor` explicitly returns one; the Runtime continues to manage locks, cross-device state, and transactions.
