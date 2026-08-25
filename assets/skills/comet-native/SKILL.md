---
name: comet-native
description: "Comet Native workflow. Use when the user explicitly invokes /comet-native, asks to start or resume a Native change, or the entry router selects Native."
---

# Comet Native

Native stores the requirements, complete target specifications, current progress, and verification conclusions in the project. After completing each phase, return to the Runtime for the next action and handle only the phase it specifies.
## Inviolable boundaries

- The on-disk `.comet/config.yaml`, current change, `comet-state.yaml`, and formal Markdown are the working source; chat memory is only supplementary.
- The Runtime manages workflow state, local tasks, logs, locks, and transactions. Advance every phase through the public `comet native` commands on PATH; users do not run these commands manually.
- If a command is unavailable, report an incomplete Comet installation and stop. Treat `comet native <command> --help` as authoritative for arguments and output.
- The current model selects Verify validation proportional to the change risk, and Runtime executes and records the selected commands; do not start an independent Verifier subagent.
- This Skill and the Runtime complete the Native workflow; Native does not depend on any external Skill.
## Start or resume

1. When the change name is known, run `comet native status <change-name> --details --json` directly. Only run `comet native status --json` when the name is unknown, then query the selected change in detail.
2. Run pagination commands from `nextPageArgs` only when the current phase needs the complete acceptance list. Run `show` or read the corresponding brief/Spec only when editing or checking formal content.
3. When an active change already exists, enter the returned `workspace.projectRoot` and run `select`. Runtime scans registered Worktrees and prefers a workspace whose bound branch matches; ask the user only when multiple equally aligned candidates remain.
4. Create a change only when no matching active change exists, using the artifact directory from configuration.
### Create a change

Choose a lowercase kebab-case name, then use the [workspace selection reference](reference/workspace.md) to decide whether to use the current directory, create a branch, or create a worktree. Explicit parallel, simultaneous, or multi-session intent automatically selects `worktree` without asking for a three-way choice.

Before creating the change, the CLI binds the branch or worktree, reuses a registered change branch, recreates a Worktree when its branch still exists but the registered Worktree was removed, maintains repository-local exclusions, validates configuration, and creates state that can be resumed across devices. Then enter the returned `preparation.projectRoot`; do not continue subsequent commands in the original directory.

If preparation does not finish, keep the resources already created, show the failure reason from `preparation`, and continue with the recovery direction from the Runtime or user.
## Read on demand

After confirming the phase, read only the needed reference:

- Shape: always read and execute the [clarification reference](reference/clarification.md).
- Read the [artifact reference](reference/artifacts.md) when editing the brief or complete target specifications, or when reviewing the verification report.
- During normal progression, execute the command returned in Runtime `continuation`. Read the [command reference](reference/commands.md) only when a returned field is unclear or command input is rejected.
- Read the [recovery reference](reference/recovery.md) only when the task cannot continue because of an interrupted process, missing local Runtime state after moving devices, repeated lack of progress, a concurrency conflict, failed legacy migration, or damaged state.
## Shape

First investigate facts that can be determined from the repository, tools, and runtime environment. Independent fact-finding may be delegated to subagents. Follow `native.clarification_mode` and the clarification reference to maintain a decision tree. Ask the user only for choices that change the visible result and cannot be inferred reliably. When the user directly supplies a file, attachment, link, or local path as a requirements source, enter source-document full-coverage mode: read all accessible content and record its `complete`, `partial`, or `unavailable` status; chunking changes only read order and working-memory management, not the final coverage set; `brief.md` records the complete source requirements and coverage states before asking about ambiguity, omissions, or implicit boundaries; every executable source unit must map to both the complete target Spec and at least one acceptance ID, while background, non-goal, or superseded units retain only their classification, reason, and replacement relationship; mark a corrected old unit `superseded` and point it to its replacement; keep `partial`, `unavailable`, unmapped, or unconfirmed content `[blocking]`. Materials supplied only for debugging, evidence, review, or implementation reference do not trigger this mode automatically; clarify an unclear purpose first. A summary cannot replace the source coverage map.

Immediately synchronize confirmed user-visible decisions and important constraints into Decisions, the brief, and complete target specifications. Keep ordinary implementation choices in the implementation and tests unless they affect visible behavior. Acceptance items must be specific, observable, and non-duplicative. When a large requirement needs decomposition, maintain `children.yaml` at the Supervisor Change root, use `depends_on` for real ordering, and use `covers` to cover the Supervisor Change's readable acceptance index. Array order is only stable display order and the priority among equally ready children. New v2 `children.yaml` files index only brief-derived parent acceptance items; `acceptance_index` stores each ID's source and complete text. Spec-derived fine-grained checks remain in the Runtime's complete acceptance matrix and do not need to be duplicated in the initial child plan. Historical v1 files remain accepted under their original contract.

