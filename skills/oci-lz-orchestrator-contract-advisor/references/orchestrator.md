# Orchestrator Reference

## Canonical Repositories

Use these official remote repositories as the default source of truth for this skill:

- `https://github.com/oci-landing-zones/terraform-oci-modules-orchestrator`
- `https://github.com/oci-landing-zones/oci-landing-zone-operating-entities`

Pin production answers to a tag or commit SHA. Use a branch only when explicitly requested, and record the resolved SHA plus drift risk. Use local checkouts only for customer-provided configs, local changes, or failing local runs. Use standard Git/HTTP/raw URL inspection; do not use hosted GitHub connectors.

For family-specific schema, IAM, or workload behavior, also inspect:

- `references/module-matrix.md`
- `references/workload-patterns.md`
- `references/iam-and-policies.md`
- `references/provider-and-rms-failures.md`
- `references/question-playbooks.md`

## Read These Files First

Inspect these files before giving prescriptive advice:

- `README.md`
- `SPEC.md`
- `variables.tf`
- `outputs.tf`
- `rms-facade/SPEC.md`
- `rms-facade/variables.tf`
- `rms-facade/schema.yml`
- `rms-facade/get_dependencies.tf`
- `rms-facade/outputs.tf`
- `UPGRADE.md`
- `RELEASE-NOTES.md`

If the question crosses into the OE blueprint, also inspect:

- the OE repo `README.md`
- OE examples or FAQ files that match the operation being discussed

## Source Strategy

### Remote First

Use official remote repositories for:

- blueprint and workload-extension template discovery
- Orchestrator contract checks against a released or requested ref
- tag, release, PR, issue, or branch-history questions
- cross-repo checks when the customer has not supplied a local checkout
- production handoff packages that need repeatable refs

Use standard tooling:

```bash
git ls-remote --tags --heads https://github.com/oci-landing-zones/terraform-oci-modules-orchestrator
git clone --depth 1 --branch <tag-or-branch> https://github.com/oci-landing-zones/terraform-oci-modules-orchestrator /tmp/orchestrator-inspect
curl -L https://raw.githubusercontent.com/oci-landing-zones/terraform-oci-modules-orchestrator/<ref>/README.md
```

### Local Checkouts Are Context, Not Default

Use the customer's local checkout for:

- current branch troubleshooting
- current configuration files
- uncommitted local changes
- drift between docs and local code
- any question tied to a failed local plan or apply

Do not assume default-branch remote contents are identical to local configs or a pinned production ref.

## Deployment Modes

If the question is about ORM field names or source selection, also load `references/rms-ui-and-sources.md`.

### Terraform CLI

Use Terraform CLI when configuration and dependency files are already local or when you want explicit control over state and output paths.

Typical pattern:

```bash
terraform init
terraform plan \
  -var-file ./path/to/credentials.json \
  -var-file ./path/to/config.json \
  -var "output_path=./path/to/output" \
  -state ./runtime/terraform.tfstate \
  -out ./runtime/plan.out
terraform apply -state ./runtime/terraform.tfstate ./runtime/plan.out
```

Rules of thumb:

- Keep one state path per operation or stack boundary.
- Use `output_path` when downstream stacks need generated dependency files.
- Reuse only the generated JSON that the downstream stack actually needs.

### Configuration file collision rules

Do not assume repeated root variables across several `-var-file` inputs will deep-merge.

- Different root variables across files: usually fine
- Same root variable across files: later file overrides earlier file

For multistack guidance or "it picks the wrong JSON" complaints, also load `references/multistack-and-config-collisions.md`.

### OCI Resource Manager via `rms-facade`

Use `rms-facade` when configs or dependency files come from:

- private GitHub repositories
- OCI Object Storage buckets
- public URLs

Rules of thumb:

- In Resource Manager, set the working directory to `rms-facade`.
- `configuration_source` controls where configuration files come from.
- `url_dependency_source` matters only when `configuration_source = "url"` and dependency files live in GitHub or OCI Object Storage.
- Use `save_output = true` when downstream stacks must consume generated files.
- Use `oci_object_prefix` or `github_file_prefix` to avoid overwriting outputs from parallel operations.

### Configuration file collision rules

