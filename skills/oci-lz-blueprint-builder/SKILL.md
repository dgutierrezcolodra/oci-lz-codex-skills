---
name: oci-lz-blueprint-builder
description: Use when designing, generating, customizing, or reviewing OCI Landing Zone Operating Entities blueprint configuration for One-OE, Multi-OE, Multi-Tenancy, shared services, operating entities, environments, platforms, projects, Jsonnet libraries, Orchestrator JSON/YAML, stack boundaries, and deployment dependency outputs.
---

# OCI LZ Blueprint Builder

## Overview

Use this skill to build enterprise-grade OCI Landing Zone blueprint configurations. The agent must use the Operating Entities blueprint sources and the Orchestrator contract, model customer choices in Jsonnet, and export validated JSON/YAML artifacts for Terraform CLI or Resource Manager.

This skill is for foundation blueprints. Use `oci-lz-workload-extension-builder` for generic workload extensions and `exacc-config-builder` for ExaDB-C@C-specific extension design.

## Source Rule

Use official remote repositories by default:

- `https://github.com/oci-landing-zones/oci-landing-zone-operating-entities` for blueprint design/runtime sources
- `https://github.com/oci-landing-zones/terraform-oci-modules-orchestrator` for accepted families, dependency inputs, output files, and RMS behavior

Read `references/blueprint-source-map.md` for current blueprint paths and `references/jsonnet-blueprint-contract.md` before designing generated output.

For blueprint and hub-variant classification, base the taxonomy only on inspected Operating Entities blueprint docs and runtime files. Customer intake and handoff notes must use official blueprint names and source-backed runtime variants, for example `One-OE` plus a confirmed `blueprints/one-oe/runtime/one-stack/oneoe_network_hub_*.json` file such as Hub A, Hub B, Hub C, or Hub E. Do not introduce alternate blueprint, hub, environment, platform, or project model labels unless the user explicitly asks for a local alias.

For production handoff, pin every remote source to a tag or commit SHA. If the customer explicitly requests a branch, resolve and record the branch head SHA and state the drift risk. Do not use hosted GitHub connectors; inspect remote sources through standard Git/HTTP/raw URLs. Use customer-provided local checkouts only for local config files, failing local runs, or when the customer explicitly asks to use that checkout. Never invent blueprint operation names, dependency files, or top-level Orchestrator families.

## Guided Flow

1. **Classify scope**
   - Blueprint: `One-OE`, `Multi-OE generic`, `Multi-OE service provider`, `Multi-Tenancy`, or tailored.
   - Organization model: single OE, multiple OEs in one tenancy, provider/pod model, or several tenancies.
   - Network hub variant: choose only from inspected runtime files such as `oneoe_network_hub_*.json`; map Hub A/B/C/E to their exact runtime files only after each file is confirmed in the selected ref.
   - Deployment mode: one-stack, multi-stack, Resource Manager/rms-facade, Terraform CLI, or config-only.
   - Required families: IAM, governance/tags/budgets, network, security, observability, tooling, workload-ready shared services.

2. **Intake gate before generation**
   - Do not generate or present customer-ready blueprint JSON until the customer requirements are explicit.
   - If required values are missing, stop and ask for them in the user's language before creating final artifacts.
   - It is acceptable to copy upstream templates as a clearly labeled `draft` or `upstream-reference` set, but do not call them configured, customer-ready, final, or deployable.
   - Do not silently keep upstream defaults for customer-facing values such as region, home region, naming prefix, compartment names, environment/project names, notification emails, CIDRs, DRG peers, load balancer backends, identity domain names, CIS level, Cloud Guard, Security Zones, scanning, logging, alarms, service connectors, output persistence, or stack lifecycle boundaries.
   - Do not infer a non-existent hub or foundation variant from prose. If the requested variant is not present in the inspected blueprint runtime files, stop and report the mismatch.
   - If the user requests a specific blueprint shape but omits these values, ask for the smallest complete set needed for that shape. Offer conservative defaults only as explicit choices requiring confirmation.
   - Capture the accepted answers in the README or final handoff notes before generation. Do not create separate profile/manifest JSON or YAML artifacts unless the user explicitly asks for them. Treat unanswered customer-facing values as blockers, not as assumptions hidden in the output.

3. **Ask only missing questions**
   - Tenancy and operating model: one tenancy or multiple tenancies?
   - OE/environment/project hierarchy and naming convention.
   - Required hub variant from the inspected runtime files, such as Hub A/B/C/E, plus that hub's firewall, load balancer, private DNS, remote peering, DRG routing, backend, and post-update requirements from its own README.
   - Security baseline: CIS level, Cloud Guard, vulnerability scanning, security zones, vault/KMS.
   - Observability baseline: notifications, events, logs, service connectors, alarms.
   - State boundaries and ownership: central platform team, OE team, project team, workload team.
   - Output persistence: `output_path` for CLI or `save_output` with OCI Object Storage/GitHub for rms-facade.

4. **Choose source templates**
   - `One-OE`: use `blueprints/one-oe/runtime/one-stack`.
   - `Multi-OE generic`: use `blueprints/multi-oe/generic_v1/runtime` unless a confirmed newer model is selected.
   - `Multi-OE service provider`: use `blueprints/multi-oe/service-providers/runtime`.
   - `Multi-Tenancy`: inspect the blueprint docs and dependent One-OE/Multi-OE sources before generating.
   - Preserve official runtime sequencing for the selected hub. Inspect the matching One-OE hub README, not only Hub E. If it defines `Step 1`/`Step 2` with `*_pre` files, final files, firewall private IPs, NLB private IPs, backends, security-zone targets, or flow-log updates, model those as an initial stack create followed by an update of the same LZ stack. Do not collapse the sequence, rename it, or turn `Step 2` into a new independent LZ stack.

