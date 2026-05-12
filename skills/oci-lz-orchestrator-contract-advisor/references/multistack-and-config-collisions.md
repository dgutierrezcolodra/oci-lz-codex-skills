# Multistack And Config Collision Reference

Use this reference when the user wants multiple operations, separate states, or multiple JSON/YAML files and the behavior looks inconsistent.

## First Principle

Split by operation and Terraform state, not by randomly scattering the same root configuration family across many files.

Good boundary:

- one operation
- one state file
- one output publication path
- one coherent set of top-level configuration families

Bad boundary:

- several files that all redefine `network_configuration`
- several files that all redefine `policies_configuration`
- expecting the orchestrator to deep-merge repeated root keys across files

## What Actually Happens

### Terraform CLI

When multiple `-var-file` arguments are passed, repeated root variables do not deep-merge. If two files define the same root variable, the later value overrides the earlier one.

Practical consequence:

- `iam.json` plus `network.json` is fine if they populate different root variables
- two files that both define `network_configuration` are not a safe composition strategy

### `rms-facade`

`rms-facade/get_configurations.tf` reads all config objects, builds a flat map by top-level key, and keeps the first occurrence for a repeated key.

Practical consequence:

- repeated top-level keys across config files are not merged
- the outcome is driven by source ordering, not by resource intent
- a user saying "it picks whichever JSON it wants" is usually seeing first-match precedence

## Recommended Pattern

Default to this decomposition unless the local repo shows a different contract:

1. Foundation or shared services
2. One operation per platform or workload family

Typical foundation responsibilities:

- compartments and IAM structure
- shared network
- shared keys, tags, logs, or guardrails

Typical platform responsibilities:

- OKE
- OCVS
- separate workload modules
- platform-specific database or application stacks outside the root orchestrator

## Recommended Answer Pattern

When the user asks "how do I do three platforms", answer in this shape:

1. Explain why multiple JSON files collide in this repo
2. Recommend separate operations and states
3. Name the upstream outputs each platform should consume
4. Call out any schema or repo limits that make the desired split impossible in the current snapshot

## Example Layout

If platforms share common foundation, recommend four operations rather than three:

```text
ops/
  00-foundation/
  10-oke/
  20-exacc/
  30-exacs/
```

Why four instead of three:

- foundation is a separate blast-radius boundary
- platform states stay independent
- shared outputs are published once and consumed many times

If the user insists on only three operations, say clearly that one of these must be true:

- the shared foundation already exists outside these three operations
- one platform stack also owns shared services, which is less clean

## Output And Dependency Flow

Use the active code path to confirm names, then advise in this order:

- `compartments_output.json` feeds `compartments_dependency`
- `network_output.json` feeds `network_dependency`
- `keys_output.json` feeds `kms_dependency`
- `tags_output.json` feeds `tags_dependency`

Do not recommend dependency files that the root module does not actually expose.

## OKE Caveats

Verify these before answering:

- root variables use `oke_clusters_configuration` and `oke_workers_configuration`
- `rms-facade/get_configurations.tf` may look for `clusters_configuration` and `workers_configuration`
- code paths write `oke_output.json`, while README tables may mention a different file name
- output key shapes differ between root `outputs.tf` and `rms-facade/outputs.tf`

For OKE questions, trust the code path the user is executing, not the README table.

## ExaCC And ExaCS Caveat

Do not assume `exacc_configuration` or `exacs_configuration` exist in the orchestrator root module. Confirm them in `variables.tf`.

If they are not present, answer like this:

- foundation can still be handled by the orchestrator
- ExaCC or ExaCS likely need separate Terraform stacks or module repos
- those stacks should consume the same upstream output files rather than trying to fit into nonexistent root families

## Useful Checks

```bash
rg -n "variable \".*_configuration\"|variable \".*_dependency\"" variables.tf
rg -n "all_json_configs_map|all_yaml_configs_map|clusters_configuration|workers_configuration" rms-facade/get_configurations.tf
rg -n "oke_output|compartments_output|network_output|keys_output" outputs.tf rms-facade/outputs.tf README.md
```
