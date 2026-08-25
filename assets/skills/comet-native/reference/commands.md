# Native command and exception reference

During the normal flow, execute the command returned in Runtime `continuation`. This file explains returned fields and handles these cases: command input is rejected, verification execution fails, verification cannot decide because external information is missing, or the Runtime asks the user to confirm the verification result. `continuation.disposition` says whether to continue, wait for the user, resolve a blocker, or finish. Use a follow-up command containing `--confirmed` only after explicit user confirmation.

Treat CLI help as authoritative for command signatures and current arguments:

```text
comet native --help
comet native <command> --help
comet native <group> <command> --help
```

## Next action returned by the Runtime

- `disposition`: whether to continue, wait for the user, resolve a blocker, or finish.
- `commandArgs` / `commandAlternatives`: complete command arguments from the Runtime. Alternatives are complete commands for mutually exclusive user decisions; execute the matching one and do not combine them.
- `inputOptions`: fields and a JSON template for this command.
- `workspace` / `preparation`: the actual working directory and change-creation result.
- `stateVersion` / `loop`: the current state version and acceptance Loop progress.
- `acceptance` / `children` / `readyChildren` / `nextPageArgs`: the acceptance summary, Supervisor Change child projections, currently startable children, and command for the next page.
- `verifierDispatch`: the inputs the current model needs to complete this verification round, including acceptance items, brief and Spec references, the Builder handoff, and Runtime check results.
- `workspaceFinishResult` / `recoveryArgs`: the post-Archive workspace result and recovery command.

Angle brackets in a template mark values to fill in. `await-user` means wait for the user's decision before running an advancing command. If `commandArgs` is `null` and `commandAlternatives` is present, confirm the user's decision, then execute the selected alternative's complete `commandArgs` while preserving `--expected-state-version` and `--expected-action`. If the command fails because the state or action binding is stale, reread the latest `continuation` and continue from the current state; do not construct an unguarded replacement command. `localExecution: absent` means only that this machine has no currently running local task; it does not mean the change is damaged.

## Fill command input

Copy `inputOptions.template` into a temporary system JSON file, replace only the requested values, then execute `continuation.commandArgs` or the selected `commandAlternative.commandArgs`. Delete the temporary file afterward. Preserve the acceptance iteration, verification attempt, state version, and task identifiers already present in the template. Fill only the fields exposed by the template.

- `builder-handoff`: submit the implementation summary for this round, addressed acceptance IDs, development checks the Builder actually ran, and known limitations. Leave acceptance conclusions to the Verify phase.
- `dispatch-verifier`: list the checks the Runtime should execute for the current candidate. Submit an empty list when no command-based check applies.
- `verifier-response`: request additional checks or submit a final result that covers every acceptance ID.
- `verifier-execution-error`: report that this verification round could not be completed. Preserve the task-binding fields from the template so a late message from an old task cannot affect a new verification round.
- `verifier-unavailable`: used only to recover an independent Verifier attempt left in historical state; the current flow does not submit it.

The Runtime executes and records verification checks. Development checks listed in the Builder handoff only describe the candidate; acceptance conclusions rely on actual Runtime check results. The latest `continuation` decides whether to add checks, retry, or start a new verification round.

## Exceptional cases

- Verification is temporarily unable to decide (`semantic blocked`): if only user or external information is missing, execute the resolution action returned by the Runtime. If the implementation must change, return to Build.
- Verification reports all items passed (`skill-coordinated pass`): the judgment comes from the current model rather than an independent execution, so when the Runtime requires user confirmation, explain that verification boundary once, then execute the returned command after confirmation.
- Verification execution fails (`execution error`): submit the error using the template, then read the new `continuation`. The Runtime decides which checks to reuse and whether to retry.

## Diagnostics

Run read-only `doctor` first. Execute a repair command only when `doctor` explicitly returns one; the Runtime continues to manage locks, cross-device state, and transactions.
