---
name: exacc-config-builder
description: Use when generating, customizing, or reviewing OCI Landing Zone Operating Entities ExaDB-C@C or ExaCC workload-extension JSON files, including single-stack or multi-stack deployment, One-OE or other blueprint fit, IAM, governance, observability, dependency outputs, and Resource Manager configuration URLs.
---

# ExaCC Config Builder

## Overview

Use this skill as a customer-facing guided builder for producing OCI Landing Zone Orchestrator JSON configuration files for the ExaDB-C@C workload extension. The agent must ask the missing deployment questions, derive changes from the current source templates, model the deployment in Jsonnet for production-grade generation, and emit valid JSON without inventing unsupported Orchestrator families.

This is the ExaDB-C@C specialization. It can run standalone when the customer already has a deployed landing zone and only needs the ExaCC workload extension. A separate `oci-lz-blueprint-builder` skill is planned for foundation blueprint design, generation, and review; until it exists in this package, treat foundation design as separate scope and do not pretend this ExaCC skill covers full foundation generation. Use `oci-lz-workload-extension-builder` for non-ExaCC workload extensions when available.

## Source Rule

Use the official Operating Entities remote repository as the default source of truth. Inspect the pinned release, tag, commit SHA, or branch explicitly selected for the customer engagement:

`https://github.com/oci-landing-zones/oci-landing-zone-operating-entities`, path `workload-extensions/exacc/`.

Current temporary source ref: branch `we_exacc_update`. Use it as the default source until the ExaDB-C@C workload extension is merged or published under a stable tag/ref. For any real handoff, resolve and record the branch head SHA so the generated configuration is tied to an immutable source revision.

For ExaCC use-case classification, base the taxonomy only on `workload-extensions/exacc/exacc_use_cases/readme.md`. Customer intake and handoff notes must use `UC1`, `UC2`, `UC3`, or an explicitly described custom combination derived from those README scenarios. Do not introduce alternate classification fields unless the user explicitly asks for a local alias.

Read `references/source-map.md` when you need the source file map, current template families, source quirks, or dependency guidance.

Read `references/uc1-patterns.md` when the selected use case is UC1 and you need stack boundaries, required inputs, dependency behavior, identity-domain handling, observability placement, or validation checks.

Read `references/uc2-patterns.md` when the selected use case is UC2. In the current temporary source ref, treat UC2 as pending completion unless dedicated UC2 templates are present in the selected ref.

Read `references/uc3-patterns.md` when the selected use case is UC3. In the current temporary source ref, treat UC3 as pending completion unless dedicated UC3 templates are present in the selected ref.

Read `references/jsonnet-approach.md` before designing or changing generated blueprints, workload-extension variants, or validation rules.

For production handoff, pin the remote source to a tag or commit SHA. If the customer explicitly requests a branch, resolve and record the branch head SHA and state the drift risk. Do not use hosted GitHub connectors; inspect remote sources through standard Git/HTTP/raw URLs. Use customer-provided local checkouts only for local config files, failing local runs, or when the customer explicitly asks to use that checkout. Always inspect the actual JSON templates before generating output.

For ORM/RMS handoff, use the OCI Landing Zones Orchestrator version referenced by the selected source documentation or explicitly selected by the customer. Inspect and pin that Orchestrator source when checking accepted top-level families, rms-facade merge behavior, dependency inputs, and output file names. If a local Orchestrator checkout is older or different, do not use it as the deployment contract without calling out the version mismatch.

## Guided Flow

1. **Classify the request**
   - Blueprint: One-OE, Multi-OE, Multi-Tenancy, or custom.
   - Deployment approach: `single-stack` for PoC/exploration or `multi-stack` for production/separate lifecycle.
   - Foundation state: fresh landing zone plus ExaCC, or extension-only on an already deployed landing zone.
   - Use case: use the numbered scenarios from `exacc_use_cases/readme.md`: `UC1` shared ExaDB-C@C platform with shared infrastructure and shared VMCs/AVMCs across environments; `UC2` hybrid platform with shared infrastructure and dedicated VMCs/AVMCs per environment; `UC3` dedicated platform with dedicated infrastructure and VMCs/AVMCs per environment; or a custom combination explicitly derived from those scenarios.
   - Template support: confirm whether the selected source ref contains templates for the requested use case and stack mode. If it only contains UC1 templates, do not present UC2 or UC3 generation as native support; handle UC2/UC3 as pending completion, not as supported output.
   - Execution target: Terraform CLI, Resource Manager/rms-facade, or config-only output.
   - If the foundation blueprint must be designed from scratch or changed, treat it as planned foundation-builder scope. Do not generate final ExaCC artifacts until the foundation choices are explicit and source-verified.

