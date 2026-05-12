# AGENTS.md - OCI Landing Zone Codex Skills Package

These instructions apply to this skills package repository.

## Purpose

This repository packages Codex skills and repo-level agent guidance for OCI Landing Zone Operating Entities work.

The official OCI Landing Zone repositories are the source of truth. Do not replace source inspection with generic OCI knowledge or model memory.

## Repository Layout

- `skills/`: publishable Codex skill folders.
- `repo-agents/`: AGENTS.md files intended to be copied into target repositories.
- `docs/`: design notes and publication support documents.

## Editing Rules

- Keep each skill self-contained.
- Do not add local user paths, private workspace paths, customer OCIDs, tenant-specific emails, or saved Resource Manager outputs.
- Do not add README files inside individual skill folders.
- Keep detailed reference material under each skill's `references/` directory.
- Keep public documentation at the package root or under `docs/`.

## Skill Source Rules

Skills must use official remote repositories as source of truth:

- `oci-landing-zone-operating-entities`
- `terraform-oci-modules-orchestrator`

For real handoff, skills must pin the selected source to a tag, commit SHA, or resolved branch SHA.

## Validation

Before publishing updates, validate each skill with the Codex skill validator when available:

```bash
python3 ~/.codex/skills/.system/skill-creator/scripts/quick_validate.py skills/oci-lz-blueprint-builder
python3 ~/.codex/skills/.system/skill-creator/scripts/quick_validate.py skills/exacc-config-builder
python3 ~/.codex/skills/.system/skill-creator/scripts/quick_validate.py skills/oci-lz-orchestrator-contract-advisor
```

Also scan for local/private residue before sharing:

```bash
rg -n "(/Users/|customer-simulation|\\.simulated|fixture|demo)" --glob '!AGENTS.md' .
```

If a match is intentionally documented, explain it in the handoff.
