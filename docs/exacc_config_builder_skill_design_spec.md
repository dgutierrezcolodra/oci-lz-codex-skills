# ExaCC Config Builder - Skill Design Specification

Date: 2026-05-12

Status: v0.1 design specification

## Purpose

The `exacc-config-builder` skill must make Codex behave like an OCI Landing Zone Operating Entities ExaDB-C@C / ExaCC workload-extension specialist.

The skill is not just a generic OCI helper. It must guide the agent to use the Operating Entities repositories, ExaCC workload-extension templates, Orchestrator contract, dependency outputs, and local naming conventions as the source of truth.

## Source Of Truth

The skill must use official OCI Landing Zone repositories as the source of truth:

- `oci-landing-zone-operating-entities`
- `terraform-oci-modules-orchestrator`

The agent must not rely on generic cloud knowledge or old model memory as the main source for:

- supported blueprints
- supported ExaCC use cases
- source template files
- Orchestrator top-level families
- dependency input names
- output file names
- Resource Manager handoff
- stack boundaries
- identity-domain behavior

Local repositories are not required by default. The skill should use official remote repositories or temporary inspection checkouts when needed. Local checkouts are used only when the user provides them, when reviewing local changes, or when troubleshooting a local run.

For real handoff, the selected source must be pinned to a tag, commit SHA, or resolved branch SHA.

Current temporary source ref:

- `we_exacc_update`
- Use it as the default ExaCC source until the workload extension is merged or published under a stable tag/ref.
- For real handoff, resolve and record the branch SHA.

## Target Users

The skill is for users who need to generate, customize, or review ExaDB-C@C / ExaCC workload-extension Orchestrator JSON.

Typical users:

- landing zone engineers
- cloud architects
- workload-extension owners
- platform teams
- delivery teams preparing Resource Manager or Terraform handoff

## Scope

The skill must support:

- ExaDB-C@C / ExaCC workload-extension design
- One-OE, Multi-OE, Multi-Tenancy, or custom landing-zone context after source inspection
- single-stack and multi-stack deployment models
- fresh Landing Zone plus ExaCC
- existing deployed Landing Zone plus ExaCC extension-only
- dependency-output handling
- OCID fallback when dependency outputs are unavailable
- IAM, governance, observability, identity-domain, and Resource Manager handoff concerns

## Non-Goals

The skill must not:

- act as a generic OCI architecture advisor
- invent unsupported Orchestrator top-level families
- create native `exacc_configuration` or `exacs_configuration` unless the active Orchestrator proves support
- generate final customer-ready JSON before required inputs are known
- silently keep upstream placeholder values
- regenerate the Landing Zone foundation when the user only needs an extension on an existing LZ
- claim production readiness without validation evidence
- run Terraform commands without explicit user approval

## Relationship With Other Skills

The skill must coordinate with related or planned skills:

- `oci-lz-blueprint-builder`: planned skill for when the user needs to design, generate, customize, or review the Landing Zone foundation.
- `oci-lz-workload-extension-builder`: use for non-ExaCC workload extensions.
- `oci-lz-orchestrator-contract-advisor`: use when Orchestrator contract, dependency inputs, outputs, RMS behavior, or runtime troubleshooting must be verified.

The ExaCC skill owns ExaDB-C@C / ExaCC workload-extension behavior. It should not absorb unrelated workload-extension logic.

## Triggering Conditions

The skill should be used when the user asks to:

- generate ExaCC or ExaDB-C@C workload-extension JSON
- customize ExaCC identity, governance, observability, or security files
- review ExaCC workload-extension JSON
- attach ExaCC to an existing Landing Zone
- prepare Resource Manager inputs for ExaCC
- understand ExaCC single-stack or multi-stack deployment
- validate ExaCC dependency outputs or OCID replacements

## Required Intake Behavior

Before generating final artifacts, the skill must ask only the missing questions.

Required classification:

- deployed blueprint: One-OE, Multi-OE, Multi-Tenancy, or custom
- foundation state: already deployed or new
- deployment model: single-stack or multi-stack
- use case: UC1, UC2, UC3, or explicit custom combination
- execution target: Resource Manager, Terraform CLI, or config-only
- source ref and Orchestrator version

For an existing deployed Landing Zone, the skill must ask for:

- deployed blueprint
- hub variant
- CIS level
- region and home region
- Orchestrator version
- output persistence location
- stack/output prefix
- available dependency output files
- identity domain name or OCID
- required groups
- environments and projects in scope
- observability requirements

The skill must treat unanswered customer-facing values as blockers for final generation.