2. **Intake gate before generation**
   - Do not generate or present customer-ready ExaDB-C@C/ExaCC workload-extension JSON until the workload requirements are explicit.
   - If required values are missing, stop and ask for them in the user's language before creating final artifacts.
   - It is acceptable to copy upstream templates as a clearly labeled `draft` or `upstream-reference` set, but do not call them configured, customer-ready, final, or deployable.
   - Do not silently keep upstream defaults for customer-facing values such as region, home region, naming prefix, compartment names, environment/project names, identity domain, admin group names, notification emails, CIS level, alarm enabled state, alarm thresholds, logging/service connector choices, dependency outputs, or separate-vs-combined stack boundaries.
   - For ExaCC UC1 specifically, follow the UC1 semantics from `exacc_use_cases/readme.md`; do not assume `COMMON-DOMAIN`, Frankfurt names, example email subscriptions, or sample load balancer/backend values unless the customer explicitly confirms them.
   - If the user asks for One-OE plus any hub variant and ExaCC together, first determine whether the landing zone already exists. For an existing deployed LZ, ask for deployed LZ shape, output dependencies, and OCID fallbacks, then generate only the ExaCC extension artifacts. If the foundation must be generated, changed, or checked against official hub runtime sequencing, treat that as planned foundation-builder scope and make the needed foundation decisions explicit before returning to ExaCC placement, IAM, and observability.
   - Capture the accepted answers in the README or final handoff notes before generation. Do not create separate profile/manifest JSON or YAML artifacts unless the user explicitly asks for them. Treat unanswered customer-facing values as blockers, not as assumptions hidden in the output.

3. **Ask only missing questions**
   Ask in the user's language. Keep the first batch focused:
   - Which blueprint and deployment approach should this target?
   - Is the landing zone foundation already deployed, and do you have dependency output files?
   - For an existing LZ: which deployed blueprint, hub variant, CIS level, region/home region, Orchestrator version, output persistence location, and stack/output prefix should the extension consume?
   - Which ExaCC use case from `exacc_use_cases/readme.md` should this implement: UC1, UC2, UC3, or an explicit custom combination?
   - Which README environments/scopes are in scope for that use case: shared, prod, preprod, DR, and project compartments?
   - How many project compartments need ExaDB-C@C database or ADB-D access per environment?
   - What naming prefix, region code, and compartment names should replace the template defaults?
   - Which identity domain should own groups: existing domain OCID/name, `COMMON-DOMAIN`, or create in the same stack?
   - For an existing identity domain, what is the actual identity domain OCID or how will it be supplied after the LZ output is reviewed?
   - Which admin groups are required for infra, DBA, project, security, and network duties?
   - Which observability settings are required: events only, alarms, CIS1/CIS2 baseline, logging/service connector, notification topics, alarm thresholds/enabled state?

4. **Check coherence before generation**
   - Resolve conflicting customer answers before writing output. For example, "shared/global only" plus "prod project 1 and preprod project 1" means a shared ExaCC platform with project DB compartments unless the customer confirms those project compartments should be removed.
   - State the interpreted scope in one compact summary: blueprint, hub/base, deployment approach, use case, environments, projects, identity domain, notification targets, and dependency source.
   - Distinguish design coherence from generated JSON validity. Do not say the JSON is correct until files have been generated and validation commands have passed.

