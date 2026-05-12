# ExaCC Workload Extension Source Map

Use this reference after loading the skill when source paths, template families, or dependency boundaries matter.

## Canonical Source

Preferred source:

`https://github.com/oci-landing-zones/oci-landing-zone-operating-entities`, pinned to the release, tag, commit SHA, or branch selected for the customer engagement.

Current temporary source ref:

- Branch `we_exacc_update`
- Use it as the default ExaDB-C@C workload-extension ref until this work is merged or published under a stable tag/ref.
- For real deployments, always resolve and record the current branch head SHA before generating or handing off configuration.

Relevant path:

`workload-extensions/exacc/`

Orchestrator contract:

- Use the OCI Landing Zones Orchestrator version referenced by the selected source documentation or explicitly selected by the customer.
- For the pre-release/update branch `we_exacc_update`, the inspected Resource Manager examples target Orchestrator `v2.1.0`; re-check this before handoff.
- If a local Orchestrator checkout is older or different, treat it as a source-inspection aid only and call out the version mismatch before giving deployment commands.

Default source policy:

- Inspect the official remote repository through standard Git/HTTP/raw URLs.
- Pin production work to a tag or commit SHA.
- Use a branch such as `we_exacc_update` only after recording the resolved SHA and drift risk.
- Use local checkouts only when the customer provides one, when inspecting customer configs, or when troubleshooting a local run.
- If network access is unavailable, ask for a pinned archive, commit checkout, or the exact files; do not work from memory.

Optional local checkout placeholder:

`/path/to/oci-landing-zone-operating-entities`

Do not assume a user-specific checkout path.

Inspect remote or local sources without switching branches:

```bash
git ls-remote --heads https://github.com/oci-landing-zones/oci-landing-zone-operating-entities <tag-or-branch>
git ls-remote --tags https://github.com/oci-landing-zones/terraform-oci-modules-orchestrator refs/tags/<orchestrator-version>
git clone --depth 1 --branch <tag-or-branch> https://github.com/oci-landing-zones/oci-landing-zone-operating-entities /tmp/oe-inspect
git clone --depth 1 --branch <orchestrator-version> https://github.com/oci-landing-zones/terraform-oci-modules-orchestrator /tmp/orchestrator-inspect
curl -L https://raw.githubusercontent.com/oci-landing-zones/oci-landing-zone-operating-entities/<resolved-sha>/workload-extensions/exacc/readme.md
git -C /path/to/oci-landing-zone-operating-entities ls-tree -r --name-only <ref> workload-extensions/exacc
git -C /path/to/oci-landing-zone-operating-entities show <ref>:workload-extensions/exacc/readme.md
git -C /path/to/oci-landing-zone-operating-entities show <ref>:workload-extensions/exacc/single-stack/exacc_identity_uc1.json | jq 'keys'
```

## Source File Map

Single-stack templates:

- `workload-extensions/exacc/single-stack/exacc_governance_uc1.json`
- `workload-extensions/exacc/single-stack/exacc_identity_uc1.json`
- `workload-extensions/exacc/single-stack/exacc_security_cis1_uc1.json`
- `workload-extensions/exacc/single-stack/exacc_security_cis2_uc1.json`
- `workload-extensions/exacc/single-stack/exacc_observability_cis1_uc1.json`
- `workload-extensions/exacc/single-stack/exacc_observability_cis2_uc1.json`

Multi-stack templates:

- `workload-extensions/exacc/multi-stack/exacc_identity_uc1.json`
- `workload-extensions/exacc/multi-stack/exacc_observability_uc1.json`

Use-case guide:

- `workload-extensions/exacc/exacc_use_cases/readme.md`

Known source quirk:

- README DeployToOCI examples can drift from the actual file list or Orchestrator version. Do not use links as the source of truth until the JSON paths and pinned Orchestrator contract have been inspected.

## Template Families

`single-stack/exacc_governance_uc1.json`:

- `tags_configuration`
- Creates `tagns-lz-role` and `tag-lz-role` when the selected foundation does not already provide them.