## Use-Case Behavior

The skill must use only the use cases documented in `workload-extensions/exacc/exacc_use_cases/readme.md`:

- UC1: shared ExaDB-C@C platform with shared infrastructure and shared VMCs/AVMCs across environments
- UC2: hybrid ExaDB-C@C platform with shared infrastructure and dedicated VMCs/AVMCs per environment
- UC3: dedicated ExaDB-C@C platform with dedicated infrastructure and VMCs/AVMCs per environment

Current template support:

- UC1 has native JSON templates in the current temporary source ref.
- UC2 and UC3 are documented use-case models, but they are pending completion and do not currently have native template-backed JSON files in `we_exacc_update`.

If the user requests UC2 or UC3 and no dedicated templates exist in the selected ref, the skill must:

- classify the use case correctly
- stop before final generation
- explain that UC2/UC3 are pending completion in the selected source ref
- ask whether the user wants to switch to UC1 or capture requirements/design notes for future UC2/UC3 completion
- avoid relabeling UC1 output as UC2 or UC3

## Deployment Model Behavior

### Fresh Landing Zone + ExaCC

If the user needs a new Landing Zone and ExaCC together, the skill must use the single-stack model unless the user requests another supported approach.

Foundation design is planned to be handled by `oci-lz-blueprint-builder`. Until that skill exists in this package, the ExaCC skill must treat foundation design as separate scope and require explicit, source-verified foundation decisions before final ExaCC generation.

### Existing Landing Zone + ExaCC

If the Landing Zone already exists, the skill must default to the multi-stack / extension-only model.

It must:

- not regenerate the foundation by default
- consume saved dependency outputs when available
- ask for OCIDs only for prior-stack boundaries when outputs are unavailable
- keep generated ExaCC resource keys inside the current stack
- identify required dependencies and OCID replacements separately

## Dependency And Output Behavior

For multi-stack deployments, the skill must distinguish:

- Orchestrator input JSON files
- dependency output files from previous stacks
- OCID fallback values
- outputs produced by the ExaCC stack

Typical dependencies include:

- `compartments_dependency`
- `tags_dependency`
- `topics_dependency` when topics are external
- `logging_dependency` when logs are external

Identity-domain outputs must be handled according to the active Orchestrator contract. The skill must not assume `identity_domains_output.json` is a supported dependency input unless the active contract proves it.

## Generation Behavior

The skill should prefer Jsonnet as the source model for generated artifacts.

Generated artifacts must:

- be complete Orchestrator JSON files
- preserve supported top-level family names
- keep logical references coherent
- replace customer-facing placeholders
- avoid unsupported families
- avoid stale source paths or unrelated workload references
- keep drafts, upstream references, generated samples, and customer-ready outputs clearly separated

The skill must not create invented artifacts such as deployment manifests, profile YAML files, wrapper JSON files, or execution maps unless the user explicitly asks for them.

## Resource Manager Handoff

For Resource Manager, the skill must provide:

- `input_config_files_urls`
- required dependency files
- required OCID replacements
- selected source ref and SHA
- Orchestrator version
- validation status

It must keep config file URLs separate from dependency files and OCID replacements.

## Validation Behavior

The skill must report validation in levels:

- design coherence
- JSON syntax
- Orchestrator contract
- Terraform validate
- Terraform plan

It must not collapse these into a single "correct" or "production-ready" claim.

The skill must not run Terraform commands unless the user explicitly approves the command.

## Expected Delivery

A normal skill response should include:

- selected blueprint
- selected deployment model
- selected use case
- source ref and SHA requirement
- generated or reviewed filenames
- required dependencies
- required OCID replacements
- Resource Manager or Terraform command shape, if requested
- validation commands run
- validation level reached
- remaining assumptions or user decisions

## Publication Maturity

Recommended publication status for v0.1:

- Ready for UC1 guided generation and review.
- Ready for existing Landing Zone extension-only guidance.
- Ready for single-stack vs multi-stack selection.
- Ready for dependency-output and OCID-fallback handling.
- Ready for Orchestrator contract-aware review.
- Not ready for UC2/UC3 generation until those use cases are completed and dedicated templates exist.

Safe public statement:

> The ExaCC skill supports guided ExaDB-C@C workload-extension design and JSON generation. UC1 is supported by the current source templates for single-stack and multi-stack flows. UC2 and UC3 are documented use-case models, but they are pending completion; if dedicated templates are not present in the selected source ref, the skill stops before generation and can only capture requirements or design notes for future UC2/UC3 work.
