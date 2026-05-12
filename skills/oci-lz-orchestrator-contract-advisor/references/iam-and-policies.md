# IAM and Policies

Use this file when the user asks what permissions the orchestrator needs, why a stack gets authorization failures, or which policy docs to inspect.

## Core Rule

The orchestrator's required IAM permissions depend on the configuration families enabled in that stack. Do not answer IAM questions from the root module alone. Map the active configuration families to their backing module repos first.

## Answer Flow

1. List the configuration families present in the stack.
2. Map them through `references/module-matrix.md`.
3. Inspect the backing module documentation for policy requirements.
4. Separate these concerns:
   - permissions needed to run the orchestrator
   - policies the stack creates as OCI resources
   - permissions needed to read or write config and dependency artifacts in GitHub or OCI Object Storage

## High-Confidence Statements

- If the stack enables `compartments_configuration` and `network_configuration`, start with the IAM `//compartments` module docs and the networking module docs.
- For private GitHub configuration sources, the token needs read access, and write access if outputs are being saved back to GitHub.
- For OCI Object Storage sources, the runner needs read access, and write access if outputs are being saved to the bucket.

## Guardrails

- Do not claim a full exact policy set unless it is confirmed from the backing module docs or the user's existing policy files.
- Do not conflate `policies_configuration` as an input family with the IAM permissions needed to run the stack itself.
- If authorization errors appear in RMS, also inspect `save_output`, bucket settings, repo settings, and whether the source type matches the configured credentials.

## Good Response Shape

1. "These configuration families drive the IAM requirements."
2. "These are the backing module repos to inspect."
3. "These permissions are definitely required for config/dependency IO."
4. "These policy statements still need confirmation from the module docs."
