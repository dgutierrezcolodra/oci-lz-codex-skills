---
name: oci-lz-orchestrator-contract-advisor
description: "Use when OCI Landing Zone Orchestrator contract or runtime behavior must be verified: Resource Manager/rms-facade, Terraform CLI, dependency files, output files, stack sequencing, configuration collisions, supported top-level families, provider/auth failures, repo drift, upgrades, or deployment troubleshooting."
---

# OCI LZ Orchestrator Contract Advisor

## Overview

Use this skill to verify the OCI Landing Zone Orchestrator contract and runtime behavior. It protects blueprint and workload-extension builders from wrong assumptions about accepted top-level families, Resource Manager/rms-facade behavior, dependency files, output files, stack sequencing, and deployment troubleshooting.

This is not a configuration builder. Use `exacc-config-builder` for ExaDB-C@C / ExaCC workload-extension generation and the planned `oci-lz-blueprint-builder` when foundation blueprint generation exists. Use this skill to validate the Orchestrator-facing contract and deployment handoff.

## Scope Boundary

Use this skill for:

- Orchestrator contract checks: variables, supported top-level families, outputs, dependency inputs, and backing module mapping.
- Runtime behavior checks: Terraform CLI versus Resource Manager/rms-facade, source selection, output persistence, and dependency loading.
- Multi-stack and Resource Manager guardrails: stack sequencing, duplicate top-level family collisions, and no-deep-merge behavior.
- Troubleshooting: provider/auth failures, missing dependencies, unresolved logical keys, stale repo aliases, release drift, and upgrade-sensitive behavior.
- Validation confidence: separating design coherence, JSON syntax validity, Orchestrator contract validity, Terraform validation, Terraform plan, and apply-time OCI/runtime risk.

Do not use this skill to:

- Generate foundation blueprint JSON/YAML.
- Generate workload-extension JSON/YAML.
- Choose customer-specific topology by itself.
- Invent IAM policies, module schemas, dependency names, or output files.
- Certify generated JSON as "100% correct" unless the relevant validation evidence has been run and reported.

For generation flows, this skill should usually act as a supporting verifier after the builder skill has selected the blueprint or workload-extension shape.

## Repository Targets

Treat these as the canonical repositories for this skill:

- `https://github.com/oci-landing-zones/terraform-oci-modules-orchestrator`
- `https://github.com/oci-landing-zones/oci-landing-zone-operating-entities`
- `https://github.com/oracle-quickstart/terraform-oci-cis-landing-zone-iam`
- `https://github.com/oci-landing-zones/terraform-oci-modules-networking`
- `https://github.com/oci-landing-zones/terraform-oci-modules-governance`
- `https://github.com/oci-landing-zones/terraform-oci-modules-security`
- `https://github.com/oci-landing-zones/terraform-oci-modules-observability`
- `https://github.com/oci-landing-zones/terraform-oci-modules-workloads`
- `https://github.com/oci-landing-zones/terraform-oci-workloads-ocvs`

Use official remote sources by default. For production handoff, pin each remote source to a tag or commit SHA. If the customer explicitly requests a branch, resolve and record the branch head SHA and state the drift risk. Do not use hosted GitHub connectors; inspect remote sources through standard Git/HTTP/raw URLs. Use customer-provided local checkouts only for local config files, failing local runs, or when the customer explicitly asks to use that checkout.

For all contract advice, use only source-backed names from inspected repositories: top-level families, variable names, dependency inputs, output files, blueprint variants, workload-extension variants, native workload families, operation names, stack modes, and Resource Manager fields. Do not invent aliases or fill gaps from memory; if the inspected source does not contain the requested variant or contract, say so and route any desired departure back to the builder skill as `tailored/custom`.

## Workflow

1. Classify the request: contract verification, deployment handoff, multistack decomposition, Terraform CLI, RMS Facade, config-file collision, dependency/output chaining, workload integration, IAM/policy requirements, provider/auth failures, repo/module mapping, repo-alias mapping, upgrade/release drift, or troubleshooting.
2. Gather only the missing context: remote repo URL/ref or customer-provided local checkout path, execution mode, config source, exact failing step, whether the config uses logical keys or raw OCIDs, and whether the question is about production handoff or local troubleshooting.
3. Choose sources in this order:
   - pinned official remote Git/HTTP source for the orchestrator, OE repo, or backing module that owns the contract being checked
   - explicitly requested branch only after recording its resolved SHA and drift risk
   - customer-provided local checkout when the question is about local configs, local changes, or a failing local run
   - raw URLs or GitHub web endpoints for individual files when a full clone is unnecessary
