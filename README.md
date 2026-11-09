# CSfC guides

Sanitized Markdown guides for system administration and troubleshooting. Start with the [guide index](guides/README.md).

The current phase is content development and organization. Testing and full technical review are deferred. These documents are not approved operational runbooks, and no target-system testing has been performed. This repository does not establish CSfC compliance or authorization to change a system.

## Start here

- [Guide index](guides/README.md): eight guides, grouped by platform.
- [Review requirements](guides/REVIEW.md): unresolved technical decisions and release boundaries for the later review phase.
- [Contribution guide](CONTRIBUTING.md): writing, repository layout, and sanitization rules.

## Repository boundaries

Keep standalone administration and troubleshooting guides here. Guides for a script, automation, or program belong beside that implementation in [csfc_scripts](https://github.com/brandonrhymer/csfc_scripts), not as duplicate procedures in this repository.

## Use these drafts safely

Public-release clearance for the sanitized drafts is separate from operational approval. Replace site-specific placeholders only in an approved operational copy. Keep credentials, private keys, live identifiers, logs, source screenshots, and environment-specific evidence out of this repository.

The responsible technical owner must supply missing product and deployment details, complete the later review and test phase, and approve the change procedure before use.
