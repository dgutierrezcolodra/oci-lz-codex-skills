# Blueprints And Extensions

Use this reference only when an orchestrator question explicitly depends on OE blueprint choice, one-stack versus multi-stack workload examples, or workload extensions that are documented around the orchestrator.

## Blueprint Selection

Use the OE repo's blueprint docs and FAQ as source of truth for model choice.

Default guidance:

- `One-OE`: one operating entity in one tenancy
- `Multi-OE`: multiple operating entities sharing one tenancy with shared services
- `Multi-Tenancy`: several tenancies, centralized services, or provider-style models

Do not answer blueprint-selection questions from the orchestrator root repo alone.

Do not use this reference to override the orchestrator contract.

## One-Stack Versus Multi-Stack

Use this framing unless the workload docs say otherwise:

- `one-stack`
  - good for PoC, exploration, faster time-to-value, simpler management
  - usually one state, coupled lifecycle, reduced selectivity during destroy

- `multi-stack`
  - good for production, separate blast radius, independent lifecycle, clearer ownership boundaries
  - usually requires explicit dependency handoff and more coordination

When the user asks about production readiness for an orchestrator-driven workload pattern, bias toward `multi-stack` unless the OE workload docs explicitly optimize for one-stack.

## Add-Ons

Treat OE add-ons as landing-zone complements, not as root orchestrator families.

Common examples in the OE repo:

- hub models
- private DNS
- TBAC
- sovereign controls
- subnetting
- remote peering

If the user asks whether the orchestrator deploys an add-on automatically, verify the specific workload or add-on docs before saying yes.

## Workload Extension Catalog

The OE repo currently exposes extension patterns for:

- OKE
- OCVS
- ExaCC
- EBS
- AI services
- HPC
- Openshift

These extensions answer landing-zone placement and operational sequencing questions even when the root orchestrator does not expose a matching top-level family.

## OKE Guidance

Useful high-confidence statements from the OE repo:

- OKE has both `single-stack` and `multi-stack` extension patterns
- `single-stack` is positioned for PoC or exploration
- `multi-stack` is positioned for production and separate lifecycle
- the orchestrator does not automatically install OKE add-ons after the cluster comes up
- multi-stack OKE may require manual post-updates to hub routing

When the question is about placement, stack choice, or extension steps, inspect OE OKE docs before answering.

## ExaCC Guidance

ExaCC appears in the OE repo as a workload extension layered on top of an existing landing zone foundation.

Use this answer shape:

- ExaCC is part of the OE workload-extension guidance
- it is not a native root configuration family in the current orchestrator snapshot unless `variables.tf` proves otherwise
- orchestrator can still help with the prerequisite foundation and extension-supporting families such as IAM or observability

## OCVS Guidance

OCVS is a special case:

- there is a root `ocvs_configuration` family in the orchestrator
- there is also an OE workload-extension path

If the user asks about pure root contract, stay in orchestrator plus OCVS module repos.
If the user asks about landing-zone placement, post-foundation sequence, or OE architecture, also inspect the OE extension docs.

## Good Answer Pattern

1. Identify whether the question is about model choice, stack boundary, or workload extension execution
2. Name the OE blueprint or extension doc that best fits
3. State the recommended boundary or model
4. Name any dependency or lifecycle tradeoff

## Useful Checks

```bash
rg -n "One-OE|Multi-OE|Multi-Tenancy|one-stack|multi-stack|Hub A|Hub B|Hub C|Hub E" /path/to/oe/repo/faq /path/to/oe/repo/blueprints /path/to/oe/repo/workload-extensions
find /path/to/oe/repo/workload-extensions -maxdepth 3 -iname 'readme.md' -o -iname 'README.md'
```
