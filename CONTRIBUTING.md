# Contribute a guide

## Choose the repository

Keep standalone administration and troubleshooting guides in this repository. Keep documentation for a script, automation, or program with its implementation in [csfc_scripts](https://github.com/brandonrhymer/csfc_scripts). Link instead of maintaining two copies.

Use Markdown. Put each operational guide under a platform folder in `guides/`. Add it to the [guide index](guides/README.md).

## Write clear instructions

Use ASD-STE100 principles where practical: short sentences, direct verbs, one action per step, and consistent technical terms. This repository does not claim formal STE conformance.

Each guide needs:

- A clear title and status.
- Scope, supported versions, and prerequisites.
- Numbered actions with expected results and stop conditions.
- Validation and rollback or escalation requirements.
- Vendor references when available.
- A link back to the guide index.

State whether a command is a procedure draft, an untested example, or a source fragment with unresolved details. Do not claim lab validation without a dated test record. Do not invent missing product, topology, service, or policy details.

## Protect source material

Do not commit raw source documents, screenshots, logs, credentials, private keys, or live environment identifiers. Use descriptive angle-bracket placeholders for site-specific values. Keep public vendor documentation links separate from site-specific download locations.

A human release decision must address content and metadata. Redaction alone does not establish classification or release status. Keep the original sources and environment-specific evidence in their authorized locations.

If a secret was exposed, stop publication and follow the owner's incident and credential-rotation process. Do not assume deleting its visible line removes it from repository history.

## Separate content from approval

The guides remain untested procedure drafts. Do not add passing-test claims or describe a guide as production-ready without supporting validation and operational approval.

Record missing inputs in the guide and the [review requirements](guides/REVIEW.md). Before operational use, the responsible owners must confirm product applicability, expected results, maintenance impact, rollback, and release status.