5. **Model in Jsonnet**
   - Jsonnet is the source of truth for generated blueprint artifacts: customer requirements, blueprint selection, stack operations, top-level families, dependency requirements, and output files.
   - Use Jsonnet libraries, functions, object composition, and overlays instead of ad hoc string templating.
   - Use Jsonnet assertions for local invariants and CUE only as an optional validation/contract layer when a CUE schema is available.
   - Python may be used as a wrapper for prompting, reading source JSON/YAML, invoking `jsonnet`, writing output, and summarizing deployment handoff. It must not own the production generation logic.
   - If `jsonnet` is unavailable, ask to install it or label Python-only output as bootstrap output.

6. **Generate artifacts**
   - Produce complete configuration files per stack boundary.
   - For same-stack updates such as One-OE `Step 2`, generate a complete replacement config set for that stack, including unchanged IAM/governance files and the selected hub's required network/security/observability files. Some hubs change only security/observability; others also require final network files or backend/IP substitutions. Do not output only the changed files unless the active Orchestrator contract proves partial updates are safe for that execution path.
   - Keep `upstream-reference` or `draft` artifacts physically separate from `customer` artifacts. Only the `customer` set may be described as configured for deployment.
   - Preserve exact Orchestrator top-level families.
   - Do not split the same top-level family across files in one RMS operation unless the active Orchestrator merge behavior has been reviewed.
   - Prefer logical keys with dependency outputs across stacks; use OCIDs only when explicitly requested or when dependency outputs are unavailable.
   - Replace all customer-facing placeholders before delivery, including notification emails, naming prefixes, region codes, compartment display names, descriptions, and default environment/project labels that do not match the accepted scope.
   - Do not blindly global-rewrite a `naming_prefix` across blueprint resource names. If the inspected blueprint does not expose a global naming-prefix variable, update only coherent customer-facing names/paths and document any retained source naming convention.
   - When expanding an upstream template beyond its built-in project/environment count, clone every related compartment, group, policy, network security group, security-zone target, and observability reference together. Use exact token/boundary-aware replacements; handle `PREPROD` before `PROD` so `PREPROD` is not accidentally rewritten as `PREdoc` or treated as prod.
   - When the customer removes an optional blueprint component such as an example public load balancer, NLB backend, firewall path, subnet, or route target, remove the component plus dependent subnets, route tables, security lists, NSGs, alarms, backend/backend-set/listener references, and then scan for the removed logical keys.
   - Do not create invented handoff artifacts such as `deployment-manifest.json`, profile YAML, wrapper JSON, or execution-map JSON by default. The only default generated JSON/YAML files must be actual Orchestrator configuration inputs derived from inspected source templates. Put procedure, operation order, `input_config_files_urls`, dependency objects, output prefixes, and pre-run edits in a README or the final response. Create a separate manifest/profile only when the user explicitly asks for that artifact.
   - After writing final customer artifacts, inspect sibling or root handoff documentation such as `README.md`, older `config/` folders, and reference notes. If any of them describe a different CIS level, deployment approach, naming prefix, source ref, operation set, or config file list, update them to point to the final customer set, move them under an explicit reference/draft area, or mark them as superseded. Do not leave two plausible deployment instructions for the same request.

7. **Validate**
   - Run Jsonnet formatting/generation checks.
   - Parse every JSON file with `jq .`.
   - Run optional CUE validation only when a CUE contract exists for the generated JSON.
   - Search for placeholders, stale repo aliases, wrong branch paths, unsupported families, and unresolved logical keys.
   - Validate the handoff boundary: every README-listed config file path must exist, and root/sibling READMEs must not advertise stale CIS levels, old one-stack versus multi-stack guidance, example emails, obsolete source refs, or older customer scopes as the current deployment.
   - Fail the handoff if placeholders remain in customer artifacts; list the exact files and fields instead of downgrading them to warnings.
   - Provide exact dependency files to pass to the next stack.
   - Report validation status in levels: design coherent, JSON syntax valid, Orchestrator contract valid, Terraform validate passed, Terraform plan passed. Do not collapse these into a single "100% correct" claim.

## Enterprise Guardrails

- Separate design decisions from deployment mechanics: blueprint model first, stack boundary second, RMS/CLI parameters third.
- Preserve blast-radius boundaries: shared services, OE foundations, environment/platform layers, and project layers should not be collapsed without explicit approval.
- Do not make RMS source storage and landing-zone object storage mean the same thing.
- Do not assume multi-stack operations are deep-merged or order-independent.
- Do not use workload-extension templates to define foundation blueprint families unless the extension docs explicitly require foundation changes.
- Do not run any Terraform command (`init`, `validate`, `plan`, `apply`, `test`, `graph`, or provider inspection) without explicit approval for that command. Generate configs, dependencies, and command shapes by default.
- Do not claim generated JSON is deployment-ready until Jsonnet generation, `jq`, placeholder scans, Orchestrator contract checks, and the requested Terraform validation level have run.

## Delivery Format

Return:

- Selected blueprint and stack boundary rationale.
- Generated filenames and top-level families per file.
- Required input dependencies and expected output files.
- Resource Manager variables or Terraform CLI command shape.
- Validation commands and result.
- Confidence level: design coherence, JSON syntax, Orchestrator contract, Terraform validate, Terraform plan.
- Remaining assumptions requiring customer approval.
