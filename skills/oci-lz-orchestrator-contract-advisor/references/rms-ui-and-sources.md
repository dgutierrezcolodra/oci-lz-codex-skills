# RMS UI And Source Selection

Use this reference when the user asks what an ORM field means, which source fields belong together, or why outputs were read from or written to an unexpected place.

## Source Of Truth

When the question is about Resource Manager UI fields, trust `rms-facade/schema.yml` first, then confirm behavior in Terraform code.

Read together:

- `rms-facade/schema.yml`
- `rms-facade/variables.tf`
- `rms-facade/get_configurations.tf`
- `rms-facade/get_dependencies.tf`
- `rms-facade/outputs.tf`

## Core Distinction

Do not conflate these two concepts:

1. `object_storage_configuration`
   - Landing zone resource family
   - creates OCI Object Storage buckets as infrastructure
   - implemented in root `buckets.tf`

2. `oci_configuration_bucket`, `oci_dependency_objects`, `url_dependency_source_oci_bucket`
   - `rms-facade` input locations
   - tell ORM where to read configuration and dependency files
   - do not create those buckets

This distinction is one of the easiest ways to avoid wrong answers.

## Input Source Logic

### `configuration_source = "ocibucket"`

Use:

- `oci_configuration_bucket`
- `oci_configuration_objects`
- optionally `oci_dependency_objects`

Do not use `url_dependency_source` here.

### `configuration_source = "github"`

Use:

- `github_token`
- optionally `github_base_url` for GitHub Enterprise
- `github_configuration_repo`
- `github_configuration_branch`
- `github_configuration_files`
- optionally `github_dependency_files`

Do not use `url_dependency_source` here.

### `configuration_source = "url"`

Use:

- `input_config_files_urls`
- `url_dependency_source`
- the matching `url_dependency_source_*` fields only if dependencies come from GitHub or OCI Object Storage

This is the only mode where `url_dependency_source` matters.

## Output Persistence Rules

`save_output` persists outputs to the same remote family used for dependency files, not to a magical third location.

Typical outcomes:

- config from OCI bucket plus dependency objects in OCI bucket -> outputs saved to OCI bucket
- config from GitHub plus dependency files in GitHub -> outputs saved to GitHub
- config from URL plus `url_dependency_source = "ocibucket"` -> outputs saved to OCI bucket
- config from URL plus `url_dependency_source = "github"` -> outputs saved to GitHub

Use:

- `oci_object_prefix` to avoid object collisions in OCI Object Storage
- `github_file_prefix` to avoid file collisions in GitHub

## Important Constraints

- ORM supports JSON and YAML config files from remote sources.
- Dependency files are JSON only.
- The bucket used for remote config or dependency storage is not automatically created by `rms-facade`.
- Write permissions are required if `save_output = true`.

## Good Answer Pattern

1. Name the active `configuration_source`
2. Name the exact fields that should be set for that mode
3. Explain whether dependency files are local to that source mode or delegated through `url_dependency_source`
4. Name where outputs will be persisted
5. Name the prefix field that prevents collisions

## Useful Checks

```bash
rg -n "configuration_source|url_dependency_source|save_output|oci_object_prefix|github_file_prefix" rms-facade/schema.yml rms-facade/variables.tf
rg -n "count.*configuration_source|count.*url_dependency_source" rms-facade/get_configurations.tf rms-facade/get_dependencies.tf
rg -n "save_output|github_file_prefix|oci_object_prefix" rms-facade/outputs.tf
```