4. Inspect source files in this order when they are available:
   - `README.md`
   - `SPEC.md`
   - `variables.tf`
   - `outputs.tf`
   - `rms-facade/SPEC.md`
   - `rms-facade/variables.tf`
   - `rms-facade/schema.yml`
   - `rms-facade/get_configurations.tf`
   - `rms-facade/get_dependencies.tf`
   - `rms-facade/outputs.tf`
   - `UPGRADE.md`
   - `RELEASE-NOTES.md`
   - `examples/oci-landing-zone-operating-entities/README.md`
   - OE repo `README.md`, `faq/README.md`, `blueprints/readme.md`, `addons/readme.md`, and `workload-extensions/readme.md` only when the orchestrator question explicitly crosses into blueprint examples, extension steps, or stack-shape decisions documented there
   - backing module README/spec files when the question goes beyond the orchestrator contract
5. For remote inspection, use standard tools only:
   - `git ls-remote --tags --heads <repo-url>` to discover refs
   - `git clone --depth 1 --branch <tag-or-branch> <repo-url>` for a disposable inspection checkout
   - `git show <ref>:<path>` from a fetched checkout when file history matters
   - `curl -L https://raw.githubusercontent.com/<owner>/<repo>/<ref>/<path>` for targeted file reads
   - GitHub web/raw endpoints for issues, PRs, releases, and tags when remote history is needed
6. Load the matching reference file:
   - `references/orchestrator.md` for CLI/RMS usage, dependency contracts, and output files
   - `references/rms-ui-and-sources.md` for Resource Manager UI fields, source-selection logic, and remote config/dependency persistence
   - `references/multistack-and-config-collisions.md` for multiple operations, repeated root keys across JSON/YAML files, and stack boundary design
   - `references/operating-entities.md` for OE sequencing and decomposition when it directly informs orchestrator operation boundaries
   - `references/blueprints-and-extensions.md` only when the orchestrator question explicitly depends on One-OE, Multi-OE, Multi-Tenancy, one-stack versus multi-stack, or OE workload-extension patterns
   - `references/aliases-and-upgrade.md` for repo renames, old path aliases still referenced by docs, and upgrade-sensitive answers
   - `references/module-matrix.md` for mapping configuration families to backing repos and outputs
   - `references/workload-patterns.md` for Compute, OKE, and OCVS integration patterns
   - `references/iam-and-policies.md` for permission and policy questions
   - `references/provider-and-rms-failures.md` for provider/auth/RMS diagnosis
   - `references/troubleshooting.md` for symptom-driven diagnosis
   - `references/question-playbooks.md` for common client questions and response structure
7. Reply with concrete next actions: exact variable names, expected output file names, dependency inputs, stack boundaries when relevant, which repo or branch the answer came from, which backing module to inspect next, repo paths to inspect, and the next validation command.

## Guardrails

