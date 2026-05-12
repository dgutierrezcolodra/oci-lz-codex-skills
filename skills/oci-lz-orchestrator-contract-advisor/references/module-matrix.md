# Module Matrix

Use this file when the user asks which module, repo, output, or documentation is behind a configuration family.

## Configuration Families to Backing Modules

| Configuration family | Root module name | Backing repo and subpath | Primary saved output from orchestrator | First follow-up check |
| --- | --- | --- | --- | --- |
| `compartments_configuration` | `oci_lz_compartments` | `oracle-quickstart/terraform-oci-cis-landing-zone-iam//compartments` | `compartments_output.json` | `compartments_dependency` consumers |
| `groups_configuration` | `oci_lz_groups` | `oracle-quickstart/terraform-oci-cis-landing-zone-iam//groups` | none | IAM module docs and outputs |
| `dynamic_groups_configuration` | `oci_lz_dynamic_groups` | `oracle-quickstart/terraform-oci-cis-landing-zone-iam//dynamic-groups` | none | IAM module docs and outputs |
| `policies_configuration` | `oci_lz_policies` | `oracle-quickstart/terraform-oci-cis-landing-zone-iam//policies` | none | IAM module docs and outputs |
| `identity_domains_configuration` | `oci_lz_identity_domains` | `oracle-quickstart/terraform-oci-cis-landing-zone-iam//identity-domains` | `identity_domains_output.json` | consumer contract must be verified |
| `network_configuration` | `oci_lz_network` | `oci-landing-zones/terraform-oci-modules-networking` | `network_output.json` | `network_dependency` consumers |
| `nlb_configuration` | `oci_lz_nlb` | `oci-landing-zones/terraform-oci-modules-networking//modules/nlb` | `nlbs_output.json` | `nlbs_dependency` consumers |
| `streams_configuration` | `oci_lz_streams` | `oci-landing-zones/terraform-oci-modules-observability//streams` | `streams_output.json` | `streams_dependency` consumers |
| `notifications_configuration` | `oci_lz_notifications` | `oci-landing-zones/terraform-oci-modules-observability//notifications` | `topics_output.json` | `topics_dependency` consumers |
| `logging_configuration` | `oci_lz_logging` | `oci-landing-zones/terraform-oci-modules-observability//logging` | `service_logs_output.json`, `custom_logs_output.json` | `logging_dependency` consumers |
| `service_connectors_configuration` | `oci_lz_service_connectors` | `oci-landing-zones/terraform-oci-modules-observability//service-connectors` | none | inspect module outputs and consumer |
| `events_configuration` | `oci_lz_events` | `oci-landing-zones/terraform-oci-modules-observability//events` | none | inspect module outputs and consumer |
| `home_region_events_configuration` | `oci_lz_home_region_events` | `oci-landing-zones/terraform-oci-modules-observability//events` | none | inspect module outputs and consumer |
| `alarms_configuration` | `oci_lz_alarms` | `oci-landing-zones/terraform-oci-modules-observability//alarms` | none | inspect module outputs and consumer |
| `vaults_configuration` | `oci_lz_vaults` | `oci-landing-zones/terraform-oci-modules-security//vaults` | `vaults_output.json`, `keys_output.json` | `vaults_dependency` and `kms_dependency` consumers |
| `bastions_configuration` | `oci_lz_bastions` | `oci-landing-zones/terraform-oci-modules-security//bastion` | `bastions_output.json` | no generic root dependency input |
| `cloud_guard_configuration` | `oci_lz_cloud_guard` | `oci-landing-zones/terraform-oci-modules-security//cloud-guard` | none | inspect module docs |
| `scanning_configuration` | `oci_lz_scanning` | `oci-landing-zones/terraform-oci-modules-security//vss` | none | inspect module docs |
| `security_zones_configuration` | `oci_lz_security_zones` | `oci-landing-zones/terraform-oci-modules-security//security-zones` | none | inspect module docs |
| `zpr_configuration` | `oci_lz_zpr` | `oci-landing-zones/terraform-oci-modules-security//zpr` | none | inspect module docs |
| `budgets_configuration` | `oci_lz_budgets` | `oci-landing-zones/terraform-oci-modules-governance//budgets` | none | inspect module docs |
| `tags_configuration` | `oci_lz_tags` | `oci-landing-zones/terraform-oci-modules-governance//tags` | `tags_output.json` | `tags_dependency` consumers |
| `object_storage_configuration` | root bucket resources | root `buckets.tf` | none | distinguish from `rms-facade` bucket source settings |
| `instances_configuration` | `oci_lz_compute` | `oci-landing-zones/terraform-oci-modules-workloads//cis-compute-storage` | `instances_output.json` | `network_dependency` is common when subnet keys are used |
| `oke_clusters_configuration`, `oke_workers_configuration` | `oci_lz_oke` | `oci-landing-zones/terraform-oci-modules-workloads//cis-oke` | `oke_output.json` | code currently writes `oke_output.json`; verify actual output key shape by execution path |
| `ocvs_configuration` | `oci_lz_ocvs` | `oci-landing-zones/terraform-oci-workloads-ocvs//ocvs/modules/ocvs` | `ocvs_output.json` | `ocvs_dependency` consumers |

## Reading Rules

- If the question is about a root variable, output file, or `rms-facade` input, stay in the orchestrator repo first.
- If the question is about allowed schema, required IAM policies, provider behavior, or resource semantics, inspect the backing module repo next.
- If the row says "none" for saved output, do not invent a generic dependency file contract.
- If README tables and Terraform code disagree, trust `outputs.tf` or `rms-facade/outputs.tf` for the execution path being debugged.
- If the user asks about `functions_dependency`, remember it is an input-side contract used by observability modules, not a generic root configuration family with its own saved output.

## Fast Answers

- "Which repo owns `network_configuration`?" → networking repo.
- "Which repo should I inspect for `compartments_configuration` IAM needs?" → IAM repo `//compartments`.
- "Which repo should I inspect for OKE behavior?" → workloads repo `//cis-oke`.