`single-stack/exacc_identity_uc1.json`:

- `compartments_configuration`
- `identity_domains_configuration`
- `identity_domain_groups_configuration`
- `policies_configuration`
- Contains One-OE foundation structure plus ExaCC compartments, groups, and policies for use case 1.

`single-stack/exacc_security_cis*_uc1.json`:

- `cloud_guard_configuration`
- `scanning_configuration`
- `security_zones_configuration`

`single-stack/exacc_observability_cis*_uc1.json`:

- `notifications_configuration`
- `events_configuration`
- `home_region_events_configuration`
- `alarms_configuration`
- Some variants also include `logging_configuration` and `service_connectors_configuration`.
- Choose CIS1/CIS2 based on the accepted security baseline after inspecting the corresponding runtime files.

`multi-stack/exacc_identity_uc1.json`:

- `compartments_configuration`
- `identity_domain_groups_configuration`
- `policies_configuration`
- Assumes parent landing-zone compartments and tag definitions already exist or are supplied as dependencies.
- Does not include `identity_domains_configuration`; verify how the active Orchestrator version will create or manage identity-domain groups before treating this file as sufficient by itself.

`multi-stack/exacc_observability_uc1.json`:

- `notifications_configuration`
- `events_configuration`
- `alarms_configuration`
- Assumes existing security, platform, environment, and project compartment keys or OCIDs are available.

## Use-Case Selection

Base use-case selection only on `workload-extensions/exacc/exacc_use_cases/readme.md`. Use these canonical values in customer handoff notes and generated reports:

- `UC1`: Shared ExaDB-C@C Platform: shared infrastructure and shared VMCs/AVMCs across multiple environments.
- `UC2`: Hybrid ExaDB-C@C Platform: shared infrastructure with dedicated VMCs/AVMCs per environment.
- `UC3`: Dedicated ExaDB-C@C Platform: fully dedicated infrastructure and VMCs/AVMCs per environment.

Do not introduce alternate classification fields in intake or generated handoff notes unless the customer explicitly asks for a local alias. If the customer has a mixed model, describe it as an explicit custom combination derived from `UC1`, `UC2`, and `UC3`; only compose it from existing template sections after the customer accepts that it is tailored work rather than native template support. Keep infra and DB administration separated unless the customer explicitly wants a combined team.

Template support in the current temporary ref:

- The enumerated source templates are UC1 templates.
- If a customer requests UC2 or UC3 and the selected ref still lacks dedicated UC2/UC3 templates, do not call UC2/UC3 output native template support.
- Treat UC2/UC3 as a tailored design derived from the documented use-case semantics, ask for explicit approval to tailor the model, and validate every cloned compartment, group, policy, event, alarm, and notification reference before handoff.

## Dependency Guidance

When generating multi-stack configs with logical keys, list required dependencies explicitly:

- `compartments_dependency` for parent compartments and compartment references such as `CMP-LZ-PLATFORM-KEY`, `CMP-LZ-SECURITY-KEY`, environment security/platform/projects keys, and project keys.
- `tags_dependency` when using `defined_tags` that reference a tag namespace/tag created by a previous stack.
- `topics_dependency` only when generated files reference notification topics that are not created in the same operation.
- `logging_dependency` only when generated files reference logs created outside the same operation.

If the customer cannot supply dependency outputs, ask for OCIDs and replace only the boundary references that come from prior stacks. Preserve generated resource keys inside the current JSON set.

## Reference Checks

Before delivering generated files:

```bash
jq . generated-file.json >/dev/null
rg -n "TODO|REPLACE|CHANGE_ME|OCID|oracle-quickstart|oke/simple|master/workload-extensions/oke" output-dir
rg -n "exacc_configuration|exacs_configuration" generated-file.json
```

For Resource Manager/rms-facade, confirm the active Orchestrator merge behavior before distributing multiple files with the same top-level family. If uncertain, produce one complete file per stack boundary instead of relying on cross-file deep merge.
