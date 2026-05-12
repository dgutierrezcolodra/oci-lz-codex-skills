# Workload Patterns

Use this file when the user asks how workloads fit after the foundational landing zone layers.

## Compute Pattern

Typical flow:

1. Upstream stack creates compartments.
2. Upstream stack creates networking and saves `network_output.json`.
3. Compute stack consumes `compartments_dependency` and often `network_dependency`.
4. Compute stack may also need `kms_dependency` if keys are referenced by logical name.

When the user says the workload uses subnet keys instead of subnet OCIDs, first check `network_dependency`.

## OKE Pattern

Typical flow:

1. Compartments and networking exist first.
2. OKE cluster config consumes logical references to network resources or raw OCIDs.
3. If outputs are needed downstream, the orchestrator writes `oke_output.json`.

Guardrails:

- Do not assume the OKE output key shape. Root and `rms-facade` paths differ.
- Inspect the generated JSON before telling the user whether keys are `clusters` or `oke_clusters`.
- If the question is about cluster schema or node pool behavior, inspect the workloads repo `//cis-oke`.
- If the question is about landing-zone placement, one-stack versus multi-stack, or extension steps, inspect the OE OKE extension docs too.
- OE OKE guidance currently positions `single-stack` toward PoC or exploration and `multi-stack` toward production-style separation.
- OE multi-stack OKE guidance also includes hub routing post-updates and manual add-on installation after cluster deployment.

## OCVS Pattern

Typical flow:

1. Compartments and networking exist first.
2. OCVS configuration consumes network and identity context.
3. Saved downstream handoff is `ocvs_output.json`.

If the question is about cluster semantics or OCVS-specific provider behavior, inspect the OCVS workload repo, not just the root orchestrator.

If the question is about OE architecture, placement in platform compartments, or extension sequencing, inspect the OE OCVS extension docs as well.

## ExaCC Pattern

Treat ExaCC as an OE workload-extension pattern unless the local orchestrator snapshot proves a native root family exists.

Typical answer shape:

1. existing landing zone foundation first
2. extension-supporting IAM, observability, or shared platform structure
3. ExaCC-specific use-case or placement guidance from the OE repo

Guardrails:

- do not invent `exacc_configuration` or `exacs_configuration` root families
- if the user asks about ExaCC placement across shared and dedicated platforms, use the OE extension docs before suggesting folder or operation layouts

## Common Workload Checks

- Are logical keys being used instead of OCIDs?
- Was the upstream output file actually written?
- Does the downstream stack point to the right file, object, or prefix?
- Is the question about the root orchestrator contract or the backing workload module?

## Good Answer Pattern

1. Name the likely dependency.
2. Name the expected upstream output file.
3. Name the backing module repo if deeper schema or IAM detail is needed.
4. Give the next validation command or file to inspect.