5. **Choose templates**
   - `single-stack`: start from `single-stack/exacc_governance_uc1.json`, `single-stack/exacc_identity_uc1.json`, the selected `single-stack/exacc_security_cis*_uc1.json`, and the selected `single-stack/exacc_observability_cis*_uc1.json`. Include governance only if the selected blueprint does not already create the required TBAC tag namespace/tag.
   - Fresh One-OE + selected hub + ExaCC UC1: inspect the selected One-OE hub README and runtime JSON first. Use the ExaCC single-stack foundation files plus the selected `blueprints/one-oe/runtime/one-stack/oneoe_network_hub_*.json` file when the selected hub's stack shape supports that combination; do not also include `oneoe_iam.json` because it duplicates identity top-level families.
   - `multi-stack`: start from `multi-stack/exacc_identity_uc1.json` and `multi-stack/exacc_observability_uc1.json`; expect existing landing-zone keys or output dependencies for parent compartments and tags.
   - Existing deployed LZ + extension-only: default to the inspected `multi-stack` ExaCC files for production/separate lifecycle. Generate no foundation blueprint JSON unless the customer explicitly asks for LZ changes. Consume saved dependency outputs when available; otherwise ask for OCIDs only for boundary references owned by the existing LZ, such as parent compartments, tag namespaces/tags, topics/logs, and identity domain.
   - For multi-stack identity with existing identity domains, verify whether the active Orchestrator applies `identity_domain_groups_configuration` without `identity_domains_configuration`. In Orchestrator `v2.1.0`, identity-domain groups are wired through the identity-domains module, so an existing-domain overlay or combined identity-domain configuration may be required.
   - Do not assume `identity_domains_output.json` can be passed back as an identity-domain dependency unless the active Orchestrator root/rms-facade contract exposes such an input. In Orchestrator `v2.1.0`, the checked contract does not expose or load an `identity_domains_dependency` input. For a separate ExaCC stack, replace `identity_domain_groups_configuration.default_identity_domain_id` with the actual identity domain OCID after the LZ deploys it, or build a same-operation/same-state update that includes the identity-domain configuration and merged groups.
   - `UC2` and `UC3`: only generate from dedicated UC2/UC3 source templates if the selected ref contains them. If not, stop before final generation, explain that UC2/UC3 are pending completion in the selected source ref, and ask whether the user wants to switch to UC1 or capture requirements for future UC2/UC3 completion.
   - For other blueprints, inspect the selected blueprint's runtime files first, then map ExaCC compartments, groups, policies, topics, and alarms onto that hierarchy. Do not assume One-OE keys exist in Multi-OE or Multi-Tenancy.

6. **Model in Jsonnet**
   - Use Jsonnet as the source model for generated ExaCC artifacts: customer input, template variants, stack boundaries, dependency requirements, and output files.
   - Use Jsonnet libraries, functions, object composition, overlays, and assertions instead of ad hoc string templating.
   - Use CUE only as an optional validation/contract layer when a CUE schema is available.
   - Use Python only as an optional wrapper for interactive questions, loading source JSON, invoking `jsonnet`, writing files, and summarizing outputs.
   - If `jsonnet` is unavailable, either ask to install it or clearly mark Python-only output as a bootstrap/MVP path, not the preferred production workflow.

