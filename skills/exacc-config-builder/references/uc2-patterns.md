# UC2 Patterns

Use this reference when the selected ExaDB-C@C / ExaCC use case is UC2.

UC2 means a hybrid ExaDB-C@C platform: shared ExaDB-C@C infrastructure, with dedicated VMCs/AVMCs per environment. Base the detailed semantics on `workload-extensions/exacc/exacc_use_cases/readme.md` in the selected source ref.

## Native Template Support

In the current temporary source ref, UC2 is documented but pending completion. It does not have native template-backed JSON support.

The `single-stack/readme.md` and `multi-stack/readme.md` deployment tables list UC2 as `coming soon`, and no `uc2` JSON files are present under `workload-extensions/exacc/`.

If the selected source ref still lacks dedicated UC2 templates, stop before final generation. Explain that UC2 is pending completion and ask whether the user wants to switch to UC1 or capture requirements/design notes for future UC2 work.

Do not relabel UC1 files as UC2.

## Resource Pattern

UC2 keeps the ExaDB-C@C infrastructure shared while moving VMCs/AVMCs and database administration boundaries to environment-specific scopes.

Model these placement rules explicitly:

- shared ExaDB-C@C infrastructure compartment for primary and DR infrastructure
- environment-specific ExaDB-C@C database compartments for VMCs, AVMCs, Oracle Homes, CDBs, PDBs, and ACDs
- project-level compartments for ADB-D databases
- software images in the relevant environment-specific ExaDB-C@C database compartments unless the customer confirms a different lifecycle model

Do not assume UC1's shared database compartment is still valid for VMCs/AVMCs in UC2.

## Administration Pattern

UC2 combines global, environment-level, and project-level groups:

- global infra admin team for shared infrastructure
- environment infra admin teams for environment-specific VMC/AVMC operations
- environment DBA teams for environment-specific database administration
- project DBA teams for project-level ADB-D administration

Keep infrastructure and DBA responsibilities separate unless the customer explicitly asks to combine them.

## Observability Pattern

UC2 observability should reflect both shared infrastructure and environment-specific database scopes.

Expected routing pattern:

- shared infrastructure events for the shared ExaDB-C@C infrastructure compartment
- environment-level database events and alarms for each environment-specific ExaDB-C@C database compartment
- project-level event routing for project ADB-D compartments
- notification topics aligned to global and environment/project scopes

When tailoring from UC1, update every event rule compartment, alarm compartment, alarm metric compartment, destination topic, display name, description, and notification topic scope together.

## Deployment Pattern

For an existing Landing Zone, default to multi-stack / extension-only and consume dependency outputs from the deployed foundation.

For a fresh foundation plus UC2 extension, only use single-stack after the foundation design and stack boundary are explicit and source-verified.

Because UC2 has no native templates in the current temporary ref, do not produce deployable UC2 JSON. Any UC2 output must be limited to requirements, design notes, or explicitly non-deployable prototype material until UC2 is completed in the source repo and passes the required validation levels.

## Required UC2 Inputs

Ask only for missing values:

- whether the Landing Zone already exists
- deployed or target blueprint and hub variant
- CIS level
- region and home region
- Orchestrator version and source ref
- shared ExaDB-C@C infrastructure scope
- environments requiring dedicated VMCs/AVMCs
- project compartments requiring ADB-D access
- identity domain name or OCID
- global infra admin group
- environment infra admin groups
- environment DBA groups
- project DBA groups
- notification endpoints
- alarm thresholds and enabled state
- available dependency outputs or required OCID fallbacks

## Dependency Pattern

For multi-stack UC2, prefer dependency outputs over hard-coded OCIDs.

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

- verify that the selected source ref still lacks or now includes completed UC2 templates
- parse every generated JSON file with `jq .`
- verify top-level families against the pinned Orchestrator version
- verify logical references are generated locally or listed as dependencies
- check that shared infrastructure references did not accidentally remain shared database references from UC1
- check that environment-specific VMC/AVMC, database, event, alarm, and notification scopes are coherent
- search for placeholders, stale source paths, and unrelated workload references
- report validation levels separately: design, JSON syntax, Orchestrator contract, Terraform validate, Terraform plan

Do not call UC2 output production-ready unless UC2 has completed source templates and the requested validation level has actually run and passed.