Do not assume `rms-facade` merges repeated top-level keys across many config files. Confirm precedence in `rms-facade/get_configurations.tf`.

Practical default:

- multiple files with different top-level families: fine
- multiple files redefining the same top-level family: unsafe

For redesign advice, load `references/multistack-and-config-collisions.md`.

## Output and Dependency Contract

Confirm the actual JSON written by the path you are using. The contract is not identical everywhere.

| Output file | Typical top-level JSON keys | Typical downstream input |
| --- | --- | --- |
| `compartments_output.json` | `compartments` | `compartments_dependency` |
| `identity_domains_output.json` | `identity_domains` | inspect the consuming module before advising |
| `network_output.json` | `network_resources` | `network_dependency` |
| `topics_output.json` | `topics` | `topics_dependency` |
| `streams_output.json` | `streams` | `streams_dependency` |
| `service_logs_output.json` | `service_logs` | `logging_dependency` |
| `custom_logs_output.json` | `custom_logs` | `logging_dependency` |
| `vaults_output.json` | `vaults` | `vaults_dependency` |
| `keys_output.json` | `keys` | `kms_dependency` |
| `tags_output.json` | `tags` | `tags_dependency` |
| `nlbs_output.json` | `nlbs_private_ips`, `nlbs_public_ips` | `nlbs_dependency` |
| `ocvs_output.json` | `clusters` | `ocvs_dependency` |

## Resource Families That Are Easy To Misread

- `object_storage_configuration` is a landing zone resource family that creates buckets through root Terraform code.
- It is not the same thing as `oci_configuration_bucket` or other `rms-facade` source variables.
- `functions_dependency` exists as a dependency input for some observability modules, but there is no generic root `functions_configuration` family or generic `functions_output.json` in this repo snapshot.

## Contract Edge Cases Worth Verifying

### README tables can drift from the actual code

When file names in `README.md` disagree with `outputs.tf` or `rms-facade/outputs.tf`, trust the Terraform code path you are actually executing.

Known examples in this repo snapshot:

- README table lists `oke_clusters_output.json`, while the code writes `oke_output.json`
- README table lists `streams.output.json`, while the code writes `streams_output.json`

### ORM UI wording comes from `schema.yml`

When the user asks what a specific ORM variable means or which fields should appear together, trust `rms-facade/schema.yml` over screenshots or paraphrased README text.

Do not answer output-file questions from the README table alone when the code says otherwise.

### OKE output shape differs by execution path

- Root `outputs.tf` writes `oke_output.json` with `clusters`, `node_pools`, and `virtual_node_pools`.
- `rms-facade/outputs.tf` writes `oke_output.json` with `oke_clusters`, `oke_node_pools`, and `oke_virtual_node_pools`.

When troubleshooting OKE chaining, inspect the actual generated file before telling the user which keys exist.

### Compute dependency shape needs confirmation

- Generated compute outputs expose `instances` and `secondary_vnics`.
- `rms-facade/get_dependencies.tf` assembles `instances_dependency` from `instances` and `private_ips`.

Do not assume the dependency contract from memory. Open the generated JSON and the consuming module's schema before prescribing a fix.

### Not every output file has a matching root dependency input

Examples:

- `oke_output.json` is generated, but the root module does not expose `oke_dependency`.
- `bastions_output.json` is generated, but there is no dedicated `bastions_dependency` input in the root module.

If a user wants to chain these resources, verify the exact downstream consumer instead of inventing a generic contract.

## Fast Diagnosis Flow

1. Identify whether the failing stack is CLI or `rms-facade`.
2. Check whether references use logical keys or OCIDs.
3. Identify whether the answer must come from a pinned remote ref or from the customer's local checkout.
4. Map the configuration family through `references/module-matrix.md` if the question is module-specific.
5. Map the missing lookup to the expected dependency input.
6. Confirm the upstream stack was configured to emit the corresponding output file.
7. Confirm the downstream stack points to the exact saved object/file path.

## Useful Search Commands

```bash
rg -n "variable \".*_dependency\"|variable \"output_path\"" variables.tf rms-facade/variables.tf
rg -n "output_file_name|output_path|save_output" outputs.tf rms-facade/outputs.tf README.md
rg -n "configuration_source|url_dependency_source|github_|oci_" rms-facade/variables.tf README.md
```
