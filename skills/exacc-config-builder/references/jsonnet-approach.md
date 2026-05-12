# Jsonnet Approach For ExaCC JSON Generation

Use Jsonnet as the primary generation layer for OCI Landing Zone Orchestrator JSON. Use Python only as a guided-flow wrapper when interactive prompting, source inspection, or file orchestration is useful. Use optional CUE only as a validation contract when a CUE schema exists.

## Why Jsonnet Here

The team already operates Jsonnet, and this workload needs repeatable generation of many closely related JSON artifacts:

- ExaCC identity and observability variants.
- UC1 ExaDB-C@C generation now, with UC2/UC3 added only after those use cases are completed in the source repo.
- Repeated environment and project compartment patterns.
- Stack-specific outputs for CLI or Resource Manager/rms-facade.

The hard part is not emitting JSON. The hard part is proving that generated blueprints and workload extensions are internally consistent for the Orchestrator:

- Top-level families must match the Orchestrator contract.
- For ORM/RMS work, the Orchestrator contract must be the pinned version referenced by the selected source documentation or explicitly selected by the customer.
- Multi-stack deployments must separate state boundaries and list required dependencies.
- Logical keys must either be created in the generated set or supplied by dependency outputs.
- RMS/rms-facade does not deep-merge duplicate top-level families across multiple config files.
- ExaCC is a workload-extension pattern, not a native `exacc_configuration` family unless active Orchestrator code proves otherwise.

Use Jsonnet libraries and assertions for generation-time consistency. Add CUE validation later only where it provides stronger contract checks over generated JSON.

## Recommended Layout

When adding implementation assets to a repo, use a structure like:

```text
jsonnet/
  lib/
    orchestrator_families.libsonnet
    dependencies.libsonnet
    exacc_model.libsonnet
  templates/
    compartments.libsonnet
    identity.libsonnet
    observability.libsonnet
    governance.libsonnet
  blueprints/
    one_oe.libsonnet
    multi_oe.libsonnet
    multi_tenancy.libsonnet
  workloads/
    exacc.libsonnet
  profiles/
    customer.jsonnet
generated/
  *.json
```

Keep source templates from the selected Operating Entities ref as reference inputs. Treat exported JSON as generated artifacts.

## Model Boundaries

Use Jsonnet objects/functions for:

- `deployment`: blueprint, stack mode, use case (`UC1`, `UC2`, `UC3`, or an explicit custom combination derived from `exacc_use_cases/readme.md`), execution target, environments, projects, naming, identity domain, observability profile.
- `stackBoundary`: generated config files, top-level families included, required dependencies, generated outputs.
- `dependencyRequirement`: `compartments_dependency`, `tags_dependency`, `topics_dependency`, `logging_dependency`, `kms_dependency`, `vaults_dependency`, and any OCID replacement fallback.
- `logicalKeyRef`: key reference plus source: `generated`, `dependency`, or `ocid`.
- `orchestratorFamily`: allowed top-level families only.

Useful invariants:

- `stack_mode` is `single-stack` or `multi-stack`.
- `use_case` is `UC1`, `UC2`, `UC3`, or an explicit custom combination derived from `workload-extensions/exacc/exacc_use_cases/readme.md`.
- `UC2` and `UC3` must not export deployable JSON unless the selected source ref contains completed dedicated templates for those use cases.
- `single-stack` may contain foundation plus ExaCC families in one state.
- `multi-stack` must declare dependencies for parent landing-zone resources referenced by key.
- No duplicate top-level family should be split across files for one RMS operation unless merge behavior has been explicitly confirmed.
- No output may contain `exacc_configuration` or `exacs_configuration`.
- No output may contain OKE DeployToOCI URLs or stale `master/workload-extensions/oke` paths.

## Generation Flow

1. Inspect the selected Operating Entities ref and source JSON templates.
2. Capture customer answers in a Jsonnet profile.
3. Compose the profile with blueprint/workload libraries.
4. Validate dependencies and logical references with Jsonnet assertions.
5. Export complete JSON files with `jsonnet`.
6. Run `jq .` and text checks on every exported file.
7. Run optional CUE validation if a CUE contract exists.
8. Return files plus dependency requirements and RMS/CLI invocation guidance.

## Minimal Commands

Use these command shapes when a Jsonnet implementation exists:

```bash
jsonnetfmt --test jsonnet/**/*.jsonnet jsonnet/**/*.libsonnet
jsonnet -J jsonnet/lib jsonnet/profiles/customer.jsonnet > generated/exacc.json
jq . generated/exacc_identity_uc1.json >/dev/null
```

If the generated set has multiple files, validate all of them and search for forbidden strings:

```bash
find generated -name '*.json' -print0 | xargs -0 -n1 jq . >/dev/null
rg -n "TODO|REPLACE|CHANGE_ME|oracle-quickstart|oke/simple|master/workload-extensions/oke|exacc_configuration|exacs_configuration" generated
```

If a CUE validation contract exists:

```bash
cue vet generated/*.json ./cue/...
```

## When Python Is Still Useful

Use Python for:

- Asking the guided configuration questions and normalizing answers.
- Loading source JSON templates and extracting reusable sections.
- Invoking `jsonnet` commands.
- Writing generated files and reports.
- Producing Resource Manager `input_config_files_urls` and dependency summaries.

Do not let Python string-format JSON for production unless Jsonnet owns the final generation path and validation gates still run.
