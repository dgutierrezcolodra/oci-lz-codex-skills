# AGENTS.md - ExaDB-C@C Workload Extension

These instructions apply to every file under `workload-extensions/exacc/`.

This directory contains the OCI Landing Zone Operating Entities workload extension for ExaDB-C@C / ExaCC. Treat it as a workload extension that plugs into an OCI Landing Zone, not as a replacement for the foundation blueprint.

## Public Use Contract

This file is intended for public repository use and for agents assisting users with real OCI environments. Keep it self-contained: do not depend on private workspaces, unpublished skills, local fixture paths, or demo-only files.

Skills can provide reusable expert workflows, but this file defines the repository-local contract: source files to inspect, supported stack boundaries, dependency behavior, validation expectations, and safety rules for this ExaDB-C@C workload extension.

When helping a user with a real environment:

- Ask for missing environment-specific values before generating final configuration.
- Do not treat repository examples as customer-approved values.
- Do not commit tenancy-specific generated artifacts, saved Resource Manager outputs, OCIDs, email addresses, or other environment data to the public repository unless they are sanitized examples.
- Keep real customer outputs and generated deployment artifacts outside the public example tree, or clearly mark them as local/uncommitted handoff artifacts.
- Do not claim that a configuration is production-ready unless the required validation level has actually been run and reported.

## Read First

Before changing or generating configuration in this directory, inspect the relevant source files:

- `readme.md` for the extension overview and supported deployment approaches.
- `exacc_use_cases/readme.md` for the supported use-case taxonomy.
- `single-stack/readme.md` and the JSON files under `single-stack/` when the deployment is a combined foundation plus workload-extension stack.
- `multi-stack/readme.md` and the JSON files under `multi-stack/` when the deployment extends an already deployed One-OE landing zone.
- The OCI Landing Zone Orchestrator contract, pinned to the version referenced by the current deployment documentation or explicitly selected by the user.

Do not rely only on README prose or Deploy to OCI links. Check the actual JSON files and the Orchestrator variables/module wiring before changing the accepted top-level families, dependency behavior, or stack boundaries.

## Supported Scope

Use only the use cases documented in `exacc_use_cases/readme.md` unless the user explicitly asks for a custom variant:

- `UC1`: shared ExaDB-C@C platform with shared infrastructure and shared VMCs/AVMCs across environments.
- `UC2`: hybrid ExaDB-C@C platform with shared infrastructure and dedicated VMCs/AVMCs per environment.
- `UC3`: dedicated ExaDB-C@C platform with dedicated infrastructure and VMCs/AVMCs per environment.

When documenting or generating files, use `UC1`, `UC2`, `UC3`, or an explicitly described custom combination derived from those scenarios. Do not invent alternate use-case labels.

## Deployment Models

The extension currently has two deployment models:

- `single-stack/`: combined One-OE landing zone plus ExaDB-C@C workload extension. Use this for PoC, exploration, or cases where the foundation and extension intentionally share one state and one deployment lifecycle.
- `multi-stack/`: extension-only deployment on top of an already deployed One-OE landing zone. Use this for production-like separation of lifecycle and state.

For an already deployed landing zone, default to `multi-stack/`. Do not regenerate or modify the One-OE foundation unless the user explicitly asks for a foundation change.

For a fresh foundation plus extension deployment, use `single-stack/` and include the full required file set for the selected CIS level. Do not mix single-stack and multi-stack files in the same Resource Manager operation unless the Orchestrator merge behavior and top-level family collisions have been checked.

## File Boundaries

Keep generated or edited artifacts aligned to the existing stack boundaries:

- Multi-stack UC1 uses:
  - `multi-stack/exacc_identity_uc1.json`
  - `multi-stack/exacc_observability_uc1.json`
- Single-stack UC1 uses:
  - `single-stack/exacc_governance_uc1.json`
  - `single-stack/exacc_identity_uc1.json`
  - one of `single-stack/exacc_observability_cis1_uc1.json` or `single-stack/exacc_observability_cis2_uc1.json`
  - one of `single-stack/exacc_security_cis1_uc1.json` or `single-stack/exacc_security_cis2_uc1.json`

If UC2, UC3, or a custom variant is requested, first confirm that matching source templates exist. In the current temporary source state, UC2 and UC3 are pending completion; do not generate deployable UC2/UC3 files or relabel UC1 files as UC2/UC3. If the user wants to continue, capture requirements or design notes as unfinished work.

Do not add invented handoff artifacts such as deployment manifests, profile YAML files, wrapper JSON files, or execution maps unless the user explicitly requests them. Procedure notes, dependency lists, Resource Manager variables, and validation results belong in README-style documentation or in the final handoff response.

## Orchestrator Contract

Preserve exact Orchestrator top-level family names. Examples used by this extension include:

- `compartments_configuration`
- `identity_domains_configuration`
- `identity_domain_groups_configuration`
- `policies_configuration`
- `tags_configuration`
- `notifications_configuration`
- `events_configuration`
- `home_region_events_configuration`
- `alarms_configuration`
- `logging_configuration`
- `service_connectors_configuration`
- `cloud_guard_configuration`
- `scanning_configuration`
- `security_zones_configuration`
- `vaults_configuration`