7. **Generate JSON**
   - Keep `upstream-reference` or `draft` artifacts physically separate from `customer` artifacts. Only the `customer` set may be described as configured for deployment.
   - Preserve Orchestrator top-level families exactly, such as `compartments_configuration`, `identity_domains_configuration`, `identity_domain_groups_configuration`, `policies_configuration`, `tags_configuration`, `notifications_configuration`, `events_configuration`, `home_region_events_configuration`, `alarms_configuration`, `logging_configuration`, `service_connectors_configuration`, `cloud_guard_configuration`, `scanning_configuration`, `security_zones_configuration`, and `vaults_configuration`.
   - Keep logical keys when the deployment will pass dependency outputs. Replace keys with OCIDs only when the customer explicitly will not use dependency files.
   - When cloning projects/environments, update every key, `parent_id`, topic destination, compartment path, policy statement, display name, and description together.
   - When cloning prod/preprod project patterns, perform exact token/boundary-aware replacements and handle `PREPROD` before `PROD`; otherwise `PREPROD` can be misclassified as prod. Validate by scanning cloned policies for project2 groups pointing at project1 compartment names.
   - Replace all customer-facing placeholders before delivery, including `email.address@example.com`, template naming prefixes, region codes, compartment display names, descriptions, and any default project/environment labels that do not match the accepted scope.
   - Do not create invented handoff artifacts such as `deployment-manifest.json`, profile YAML, wrapper JSON, or execution-map JSON by default. The only default generated JSON/YAML files must be actual Orchestrator configuration inputs derived from inspected source templates. Put procedure, operation order, `input_config_files_urls`, dependency objects, output prefixes, and pre-run edits in a README or the final response. Create a separate manifest/profile only when the user explicitly asks for that artifact.
   - After writing final customer artifacts, inspect sibling or root handoff documentation such as `README.md`, older `config/` folders, and reference notes. If any of them describe a different CIS level, deployment approach, naming prefix, source ref, operation set, or config file list, update them to point to the final customer set, move them under an explicit reference/draft area, or mark them as superseded. Do not leave two plausible deployment instructions for the same request.
   - Do not create `exacc_configuration` or `exacs_configuration` unless the active Orchestrator `variables.tf` proves that family exists.
   - Treat actual ExaDB-C@C infrastructure, VMCs, AVMCs, CDBs, and PDBs as placement and IAM/observability concerns unless a backing module contract is confirmed.

8. **Validate before delivery**
   - Run Jsonnet formatting/generation checks first, then parse every generated file with `jq .`.
   - Run optional CUE validation only when a CUE contract exists for the generated JSON.
   - Search output for unresolved placeholders, stale branch names, wrong workload names, and accidental OKE URLs.
   - Validate the handoff boundary: every README-listed config file path must exist, and root/sibling READMEs must not advertise stale CIS levels, old single-stack versus multi-stack guidance, example emails, obsolete source refs, or older customer scopes as the current deployment.
   - Fail the handoff if placeholders remain in customer artifacts; list the exact files and fields instead of downgrading them to warnings.
   - Verify all logical references either resolve in the same generated set or are listed as required dependencies.
   - If ExaCC multi-stack groups target an identity domain created by a prior LZ stack, verify whether the active Orchestrator accepts an identity-domain dependency input. If it does not, list the exact OCID replacement required before the ExaCC stack can run separately.
   - Verify the selected files against the pinned Orchestrator version for the engagement.
   - Report validation status in levels: design coherent, JSON syntax valid, Orchestrator contract valid, Terraform validate passed, Terraform plan passed. Do not collapse these into a single "100% correct" claim.
   - For RMS/ORM output, provide the `input_config_files_urls` list and required dependency objects/files separately.

## Guardrails

- Do not rely on README DeployToOCI links if they reference OKE or `master`; inspect the JSON file list in the selected branch.
- Do not deep-merge multiple files with the same top-level family unless the active Orchestrator code path has been checked.
- Do not invent exact IAM policy statements. Start from the ExaCC templates and edit paths, groups, compartments, and resource scope deliberately.
- Do not ask for OCIDs when the user has saved Orchestrator dependency outputs and the template can use logical keys.
- Do not deliver partial JSON fragments unless the user explicitly asks for snippets. Generate complete files with filenames.
- Do not use text templating as the source of truth for production output. Use Jsonnet libraries and assertions, then export JSON.
- Do not absorb unrelated workload-extension logic into this skill; route OKE, OCVS, EBS, HPC, AI services, and OpenShift to `oci-lz-workload-extension-builder` unless the user explicitly asks for cross-workload comparison.
- Do not run any Terraform command (`init`, `validate`, `plan`, `apply`, `test`, `graph`, or provider inspection) without explicit approval for that command. Generate configs, dependencies, and command shapes by default.
- Do not claim generated JSON is deployment-ready until Jsonnet generation, `jq`, placeholder scans, Orchestrator contract checks, and the requested Terraform validation level have run.

## Delivery Format

Return:

- Generated file paths or filenames.
- Selected blueprint, deployment approach, and use case.
- Required dependency files or OCID replacements.
- Validation commands run and result.
- Confidence level: design coherence, JSON syntax, Orchestrator contract, Terraform validate, Terraform plan.
- Remaining assumptions or client answers still needed.
