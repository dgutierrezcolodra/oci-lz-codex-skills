# Repo Aliases And Upgrade Reference

Use this reference when the user mixes old repo names with current ones, asks whether a behavior changed in a release, or needs to migrate old configs to the current orchestrator.

## What "Alias" Means Here

This does not mean OCI Open LZ as a concept is obsolete.

It means:

- some current docs still use old repo paths
- some current docs still use older branding terms
- some migration-sensitive variables changed names across orchestrator versions

Normalize these names before answering:

- `OCI Open LZ` -> `oci-landing-zone-operating-entities`
- `terraform-oci-open-lz` -> legacy name frequently used by OE docs and examples
- legacy orchestrator under `terraform-oci-open-lz/.../orchestrator` -> current `oci-landing-zones/terraform-oci-modules-orchestrator`

Supporting module repo moves called out in release notes:

- legacy networking repo -> `oci-landing-zones/terraform-oci-modules-networking`
- legacy security repo -> `oci-landing-zones/terraform-oci-modules-security`
- legacy observability repo -> `oci-landing-zones/terraform-oci-modules-observability`
- legacy governance repo -> `oci-landing-zones/terraform-oci-modules-governance`
- legacy secure workloads repo -> `oci-landing-zones/terraform-oci-modules-workloads`

`OCI Open LZ` can be branding around the OE repo and its blueprints. It is not automatically the same thing as the orchestrator itself.

## When To Open Which File

- `UPGRADE.md` for migration-sensitive configuration changes
- `RELEASE-NOTES.md` for feature-introduction dates and bug-fix timing
- current `README.md` when the question is about present behavior

## High-Value Upgrade Changes

### Versions earlier than 2.0.0

Key migration items documented in `UPGRADE.md`:

- `home_region` became `region`
- IAM policy module argument `compartment_ocid` became `compartment_id`
- networking references moved from `*_compartment_key` style to `*_compartment_id`
- OSN service token changed from `all-<region>-services-in-oracle-services-network` to `all-services`

### Releases with behavior that matters in support

- `2.0.1`: outputs can be saved when configs come from plain public URLs, with persistence to private GitHub or OCI buckets
- `2.0.5`: `save_output` no longer gates the Object Storage namespace lookup
- `2.0.6` and `2.0.7`: `nlbs_output` alignment fixes
- `2.0.8`: OKE support added
- `2.0.3`: repo moved into `oci-landing-zones` organization and several module repos were renamed

## Guardrails

- Do not say "this doc is wrong" when the user may simply be on an older path, branding term, or version.
- Do not assume every OE document was updated to the current repo names; many still reference old paths.
- When the user cites an older example, first map the repo or doc alias to the current location and then answer whether the contract still holds.

## Good Response Pattern

1. State whether the user is on a legacy name, legacy version, or both
2. Map the old path or variable name to the current equivalent
3. Name the release or upgrade note that introduced the change
4. Give the minimum config edits needed

## Useful Checks

```bash
rg -n "terraform-oci-open-lz|oci-open-lz|legacy Orchestrator" README.md UPGRADE.md RELEASE-NOTES.md
rg -n "home_region|compartment_ocid|compartment_key|all-services" UPGRADE.md
rg -n "^# .*Release Notes|Support for OKE|save_output|nlbs_output" RELEASE-NOTES.md
```