For a large requirement, run one decomposition preflight before the final Shape confirmation: recommend Supervisor Change mode only when workers can independently implement and verify at least two outcomes, acceptance items map completely, and real dependency or parallel value exists; keep a single Native change when the goal is tightly coupled, repeatedly edits one core area, coordination costs exceed independent verification, or the user asks for one change; text length and task count alone must not trigger decomposition.
When decomposition is recommended, put the `children.yaml` draft, execution waves, and coverage summary into one Shape confirmation; the user can confirm, adjust, or keep one. Do not create child changes before confirmation; do not create worktrees or dispatch Agents before confirmation.
After confirmation, the Skill automatically dispatches current ready children from Runtime; use parallel workers when available, otherwise use a serial fallback. Child scope inherits the parent confirmation; new user-visible decisions return to parent Shape.
On `/comet-native` resume, continue from Runtime state and do not duplicate existing children or worktrees.
Keep unresolved questions `[blocking]`; do not modify implementation while a blocker remains. Completion criterion: every choice that affects the visible result and every unstated assumption has been handled, no `[blocking]` item remains, the user has explicitly confirmed the outcome, scope, key decisions, acceptance items, and non-goals, and the Runtime has entered Build. Advance with the continuation containing `--confirmed` only after explicit user confirmation.

## Build

For the first implementation, read the current brief, complete target specifications, and all acceptance items, and change only the confirmed scope. When the candidate is ready, submit the concise Builder handoff from the Runtime template: the changes made, addressed acceptance items, development checks, and known limitations. An accepted handoff enters Verify.

One parent Shape confirmation authorizes strictly derived child changes, so do not ask the user to repeat the same scope. Execute only actions returned by Runtime continuation and reread `readyChildren` after each child completes.

When status contains `children`, the current change is a Supervisor Change: do not run a Supervisor Change Builder; advance only `readyChildren`. Every child is an ordinary Native change in an independent worktree targeting the Supervisor Change's `workspace.changeBranch`. Children without dependencies may Build in parallel, but Archive them one at a time with `finish=merge`. A child is `done` only after its Archive is merged; after every child is done, the Supervisor Change enters Verify. If Supervisor Change Verify fails, do not reopen archived children; follow `repair-child`, append a repair child covering the failed acceptance items, reconfirm Supervisor Change Shape, and continue.

Completion criterion: the implementation is ready for validation, the complete acceptance list has been rechecked, and the Runtime accepts the handoff and enters Verify.

## Verify

The current model selects the smallest sufficient validation based on the actual change risk; do not start a new Verifier subagent. Prefer focused tests covering the change. Include the corresponding build for compiler, Runtime, or generated-asset changes. Expand validation only for cross-module or high-risk changes. Do not mechanically select the full suite, and never skip checks for security, data loss, trust boundaries, or accessibility.

After checking every acceptance item, submit the validation summary, the per-item acceptance verdicts (each with `id`, `result`, and a traceable `reason` stating what the pass judgment is based on), the executable check plan, and known limitations with the Runtime `verification-result` template. Runtime executes the plan, records exit codes and logs, and rejects missing acceptance items, non-passing acceptance verdicts, or non-passing checks. Submit an empty `checks` array only when no command check applies, and explain that in the summary. Non-trivial logic must select one runnable check that fails on regression.

When a Runtime check fails or any acceptance item does not pass, return to Build and fix the implementation instead of bypassing the failure. When everything passes, Runtime records the result as `self-validated` and awaits the user's decision: present the validation conclusions to the user, and after the user accepts, follow the continuation and use `--accept-result` to enter Archive.

Classify requirement changes before proceeding: implementation omissions use `--revise-implementation` to return to Build; changes to visible behavior or acceptance criteria use `--revise-requirements`, update formal artifacts, and return to Shape for confirmation; unrelated requests belong to another change. Apply the same rule when the user explicitly adds to the current scope.

Completion criterion: risk-matched validation was actually run and recorded honestly, every acceptance item was checked with a per-item verdict, the user accepted the current result, and the Runtime advances to Archive.

## Archive

Continue only when `continuation` permits Archive. Archive uses the accepted verification result directly. When a `branch` or `worktree` needs a finish decision, show the actual change branch, target branch, and directory together, then let the user choose merge, push, create a PR, keep the workspace, or defer Archive.

Commit only the implementation and formal artifacts that belong to the current change, preserving other user changes. Execute the returned `commandArgs`, then inspect `workspaceFinishResult`. If it is `blocked`, preserve the workspace and run the recovery command in `recoveryArgs`.

Completion criterion: state is `done`, and the user-authorized workspace finish result is `completed` or `kept`. Follow `continuation` for any other result.

## Follow-up actions

After every command, handle only the latest `continuation`:

- `continue`: execute `commandArgs` and fill `inputOptions` from its template.
- `await-user`: wait for the listed user decision. With `commandAlternatives`, execute the matching complete `commandArgs`, preserve `--expected-state-version` plus `--expected-action`, and reread the latest `continuation` if the alternative is stale. Do not reconstruct an unguarded command.
- `blocked`: resolve the listed blocker or recovery action first.
- `done`: finish.

A state-changing command already returns the latest `state` and `continuation` in its response; rely on them directly, and query the change details again only when a command fails, a guard is stale, or the working directory changed. Run `show` only when formal content is needed.
