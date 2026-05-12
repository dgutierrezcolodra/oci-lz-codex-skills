# Blueprint Source Map

Use this reference when choosing source templates or verifying current blueprint coverage.

## Canonical Repos

- `https://github.com/oci-landing-zones/oci-landing-zone-operating-entities`
- `https://github.com/oci-landing-zones/terraform-oci-modules-orchestrator`

Default source policy:

- Inspect official remote repositories through standard Git/HTTP/raw URLs.
- Pin production work to a tag or commit SHA.
- Use a requested branch only after recording the resolved SHA and drift risk.
- Use local checkouts only when the customer provides one, when inspecting customer configs, or when troubleshooting a local run.
- If network access is unavailable, ask for a pinned archive, commit checkout, or the exact files; do not work from memory.

Optional local checkout placeholders:

- `/path/to/oci-landing-zone-operating-entities`
- `/path/to/terraform-oci-modules-orchestrator`

Do not assume a user-specific checkout path.

## Blueprint Families

`blueprints/readme.md` identifies:

- `One-OE`: one Operating Entity with environments, platforms, and projects in one tenancy.
- `Multi-OE`: several Operating Entities with shared services and OE-dedicated environments/platforms/projects in one tenancy.
- `Multi-Tenancy`: multiple tenancies using One-OE and Multi-OE patterns for centralized/shared/managed services.

## Runtime Sources

One-OE:

- `blueprints/one-oe/runtime/one-stack/`
- Important files include `oneoe_governance.json`, `oneoe_iam.json`, `oneoe_network_hub_*.json`, `oneoe_security_cis*.json`, and `oneoe_observability_cis*.json`.
- Treat hub variants as source-backed runtime files, not free-form labels. For example, use Hub A/B/C/E only when the matching `blueprints/one-oe/runtime/one-stack/oneoe_network_hub_*.json` file exists in the selected ref.

Multi-OE generic v1:

- `blueprints/multi-oe/generic_v1/runtime/op01_manage_shared_services/`
- `blueprints/multi-oe/generic_v1/runtime/op02_manage_oes/`
- `blueprints/multi-oe/generic_v1/runtime/op03_manage_department/`
- `blueprints/multi-oe/generic_v1/runtime/op04_manage_projects/`

Multi-OE service provider:

- `blueprints/multi-oe/service-providers/runtime/mgmt-plane/`
- `blueprints/multi-oe/service-providers/runtime/mt/shared/`
- `blueprints/multi-oe/service-providers/runtime/pod/customer1/`

Multi-Tenancy:

- Inspect `blueprints/multi-tenancy/readme.md` and the corresponding One-OE/Multi-OE sources for each tenancy role before generating artifacts.

## Orchestrator Output Files

Confirmed output files from the Orchestrator README:

- `compartments_configuration` -> `compartments_output.json`
- `network_configuration` -> `network_output.json`
- `nlb_configuration` -> `nlbs_output.json`
- `notifications_configuration` -> `topics_output.json`
- `streams_configuration` -> `streams_output.json`
- `logging_configuration` -> `service_logs_output.json`, `custom_logs_output.json`
- `vaults_configuration` -> `vaults_output.json`, `keys_output.json`
- `tags_configuration` -> `tags_output.json`
- `instances_configuration` -> `instances_output.json`
- `oke_clusters_configuration` / `oke_workers_configuration` -> `oke_output.json`
- `ocvs_configuration` -> `ocvs_output.json`

## Inspection Commands

```bash
git ls-remote --tags --heads https://github.com/oci-landing-zones/oci-landing-zone-operating-entities
git ls-remote --tags --heads https://github.com/oci-landing-zones/terraform-oci-modules-orchestrator
git clone --depth 1 --branch <tag-or-branch> https://github.com/oci-landing-zones/oci-landing-zone-operating-entities /tmp/oe-inspect
git clone --depth 1 --branch <tag-or-branch> https://github.com/oci-landing-zones/terraform-oci-modules-orchestrator /tmp/orchestrator-inspect
curl -L https://raw.githubusercontent.com/oci-landing-zones/oci-landing-zone-operating-entities/<ref>/blueprints/readme.md
find /path/to/oe/blueprints -maxdepth 6 -type f \( -name '*.json' -o -name '*.yml' -o -name '*.yaml' \) | sort
rg -n "one-stack|multi-stack|Resource Manager|Terraform CLI|output|dependency" /path/to/oe/blueprints
rg -n "output_path|save_output|configuration_source|url_dependency_source|_dependency" /path/to/orchestrator/README.md /path/to/orchestrator/rms-facade
```

## Known Drift Risks

- README links, branch aliases, and output-name tables may lag behind code. When output names matter, prefer `outputs.tf` and `rms-facade/outputs.tf` over README prose.
- Blueprint prose can mention patterns that are not represented by runtime files in a selected ref. If a requested hub or variant is absent from the inspected file list, stop and ask whether to switch refs, use a different official variant, or create a clearly marked tailored blueprint.
- In current checked Orchestrator code, OKE output is written as `oke_output.json`; README tables may still mention `oke_clusters_output.json`.
- Multi-OE generic v1 may differ from newer unpublished/generic v2 design language.
- YAML and JSON examples can coexist; keep one canonical emitted format per customer run unless requested.
- `rms-facade` source precedence and duplicate top-level family behavior must be checked before splitting same-family configs across files.
