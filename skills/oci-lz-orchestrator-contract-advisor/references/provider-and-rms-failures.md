# Provider and RMS Failures

Use this file when the user reports provider/auth errors, Resource Manager problems, source configuration mistakes, or output persistence failures.

## First Checks

- Terraform version is at least `1.3.0`.
- In OCI Resource Manager, the working directory is `rms-facade`.
- `rms-facade/schema.yml` matches the field interpretation being discussed.
- `configuration_source` matches the actual configuration source.
- `url_dependency_source` is only used for URL-based configurations that pull dependencies from GitHub or OCI Object Storage.
- `save_output` is enabled when the user expects output files to be persisted.

## Symptom Table

| Symptom | Likely cause | Checks | First fix |
| --- | --- | --- | --- |
| RMS stack loads but plan cannot find config files | wrong `configuration_source`, wrong repo/bucket/object path, wrong working directory | inspect `configuration_source`, `github_*`, `oci_*`, and working directory | set working directory to `rms-facade` and correct the source-specific fields |
| RMS reads configs but not dependencies | dependency source fields do not match the config source pattern | inspect `url_dependency_source`, dependency file/object fields, and path prefixes | align dependency source fields with the actual source |
| Outputs are not persisted to GitHub or OCI | `save_output` is false, wrong prefix, or missing write permission | inspect `save_output`, `github_file_prefix`, `oci_object_prefix`, repo/bucket permissions | enable output saving and confirm write access |
| GitHub source works for reads but saving fails | token has read-only scope | inspect GitHub token scope and target branch/file prefix | provide write access when `save_output = true` |
| OCI bucket source works for reads but saving fails | object storage write permissions are missing | inspect bucket permissions and object prefix | add write access for the execution principal |
| User thinks the bucket should have been created automatically by ORM source settings | confusion between `rms-facade` source fields and `object_storage_configuration` | inspect whether the question is about remote config storage or landing-zone bucket resources | separate the two concepts and point to the right variable family |
| Provider auth error in CLI | bad credentials, missing variable file, wrong region, or broken private key path | inspect `tenancy_ocid`, `user_ocid`, `fingerprint`, `private_key_path`, `region`, and the actual `-var-file` inputs | fix the provider credentials path and rerun plan |
| Local run and RMS behave differently | different working directory, different source type, or different saved dependency paths | compare effective variables and input paths | normalize the execution path before deeper debugging |

## Guardrails

- Do not blame provider auth first when the evidence points to wrong source selection or missing dependency artifacts.
- Do not answer "RMS is broken" without checking working directory and source settings.
- Treat GitHub Enterprise specifics as repo/provider configuration questions that may require inspecting `github_base_url`.
