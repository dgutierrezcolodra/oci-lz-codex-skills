# UC1 Patterns

Use this reference when the selected ExaDB-C@C / ExaCC use case is UC1.

UC1 means a shared ExaDB-C@C platform with shared infrastructure and shared VMCs/AVMCs across environments. Base the detailed semantics on `workload-extensions/exacc/exacc_use_cases/readme.md` in the selected source ref.

## Native Template Support

In the current temporary source ref, UC1 has native template-backed support.

Single-stack UC1 templates:

- `workload-extensions/exacc/single-stack/exacc_governance_uc1.json`
- `workload-extensions/exacc/single-stack/exacc_identity_uc1.json`
- `workload-extensions/exacc/single-stack/exacc_observability_cis1_uc1.json`
- `workload-extensions/exacc/single-stack/exacc_observability_cis2_uc1.json`
- `workload-extensions/exacc/single-stack/exacc_security_cis1_uc1.json`
- `workload-extensions/exacc/single-stack/exacc_security_cis2_uc1.json`

Multi-stack UC1 templates:

- `workload-extensions/exacc/multi-stack/exacc_identity_uc1.json`
- `workload-extensions/exacc/multi-stack/exacc_observability_uc1.json`

Always inspect the selected source ref before assuming this list is still current.

## Deployment Patterns

### Fresh Landing Zone + UC1 ExaCC

Use the single-stack pattern when the foundation and ExaCC workload extension should be deployed together.

Expected file set:

- governance
- identity
- selected observability file for the chosen CIS level
- selected security file for the chosen CIS level

Do not add a separate foundation identity file if the selected ExaCC identity template already includes the required foundation identity families. Check top-level family collisions before combining files.

For the currently inspected source ref, the CIS2 security file includes `vaults_configuration`; preserve it when CIS2 is selected and verify it against the pinned Orchestrator version.

### Existing Landing Zone + UC1 ExaCC

Use the multi-stack / extension-only pattern when the Landing Zone already exists.

Expected file set:

- `multi-stack/exacc_identity_uc1.json`
- `multi-stack/exacc_observability_uc1.json`

Do not regenerate the foundation by default. Consume dependency outputs from the existing Landing Zone when available.

## Required UC1 Inputs

Ask only for missing values:

- deployed blueprint or target foundation blueprint
- fresh vs existing Landing Zone
- hub variant when relevant
- CIS level
- region and home region
- Orchestrator version
- naming prefix and region code
- environments in scope, typically shared, prod, and preprod
- project compartments in scope
- identity domain name or OCID
- global ExaCC infra admin group
- global ExaCC DBA group
- project DBA/admin groups
- notification endpoints
- observability choices, including events, alarms, thresholds, and enabled state

Do not silently keep source defaults such as placeholder emails, `COMMON-DOMAIN`, sample region naming, or sample project names unless they are explicitly accepted for an example.

## Dependency Pattern

For multi-stack UC1, prefer dependency outputs over hard-coded OCIDs.

Typical dependencies:

- `compartments_dependency` for existing parent compartments and environment/project compartment references
- `tags_dependency` for tag namespace/tag references created by the foundation
- `topics_dependency` only when notification topics are external to the current ExaCC operation
- `logging_dependency` only when logs are external to the current ExaCC operation
- `kms_dependency` and `vaults_dependency` only when CIS2 keys or vaults are external to the current operation

If dependency outputs are unavailable, ask for OCIDs only for prior-stack boundary resources. Preserve generated keys for resources created in the current ExaCC stack.

## Identity Domain Pattern

For an existing Landing Zone, confirm whether groups should be created in an existing identity domain.

Do not assume `identity_domains_output.json` can be passed as a dependency input unless the active Orchestrator contract exposes that input.

For Orchestrator `v2.1.0`, the checked contract does not expose `identity_domains_dependency`; the existing identity domain OCID may need to be supplied directly, or the operation must use a verified same-state pattern that activates the identity-domain module.

## Observability Pattern

UC1 observability should keep shared ExaCC platform monitoring and project-level notification routing coherent.

Check that:

- event rules target the correct compartments
- alarms are placed in the expected ExaCC database scope
- alarm destinations point to existing or generated notification topics
- notification endpoints are customer-approved
- topic references resolve in the same file set or through listed dependencies

## Validation Checklist

Before final handoff:

- parse every generated JSON file with `jq .`
- verify top-level families against the pinned Orchestrator version
- verify logical references are generated locally or listed as dependencies
- check that no UC2/UC3 labels were applied to UC1 templates
- search for placeholders, stale source paths, and unrelated workload references
- report validation levels separately: design, JSON syntax, Orchestrator contract, Terraform validate, Terraform plan

Do not call the output production-ready unless the requested validation level has actually run and passed.
