# Operating Entities Reference

## Canonical Repository

Treat `https://github.com/oci-landing-zones/oci-landing-zone-operating-entities` as the OE source of truth for this skill.

Use the official remote repository by default. Pin production advice to a tag or commit SHA. Use a branch only when explicitly requested, and record the resolved SHA plus drift risk. Use standard Git/HTTP/raw URL inspection. Use a local checkout only when the customer provides one, when inspecting customer configs, or when troubleshooting a local run.

When the question is about blueprint choice, one-stack versus multi-stack, add-ons, or workload extensions, also load `references/blueprints-and-extensions.md`.

Treat this entire reference as supporting context for orchestrator answers, not as the primary source of truth for the orchestrator contract.

## What This Blueprint Is Good For

Operating Entities style landing zones are useful when the organization wants:

- strong separation of duties
- independent operating teams
- isolated tenancy structures and networking boundaries
- separate lifecycle management for shared services, operating entities, and projects/workloads

The orchestrator fits this model well because each operation can live in its own configuration set, state file, repo path, and output publication path.

The OE repo adds an architectural layer around the root orchestrator:

- blueprint choice
- add-on choice
- workload-extension placement

## Recommended Sequencing

When the user asks how to organize the rollout, default to this order unless their repo says otherwise:

1. Foundation and shared services
2. Operating-entity level structure and controls
3. Project or workload layers that consume earlier outputs

Treat every handoff between these layers as an explicit contract:

- which team owns the stack
- where the state lives
- which output files it publishes
- which dependency files it consumes
- whether configs are local, in GitHub, or in OCI Object Storage

## How To Answer OE Design Questions

When the user asks how to split operations, answer in terms of boundaries:

- identity and shared services boundaries
- network and security boundaries
- workload boundaries
- state and blast-radius boundaries

Good OE advice is usually about ownership and dependency flow, not just about Terraform syntax.

## Workload Guidance

Compute, OKE, and OCVS stacks usually sit after the foundational layers and commonly depend on:

- compartments from IAM or operating-entity setup
- network resources such as VCNs, subnets, NSGs, DRGs, or DNS objects
- vault keys when encryption keys are referenced by logical name
- sometimes tags, logs, or other shared resources

If a workload configuration refers to names or keys instead of raw OCIDs, treat dependency files as required inputs, not optional convenience files.

## Guardrails

- Do not invent operation numbers or folder names for the blueprint repo.
- If the user wants exact operation naming from a customized local OE repo, ask for the local checkout path or the specific files they are using.
- Use the local `examples/oci-landing-zone-operating-entities/README.md` as a sequencing anchor, not as a complete catalog of every possible operation.
- When using remote OE context, say explicitly which repo URL and ref the answer comes from.

## Useful Questions To Ask

- Which operation owns shared services?
- Which operation first creates the compartments your workload uses?
- Where are dependency files saved today?
- Are teams deploying from local files, GitHub, or OCI Object Storage?
- Is the failing reference a compartment key, subnet key, NSG key, vault key, or something else?