Do not add families such as `exacc_configuration`, `exacs_configuration`, or native ExaDB-C@C infrastructure families unless the active Orchestrator version and backing modules prove that those families are supported.

When using Orchestrator `v2.1.0`, be careful with identity-domain groups in multi-stack mode:

- `identity_domain_groups_configuration` is wired through the identity-domains module.
- The module activation depends on `identity_domains_configuration`.
- `identity_domains_output.json` is not a normal Resource Manager dependency input exposed by the checked `v2.1.0` root contract.
- If groups must be created in an existing identity domain, use the actual identity domain OCID when required, or use a verified same-operation contract pattern that activates the identity-domains module without creating a new domain.

Re-check this behavior before changing it for a different Orchestrator version.

## Dependencies And Outputs

For multi-stack extension-only deployments, consume dependency outputs from the already deployed landing zone whenever possible. Prefer logical keys that resolve through dependency files over hard-coded OCIDs, except where the Orchestrator contract requires an OCID.

Typical foundation outputs for ExaDB-C@C multi-stack deployments include:

- `compartments_output.json`: input dependency for parent compartments and existing landing-zone compartment keys.
- `tags_output.json`: input dependency for tag namespace and tag keys when policies or resources reference landing-zone tags.
- `identity_domains_output.json`: lookup/context for existing identity domain identifiers unless the active Orchestrator version explicitly supports identity domains as dependency inputs.

When example outputs, generated outputs, or saved Resource Manager outputs are present, keep their roles explicit:

- Foundation outputs from a previous landing-zone stack are dependency inputs for an extension stack.
- Extension configuration JSON files are Orchestrator inputs.
- Extension outputs produced after a successful run are outputs for downstream consumers, not inputs to the same operation.
- Reference or sample files must not be treated as customer-specific deployment inputs unless their source and freshness are confirmed.

## Generation Rules

When generating customer-specific JSON:

- Start from the existing JSON templates in this directory.
- Produce complete Orchestrator configuration files, not partial snippets.
- Keep upstream examples, drafts, generated samples, and customer-ready files physically separated.
- Replace customer-facing placeholders before handoff, including emails, identity domain placeholders, naming prefixes, region codes, compartment names, descriptions, and environment/project labels.
- Keep logical references coherent across compartments, policies, groups, events, alarms, topics, and tags.
- When cloning environment or project patterns, update keys, `parent_id` values, policy statements, topic destinations, event destinations, display names, and descriptions together.
- Process longer tokens before shorter tokens when replacing names, for example `PREPROD` before `PROD`.
- Do not silently keep upstream defaults such as `COMMON-DOMAIN`, example email addresses, Frankfurt-specific naming, or sample project names unless they are explicitly accepted for the target environment or documented example.

Do not use ad hoc text templating as the source of truth for production output. Prefer Jsonnet or a structured generator that models stack boundaries, dependency requirements, and invariants.

## Resource Manager Handoff

For Resource Manager, provide the `input_config_files_urls` list separately from dependency files or OCID replacements.

For multi-stack UC1, the config file list is normally:

```text
workload-extensions/exacc/multi-stack/exacc_identity_uc1.json
workload-extensions/exacc/multi-stack/exacc_observability_uc1.json
```

For single-stack UC1, the config file list depends on the selected CIS level and must include governance, identity, the matching observability file, and the matching security file.

Do not claim a Resource Manager deployment is ready unless required dependency files or OCID substitutions are explicitly identified.

## Validation

Before saying that generated or edited JSON is valid, run the strongest practical checks for the requested scope:

- Parse every changed JSON file with `jq .`.
- Search for unresolved placeholders such as `COMMON-DOMAIN`, example email addresses, stale branch names, stale OKE references, and source-only naming left in customer files.
- Check that every logical reference either resolves in the same generated set or is listed as a required dependency.
- Check top-level family names against the active Orchestrator version.
- For multi-stack, verify that required dependency output files and output names match the active Orchestrator contract.

Do not run `terraform init`, `terraform validate`, `terraform plan`, `terraform apply`, `terraform test`, or `terraform graph` unless the user explicitly approves that command.

Report validation in levels:

- design coherence checked
- JSON syntax checked
- Orchestrator contract checked
- Terraform validate run or not run
- Terraform plan run or not run

Do not collapse these into a generic "correct" or "production ready" statement.

## Documentation Updates

When changing deployment behavior, stack boundaries, file lists, CIS level, source branch/tag, dependency requirements, or customer scope, update the relevant README in the same change.

Avoid leaving two plausible but conflicting deployment paths for the same use case. If older notes are retained for reference, clearly mark them as reference, draft, or superseded.

## Git And Review Discipline

Keep changes scoped to ExaDB-C@C workload-extension behavior. Do not reformat unrelated files or rewrite generated images/content unless the task requires it.

Before handoff, summarize:

- selected deployment model and use case
- files changed or generated
- dependency files or OCID substitutions required
- validation commands run and their results
- remaining assumptions or user decisions
