# Question Playbooks

Use this file when the user's question matches a common client pattern and you want a consistent answer shape.

## 1. "Which dependency am I missing?"

Check:

- which config family is consuming the reference
- whether the reference is a logical key or an OCID
- which upstream output file should exist

Answer with:

- likely `*_dependency`
- expected output file
- next file or command to inspect

## 2. "Should this go through CLI or RMS?"

Check:

- where configs live
- where dependencies live
- whether outputs must be saved back to GitHub or OCI

Answer with:

- recommended execution path
- why
- critical variables or flags

## 3. "Why was no output file generated?"

Check:

- `output_path` or `save_output`
- working directory
- whether that resource family was actually created

Answer with:

- most likely gating condition
- exact file name expected
- rerun or validation command

## 4. "Which repo/module should I inspect?"

Check:

- configuration family or output file involved

Answer with:

- backing repo
- module subpath when known
- whether the answer is root orchestrator or module specific

## 5. "What IAM policies do I need?"

Check:

- active configuration families
- GitHub or OCI write-back requirements

Answer with:

- module docs to inspect
- high-confidence permission categories
- what still must be confirmed

## 6. "Why does RMS fail but local CLI works?"

Check:

- `rms-facade` working directory
- source variables
- persisted dependency paths

Answer with:

- likely environmental mismatch
- exact variable block to compare

## 7. "How do I split shared services, OE, and workloads?"

Check:

- ownership boundaries
- state boundaries
- dependency publication model

Answer with:

- recommended operation boundaries
- upstream/downstream contracts

## 8. "Did a PR or release change this behavior?"

Check:

- pinned official remote ref first for repo behavior
- customer-provided local checkout only for local configs, uncommitted changes, or failing local runs

Answer with:

- whether the claim comes from a remote repo/ref, local code, or both
- the repo and branch or release compared

## 9. "How do I make this multistack?" or "Why does it pick the wrong JSON?"

Check:

- whether the user is running CLI or `rms-facade`
- which top-level configuration families are repeated across files
- whether the requested platforms actually exist as root orchestrator families
- which upstream outputs each platform needs

Answer with:

- the real collision behavior for the active path
- the recommended operation and state split
- the concrete dependency flow
- any repo limitation or inconsistency that blocks the requested design

## 10. "Should this be one-stack or multi-stack?"

Check:

- production versus PoC intent
- lifecycle coupling tolerance
- ownership and blast-radius requirements
- whether the specific OE workload docs define both options

Answer with:

- recommended stack style
- lifecycle and state tradeoff
- whether the answer comes from generic OE guidance or a workload-specific doc

## 11. "Is this old repo or path the same as the current one?"

Check:

- old repo name or path mentioned
- current repo equivalent
- whether a release note or upgrade note documents the move

Answer with:

- normalized current name
- whether the contract likely still applies
- any migration-sensitive differences that matter
