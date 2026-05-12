# UC3 Patterns

Use this reference when the selected ExaDB-C@C / ExaCC use case is UC3.

UC3 means a dedicated ExaDB-C@C platform: dedicated ExaDB-C@C infrastructure and dedicated VMCs/AVMCs per environment. Base the detailed semantics on `workload-extensions/exacc/exacc_use_cases/readme.md` in the selected source ref.

## Native Template Support

In the current temporary source ref, UC3 is documented but pending completion. It does not have native template-backed JSON support.

The `single-stack/readme.md` and `multi-stack/readme.md` deployment tables list UC3 as `coming soon`, and no `uc3` JSON files are present under `workload-extensions/exacc/`.

If the selected source ref still lacks dedicated UC3 templates, stop before final generation. Explain that UC3 is pending completion and ask whether the user wants to switch to UC1 or capture requirements/design notes for future UC3 work.

Do not relabel UC1 files as UC3.

## Resource Pattern

UC3 isolates both infrastructure and database compute by environment.

Model these placement rules explicitly:

- environment-specific ExaDB-C@C infrastructure compartments, including primary and DR infrastructure where required
- environment-specific ExaDB-C@C database compartments for VMCs, AVMCs, Oracle Homes, CDBs, PDBs, and ACDs
- project-level compartments for ADB-D databases
- software images in the relevant environment-specific ExaDB-C@C database compartments unless the customer confirms a different lifecycle model

Do not assume UC1's shared infrastructure or shared database compartment is valid for UC3.

## Administration Pattern

UC3 is primarily environment-scoped, with project-level DBA boundaries for Autonomous databases:

- environment infra admin teams for environment-specific ExaDB-C@C infrastructure, VMCs, and AVMCs
- environment DBA teams for environment-specific Oracle Homes, CDBs, PDBs, and ACDs
- project DBA teams for project-level ADB-D administration

Use global groups only if the customer explicitly requires central governance or cross-environment operations.

## Observability Pattern

UC3 observability should keep resource monitoring scoped to each environment while preserving any central notification routing the customer explicitly wants. The current use-case README describes both global and environment-level notification topics, so confirm whether those global topics remain central routing points or should be cloned per environment.

Expected routing pattern:

- environment-level infrastructure events for each ExaDB-C@C infrastructure compartment
- environment-level database events and alarms for each ExaDB-C@C database compartment
- project-level event routing for project ADB-D compartments
- notification topics aligned to the confirmed global, environment, and project scopes

When tailoring from UC1, update every infrastructure, database, event, alarm, metric compartment, destination topic, display name, description, and notification topic reference together.

## Deployment Pattern

For an existing Landing Zone, default to multi-stack / extension-only and consume dependency outputs from the deployed foundation.

For a fresh foundation plus UC3 extension, only use single-stack after the foundation design and stack boundary are explicit and source-verified.

Because UC3 has no native templates in the current temporary ref, do not produce deployable UC3 JSON. Any UC3 output must be limited to requirements, design notes, or explicitly non-deployable prototype material until UC3 is completed in the source repo and passes the required validation levels.

## Required UC3 Inputs

Ask only for missing values:

- whether the Landing Zone already exists
- deployed or target blueprint and hub variant
- CIS level
- region and home region
- Orchestrator version and source ref
- environments requiring dedicated ExaDB-C@C infrastructure
- primary and DR infrastructure placement per environment
- project compartments requiring ADB-D access
- identity domain name or OCID
- environment infra admin groups
- environment DBA groups
- project DBA groups
- notification endpoints
- alarm thresholds and enabled state
- available dependency outputs or required OCID fallbacks

## Dependency Pattern

For multi-stack UC3, prefer dependency outputs over hard-coded OCIDs.

Typical dependencies:

- `compartments_dependency` for existing foundation compartments and environment/project references
- `tags_dependency` for tag namespace/tag references created by the foundation
- `topics_dependency` only when notification topics are external to the current ExaCC operation
- `logging_dependency` only when logs are external to the current ExaCC operation
- `kms_dependency` and `vaults_dependency` only when keys or vaults are external to the current operation

If dependency outputs are unavailable, ask for OCIDs only for prior-stack boundary resources. Preserve generated keys for resources created in the current ExaCC stack.

## Identity Domain Pattern

For an existing Landing Zone, confirm whether groups should be created in an existing identity domain.

Do not assume `identity_domains_output.json` can be passed as a dependency input unless the active Orchestrator contract exposes that input.

For Orchestrator `v2.1.0`, the checked contract does not expose `identity_domains_dependency`; the existing identity domain OCID may need to be supplied directly, or the operation must use a verified same-state pattern that activates the identity-domain module.

## Validation Checklist

Before final handoff:

- verify that the selected source ref still lacks or now includes completed UC3 templates
- parse every generated JSON file with `jq .`
- verify top-level families against the pinned Orchestrator version
- verify logical references are generated locally or listed as dependencies
- check that shared infrastructure or shared database references from UC1 did not remain by accident
- check that each environment's infrastructure, database, event, alarm, and notification scopes are isolated as intended
- search for placeholders, stale source paths, and unrelated workload references
- report validation levels separately: design, JSON syntax, Orchestrator contract, Terraform validate, Terraform plan

Do not call UC3 output production-ready unless UC3 has completed source templates and the requested validation level has actually run and passed.
