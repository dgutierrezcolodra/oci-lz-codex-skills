# OCI Landing Zone Codex Skills

This repository packages Codex skills and repo-level agent guidance for OCI Landing Zone Operating Entities work.

## Contents

```text
skills/
  exacc-config-builder/
  oci-lz-orchestrator-contract-advisor/

repo-agents/
  workload-extensions/
    exacc/
      AGENTS.md

docs/
  exacc_config_builder_skill_design_spec.md
```

## Skills

### `exacc-config-builder`

Use this skill to design, generate, customize, or review ExaDB-C@C / ExaCC workload-extension Orchestrator JSON.

Current publication scope:

- UC1 has native template-backed support in the current temporary source ref.
- UC2 and UC3 are documented use-case models, but they are pending completion.
- If dedicated UC2/UC3 templates are not present in the selected source ref, the skill must stop before generation and treat them as pending work, not as supported tailored generation flows.
- UC1, UC2, and UC3 have separate reference files under `skills/exacc-config-builder/references/` so the agent does not flatten the three models into one generic pattern.
- Existing deployed Landing Zone scenarios default to multi-stack / extension-only.
- Fresh Landing Zone plus ExaCC scenarios use single-stack when the foundation and workload extension should be deployed together.

Current temporary ExaCC source ref:

- `we_exacc_update`
- For real handoff, resolve and record the branch SHA until this moves to a stable tag or ref.

### `oci-lz-orchestrator-contract-advisor`

Use this skill to verify OCI Landing Zone Orchestrator contract behavior and troubleshoot runtime issues.

It checks areas such as:

- supported top-level families
- RMS/rms-facade behavior
- dependency input names
- output file names
- stack sequencing
- multi-stack boundaries
- config collisions
- provider and RMS failures
- version drift across local checkouts, branches, tags, and Resource Manager links

## Planned Skills

`oci-lz-blueprint-builder` is a planned skill for Landing Zone foundation blueprint design and review. It is mentioned because ExaCC work may depend on foundation context, but it still needs to be created and is not included in this package.

## Source Of Truth

The official OCI Landing Zone repositories are the source of truth:

- `oci-landing-zone-operating-entities`
- `terraform-oci-modules-orchestrator`

The skills should not rely on generic cloud knowledge or old model memory as the main source. They must inspect the relevant repositories, source templates, docs, Orchestrator contracts, dependency outputs, and naming patterns.

Local repositories are not required by default. The skills should use official remote repositories or temporary inspection checkouts when needed. Local checkouts are only needed when a user provides one, when reviewing local changes, or when troubleshooting a local run.

## Installing Locally

Copy the desired skill folders into your Codex skills directory:

```bash
mkdir -p ~/.codex/skills
cp -R skills/exacc-config-builder ~/.codex/skills/
cp -R skills/oci-lz-orchestrator-contract-advisor ~/.codex/skills/
```

Restart Codex or reload skills after copying.

## Repo-Level AGENTS.md

The ExaCC repo-level agent guidance is stored here:

```text
repo-agents/workload-extensions/exacc/AGENTS.md
```

When publishing it into the Operating Entities repository, place it at:

```text
workload-extensions/exacc/AGENTS.md
```

This file is not a replacement for the skills. It gives repo-local rules to any agent working inside that part of the repository.

## Maturity

This package is v0.1.

It is ready for team review and controlled testing. Do not treat generated output as production-ready unless the requested validation level has actually been run and reported.