- Do not generate blueprint or workload-extension configuration as the primary output. Route generation to the matching builder skill and use this skill to validate the Orchestrator contract and handoff.
- Do not invent blueprint variants, workload-extension variants, operation names, stack modes, Resource Manager fields, top-level families, native workload families, dependency inputs, or output files. Confirm each from inspected source code/docs, or explicitly state that the source does not define it.
- Never invent dependency mappings or output file names. Confirm them from repo docs or Terraform code.
- Never present custom handoff files such as `deployment-manifest.json`, wrapper JSON, profile YAML, or execution-map JSON as Orchestrator or blueprint artifacts. Orchestrator consumes configuration JSON/YAML through its supported variables and `rms-facade` inputs; operation order and handoff notes belong in README/final instructions unless the user explicitly asks for a separate manifest.
- Never use a single correctness claim for all stages. Say exactly which level passed: coherent design, valid JSON, valid Orchestrator contract, `terraform validate`, `terraform plan`, or apply.
- Never imply that multiple configuration files with the same top-level family will be deep-merged. Confirm the active merge or precedence behavior from Terraform CLI or `rms-facade/get_configurations.tf`.
- For same-state stack updates, verify that the next run still passes every configuration family that must remain managed by that state. If an Operating Entities blueprint documents `Step 1` and `Step 2`, treat `Step 2` as an update of the original stack with a complete config set, not as an independent stack containing only the changed files.
- Never conflate deployment-time remote config storage in `rms-facade` with `object_storage_configuration` inside the landing zone itself. These are separate concepts.
- Treat configuration source and dependency source as separate decisions in `rms-facade`.
- If the config uses logical keys instead of OCIDs, check whether a `*_dependency` input is part of the contract before suggesting changes.
- For identity-domain groups, do not infer that `identity_domains_output.json` is accepted as a dependency. Inspect whether the active root module and `rms-facade/get_dependencies.tf` expose and load `identity_domains_dependency`; in Orchestrator `v2.1.0` they do not. Separate stacks that create groups in a previously created identity domain may need the domain OCID in `default_identity_domain_id` or a same-state/combined update.
- When the user asks about IAM requirements, map the enabled configuration families to their backing module repos before answering.
- Distinguish the orchestrator contract from the backing module contract. Many client questions are really about the underlying module, not the root orchestrator.
- Prefer a minimal reproduction path: one stack, one config family, one dependency file, one failing lookup.
- For multistack design, prefer splitting by operation and state boundary, not by arbitrarily scattering the same root configuration family across multiple files.
- When the user says the orchestrator "picks whichever JSON it wants", verify whether the behavior is CLI last-wins override or `rms-facade` first-match precedence before recommending a redesign.
- Do not assume `exacc` or `exacs` are native root configuration families in the current orchestrator snapshot. Confirm their existence in `variables.tf` or treat them as separate Terraform stacks that consume upstream outputs.
- For OKE, verify both accepted input key names and generated output file/key names from the active code path before answering. README tables can drift.
- Default to the orchestrator repo as the primary source of truth. Use OE docs as supporting context, not as the primary contract, unless the question is explicitly about blueprint examples or workload extensions built around the orchestrator.
- If the user uses `OCI Open LZ`, `terraform-oci-open-lz`, or old repo paths, normalize the repo or doc alias before answering. Do not conflate a branding term, a blueprint, and the orchestrator itself.
- If the user asks which stack type to use for an orchestrator-driven workload pattern, use OE workload docs as supporting evidence, not as a replacement for the orchestrator contract.
- If the user asks what an ORM field means, inspect `rms-facade/schema.yml` before relying on screenshots or README prose.
- If the required remote source cannot be inspected and no local checkout was explicitly provided, say so and avoid inventing exact operation names or folder layouts.
- Do not use hosted GitHub connectors. Use standard Git/HTTP/raw URL inspection instead.
- Do not substitute remote default-branch contents for the user's local worktree when the question is about current configs, local changes, or a failing local run.
- Do not invent exact IAM policy statements from memory. Point to the backing module documentation and frame the answer as a checklist unless the exact policy text is confirmed.

## Local Inspection Shortcuts

Use `rg` to narrow the question before answering:

```bash
rg -n "output_path|save_output|configuration_source|url_dependency_source|_dependency" README.md SPEC.md outputs.tf rms-facade
rg -n "schemaVersion|variableGroups|configuration_source|save_output|oci_object_prefix|github_file_prefix" rms-facade/schema.yml
rg -n "compartments_output|network_output|oke_output|ocvs_output" README.md outputs.tf rms-facade/outputs.tf
rg -n "home_region|compartment_key|compartment_ocid|all-services|terraform-oci-open-lz|oci-open-lz" UPGRADE.md RELEASE-NOTES.md README.md
rg -n "all_json_configs_map|clusters_configuration|workers_configuration|oke_clusters_configuration|oke_workers_configuration" variables.tf rms-facade/get_configurations.tf README.md
rg -n "one-stack|multi-stack|One-OE|Multi-OE|Multi-Tenancy|add-ons|workload extensions|Hub A|Hub B|Hub C|Hub E" /path/to/oe/repo/README.md /path/to/oe/repo/faq /path/to/oe/repo/blueprints /path/to/oe/repo/addons /path/to/oe/repo/workload-extensions
rg -n "operating entities|shared services|project" examples/oci-landing-zone-operating-entities/README.md
rg -n "git::https://github.com|module_oci_lz_" SPEC.md
```

## Quick Prompt Examples

- "What dependency file am I missing if my workload uses subnet keys instead of subnet OCIDs?"
- "Should this stack run through Terraform CLI or `rms-facade`?"
- "Why did `network_output.json` not get generated?"
- "How do I split this into multiple operations without JSON files overriding or shadowing each other?"
- "Why does RMS seem to pick the wrong JSON file when I split IAM and platform configs?"
- "Should this be one-stack or multi-stack?"
- "Is `terraform-oci-open-lz` the same thing as `oci-landing-zone-operating-entities`?"
- "Can ExaCC be deployed by this orchestrator directly or only as an OE workload extension?"
- "What does this ORM field actually mean in `rms-facade`?"
- "How should I split shared services, operating entities, and workloads across states and outputs?"
- "Which backing module repo should I inspect for `vaults_configuration`?"
- "What IAM permissions should I review first for this orchestrator stack?"
- "Why is RMS failing to save outputs to GitHub or OCI Object Storage?"
- "Check the official remote repo with standard Git/HTTP tools to see whether this orchestrator behavior changed in a recent PR or release."
