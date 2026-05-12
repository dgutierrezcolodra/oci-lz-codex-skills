# Troubleshooting Reference

## Minimum Context To Collect

Before prescribing a fix, get these details when they are missing:

- execution path: Terraform CLI or `rms-facade`
- exact command, plan job, or apply job that fails
- the configuration block involved
- whether the config uses logical keys or OCIDs
- the dependency file path or object name being passed
- whether the question is about a pinned remote repo/ref or a customer-provided local checkout
- the exact error message

## Symptom-Driven Diagnosis

| Symptom | Likely cause | Checks | Typical fix |
| --- | --- | --- | --- |
| A lookup by compartment, subnet, NSG, vault, or stream key fails | Missing or wrong dependency input | Confirm which `*_dependency` variable the consumer expects and inspect the upstream JSON file | Generate or pass the correct output file from the upstream stack |
| `network_output.json` or another dependency file was not created | `output_path` missing, `save_output` disabled, wrong working directory, or no resources of that family were created | Check CLI plan args or RMS variables; verify the stack actually provisions that resource family | Enable output publishing and rerun the upstream stack |
| The orchestrator seems to pick the wrong JSON file | Repeated top-level configuration keys across multiple files and no deep merge | Check whether the user is on CLI or `rms-facade`; inspect repeated root families and file ordering | Redesign as separate operations or collapse each root family into a single operation-level config |
| RMS can read configs but not dependencies | `configuration_source` and dependency source are mixed up | Inspect `configuration_source`, `url_dependency_source`, repo/bucket fields, and object/file paths | Set the source variables for configs and dependencies independently |
| Output file exists but downstream stack still cannot resolve keys | Wrong prefix or object/file path passed to the downstream stack | Compare the saved output path with the exact dependency path configured downstream | Fix the path and rerun the downstream plan |
| OKE chaining advice seems inconsistent | OKE output keys differ between CLI and RMS paths | Open the generated `oke_output.json` instead of assuming the key names | Use the keys that actually exist in the generated file |
| Compute dependency advice seems inconsistent | Generated instance output shape and `instances_dependency` assembly do not line up cleanly | Inspect generated JSON and the consumer's expected schema | Work from the actual file contents, not from memory |
| IAM or permission question cannot be answered cleanly from the root repo | The real contract lives in the backing module docs | Map config families through `references/module-matrix.md` and inspect `references/iam-and-policies.md` | Answer with confirmed permission categories and name the docs that still need checking |
| RMS or provider failure looks unrelated to dependencies | The problem may be source selection, working directory, or provider auth | Inspect `references/provider-and-rms-failures.md` and compare effective source variables | Normalize execution settings before deeper debugging |
| Local behavior disagrees with docs or prior advice | Repo history, release drift, local changes, or recent remote changes may be involved | Inspect the customer's local checkout for local failures; inspect the official remote repo/ref with standard Git/HTTP tools for release or history questions | Explain whether the diagnosis comes from local code, a remote ref, or both before suggesting a fix |
| Destroy fails or leaves residual dependencies | Dependency order is reversed or stacks share too much state | Confirm whether dependents are destroyed before foundations | Destroy workloads first, then foundational stacks, and keep states separated |

## Response Format To Prefer

Use this shape when answering a troubleshooting question:

1. Root-cause hypothesis
2. Source of truth used: remote repo/ref, customer-provided local checkout, or both
3. Exact checks to confirm it
4. Concrete fix with variable names and file names
5. Next validation command or plan step

## Useful Commands

```bash
rg -n "_dependency|output_path|save_output|configuration_source|url_dependency_source" README.md SPEC.md outputs.tf rms-facade
rg -n "compartments|network_resources|keys|topics|streams|clusters" *.json rms-facade 2>/dev/null
terraform plan -var-file ... -out ./runtime/plan.out
```
