# Technical guides

Eight Markdown guides for system administration and troubleshooting. These remain procedure drafts that require technical review and lab validation. Environment identifiers, credentials references, source-specific download links, and screenshots are excluded. Commands are documentation only and have not been executed or tested.

## Draft status

Public release of sanitized drafts is separate from operational approval. Do not use a guide to change a production system until the responsible technical owner has approved and tested its procedure. Redaction alone does not establish classification or release status.

Read [Review requirements](REVIEW.md) before using the guides. Replace angle-bracket placeholders only in an approved operational copy. Never store passwords, tokens, private keys, or live environment identifiers in this collection.

## Linux

- [Recover a RHEL 9 root password](linux/recover-rhel-root-password.md)
- [Update ClamAV definitions offline](linux/update-clamav-definitions.md)

## Networking

- [Update Cisco ASA software](networking/update-cisco-asa.md)
- [Run GP 2.0 troubleshooting checks](networking/gp-2-troubleshooting.md)

## Virtualization

- [Investigate high CPU Ready](virtualization/investigate-cpu-ready.md)
- [Review an ESXi host update](virtualization/update-esxi-host.md)

## Windows

- [Import and link a Group Policy Object](windows/import-link-gpo.md)
- [Review an LDAPS certificate request](windows/request-ldaps-certificate.md)

## Document conventions

- Each guide states its scope, prerequisites, procedure, validation, and unresolved review items.
- Procedures use placeholders for site-specific details. Safeguards, references, and planned validation steps are requirements, not evidence of operational testing.
- Commands and filenames in angle brackets are placeholders, not literal input.
- The original source files and review images are not part of this collection.
