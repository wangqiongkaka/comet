# Install Comet from Local Code

This document explains how to run or install Comet from the repository so you can verify unreleased
changes. Ordinary users should install the published package with `npm install -g @rpamis/comet` and
do not need any step described here.

## Build first

All three approaches depend on build output: `bin/comet.js` loads `dist/`, and the Skill launchers
load the generated `.mjs` files under `assets/skills*/`. Build once after a fresh clone or after
switching branches.

```bash
cd <repository-root>
pnpm install
pnpm build
```

When only one runtime changed, the matching incremental build is faster:

```bash
pnpm build:native-runtime    # changes under domains/comet-native/
pnpm build:classic-runtime   # changes under domains/comet-classic/
pnpm build:entry-runtime     # changes under domains/comet-entry/
```

Confirm the built version:

```bash
node bin/comet.js --version
```

## Approach 1: run the local CLI directly

Runs the repository entry point without installing anything or touching global state.

```bash
node <repository-root>/bin/comet.js --version
node <repository-root>/bin/comet.js native status --json
```

- Use when verifying CLI behavior itself, such as command output, exit codes, or JSON structure.
- Limitation: Skill boilerplate resolves `comet` through PATH, so this approach cannot verify the
  full Skill execution path.

## Approach 2: pack a tarball and install it globally

Closest to a real user installation. It also verifies the `files` allowlist in `package.json` and the
postinstall script.

```bash
cd <repository-root>
pnpm build
npm pack --pack-destination /tmp
npm install -g /tmp/rpamis-comet-<version>.tgz
comet --version
```

- Use for final confirmation before a release, or to diagnose "works locally, breaks after install"
  packaging gaps.
- Note: this replaces the current global Comet. Restore the published version with:

```bash
npm install -g @rpamis/comet
```

To verify packaging and installation without touching the global environment, run the repository's
end-to-end script instead. It performs pack, install, and smoke verification in a temporary
directory and leaves the global installation unchanged:

```bash
pnpm test:package-e2e
```

## Approach 3: npm link

Symlinks the global `comet` to the current repository. Best for development that requires repeated
changes and immediate verification.

```bash
cd <repository-root>
pnpm build
npm link
comet --version
```

Once linked, a rebuild is enough for each change; no reinstall is needed:

```bash
pnpm build:native-runtime && comet native status --json
```

Restore the published version when finished:

```bash
npm unlink -g @rpamis/comet
npm install -g @rpamis/comet
```

- Use during development for repeated verification.
- Note: until you unlink, the global `comet` always points at the working tree, so switching branches
  directly changes the behavior of the global command.

## Choosing an approach

| Goal                                           | Recommended             |
| ---------------------------------------------- | ----------------------- |
| Verify CLI output or exit codes only           | Approach 1              |
| Repeated changes during development            | Approach 3              |
| Verify the full installed result and assets    | Approach 2              |
| Verify packaging without changing global state | `pnpm test:package-e2e` |
