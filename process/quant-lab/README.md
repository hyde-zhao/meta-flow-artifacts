# Quant Lab Process Ledger

This directory is the `quant-lab` namespace inside the external process ledger container.

## Purpose

The ledger stores Meta Flow runtime and audit artifacts for the `quant-lab` project, including workflow state, change requests, checkpoints, checks, context capsules, handoffs, release context, review records and story execution evidence.

The workspace-level process container lives at:

```text
/home/hyde/workspace/process
```

This project namespace lives at:

```text
/home/hyde/workspace/process/quant-lab
```

The business project lives at:

```text
/home/hyde/workspace/quant-lab
```

The business project exposes this ledger through a local symlink:

```text
/home/hyde/workspace/quant-lab/process -> ../process/quant-lab
```

## Ownership

This namespace is the owner of `process/**` after CR082. The `quant-lab` Git repository no longer directly tracks the `process/**` file tree. The repository only keeps pointer and bootstrap files such as:

- `LEDGER.md`
- `ledger.yaml`
- `scripts/link-engineering-ledger.sh`
- `.gitignore`

## Bootstrap

From the `quant-lab` project root, rebuild the symlink with:

```bash
scripts/link-engineering-ledger.sh
```

If this ledger namespace is stored at a non-default path:

```bash
PROCESS_LEDGER_ROOT=/path/to/process/quant-lab scripts/link-engineering-ledger.sh
```

## Scope

In scope:

- `STATE.md`
- `changes/`
- `checks/`
- `checkpoints/`
- `context/`
- `discussions/`
- `handoffs/`
- `release/`
- `reviews/`
- `stories/`
- Other process and audit files previously under `quant-lab/process`

Out of scope:

- `data/`
- `reports/`
- credentials or `.env`
- provider / lake / catalog operations
- QMT / MiniQMT runtime
- Git remote publication
- destructive cleanup or old root retirement

## Current CR

Externalization was introduced by CR081 and the project namespace was normalized by CR082:

```text
CR-081 Process Ledger Externalization / Symlink Cutover Gate
CR-082 Process Ledger Project Namespace Normalization
```

The original in-project process directory was backed up during CR081 cutover. See CR081 CP6 / CP7 / CP8 evidence for the exact backup path and verification results. See CR082 CP6 / CP7 / CP8 evidence for namespace verification.
