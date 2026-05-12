# Jsonnet Blueprint Contract

Use Jsonnet as the primary generation layer for enterprise blueprint decisions before exporting Orchestrator JSON/YAML. Use optional CUE only as a validation contract when a CUE schema exists.

## Recommended Layout

```text
jsonnet/
  lib/
    orchestrator_families.libsonnet
    dependencies.libsonnet
    stack_boundary.libsonnet
    blueprint_model.libsonnet
  blueprints/
    one_oe.libsonnet
    multi_oe_generic.libsonnet
    multi_oe_service_provider.libsonnet
    multi_tenancy.libsonnet
  templates/
    iam.libsonnet
    governance.libsonnet
    network.libsonnet
    security.libsonnet
    observability.libsonnet
    tooling.libsonnet
  profiles/
    customer.jsonnet
generated/
```

## Required Model Objects

- `customerRequirements`: naming prefix, region code, tenancy model, OE/environment/project hierarchy.
- `blueprintSelection`: official blueprint name, source-backed runtime variant, network hub file when applicable, source branch/path/ref, target execution mode.
- `stackOperation`: operation name, owner, families included, dependencies consumed, outputs produced.
- `dependency`: Orchestrator dependency variable, source file/object, required keys, optional OCID fallback.
- `generatedFile`: filename, top-level families, intended RMS/CLI operation.

## Jsonnet Generation Rules

- Keep reusable libraries in `.libsonnet` files and customer requirement entrypoints small.
- Use object composition and overlays to derive customer variants from inspected source templates.
- Use functions for repeated environment/project expansion, but keep generated keys deterministic.
- Use Jsonnet assertions for local checks such as allowed blueprint names, required fields, and forbidden families.
- Do not use Python or shell string formatting as the production generator.

## Invariants

- Blueprint and runtime variant must be one of the inspected source variants or explicitly marked `tailored`.
- Hub variants must map to inspected runtime files such as `oneoe_network_hub_*.json`; do not model a hub from prose alone.
- Every top-level family must be accepted by the active Orchestrator.
- A logical key reference must resolve in the same stack or in a declared dependency.
- Multi-stack outputs must match the Orchestrator output file names.
- Do not emit duplicate top-level families for the same RMS operation unless verified.
- Do not include workload-only families in foundation blueprint output unless source docs require it.

## Validation Commands

```bash
jsonnetfmt --test jsonnet/**/*.jsonnet jsonnet/**/*.libsonnet
jsonnet -J jsonnet/lib jsonnet/profiles/customer.jsonnet > generated/blueprint.json
find generated -name '*.json' -print0 | xargs -0 -n1 jq . >/dev/null
rg -n "TODO|REPLACE|CHANGE_ME|oracle-quickstart|terraform-oci-open-lz|unsupported_configuration" generated
```

If a CUE validation contract exists, run it against the generated JSON after Jsonnet export:

```bash
cue vet generated/*.json ./cue/...
```

## Deployment Handoff

For each operation, emit:

- Config files or URLs.
- Dependencies to consume: `compartments_dependency`, `network_dependency`, `topics_dependency`, `logging_dependency`, `tags_dependency`, `vaults_dependency`, `kms_dependency`, as applicable.
- Outputs to save: CLI `output_path` or rms-facade `save_output` with OCI Object Storage/GitHub target.
- Human approval points before Terraform/ORM apply.
