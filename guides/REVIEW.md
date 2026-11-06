# Review requirements

## Distribution gate

These drafts have been screened for obvious secrets and environment identifiers. That screening is not a security audit, classification determination, declassification action, or public-release approval. The owner confirmed that the appropriate authority cleared the sanitized drafts for public release. This does not establish that the original source material is cleared or that the procedures have been technically validated.

## Guide collection

The collection contains eight guides: RHEL root recovery, Cisco ASA updates, ClamAV definitions, CPU Ready investigation, ESXi update review, GPO import and linking, GP 2.0 troubleshooting, and LDAPS certificate review. See the [guide index](README.md) for links. The source-provenance record is maintained separately and is not included in this collection.

## Redactions and omissions

- Removed the source-specific ASA download URL, fixed image version, checksum values, environment labels, and management-address convention. The appropriate software image and checksum must come from an approved, verified source.
- Generalized the removable-media staging path and removed instructions to retrieve credentials from management notes.
- Replaced GPO environment, domain, and organizational-unit references with placeholders. Removed the source's web domain and retained the named public publisher.
- Replaced the literal connectivity-test address and source-specific service and CLI names in the GP 2.0 note with placeholders. Product scope remains unresolved.
- Removed the source-specific certificate friendly name.
- Omitted all five embedded screenshots from the three-page ESXi RTF. Some reveal a real host identifier and inventory details. The text workflow is preserved with an explicit package-format blocker. No screenshot sanitization or release approval is implied.
- Omitted references to missing screenshots in the RHEL and ClamAV notes. Those images were not supplied with the text files.
- No literal passwords, authentication tokens, or private keys were observed in the text and five inspected screenshots. This limited observation is not proof that all sensitive content has been identified.
- No original screenshots, embedded source metadata, logs, source download links, raw source files, or live environment hostnames, network addresses, or domain names are included in the draft collection. Public vendor documentation links may be included as review references.

## Technical review required

| Guide | Required resolution |
| --- | --- |
| RHEL root recovery | Confirm RHEL version, bootloader access, filesystem state, encryption requirements, and supported SELinux relabel/recovery procedure. Source advice to temporarily disable enforcement is retained only as a review warning, not an instruction. Confirm the backup or snapshot policy. |
| Cisco ASA | Verify model, running release, supported upgrade path, package integrity, command modes, high-availability sequence, boot selection, and rollback. Source checksum values are not an authority. A fixed version is not a recommendation to install that version. |
| ClamAV | Confirm the active database directory, engine compatibility, permitted update mechanism, ownership, service behavior, and how the engine reloads the new databases. The source contains no validation method. |
| CPU Ready | Validate units, sampling interval, vCPU normalization, and workload symptoms before using thresholds. Determine whether virtual socket topology is relevant to this ESXi and guest configuration. The proposed one-socket change is not a general fix or a licensing assurance. |
| ESXi | Resolve the mismatch between the displayed offline depot ZIP and a dialog requesting a VIB. Confirm the supported update method for the exact host release and hardware, maintenance requirements, compatibility, and rollback. Do not execute the source sequence with the depot ZIP on assumption. |
| GPO | Confirm the package, backup/import format, target OU, inheritance, security filtering, conflicts, test scope, and rollback. An imported GPO has no intended effect until its scope is correctly established and policy refresh occurs. |
| GP 2.0 | Identify the product, operating system, approved test destination, service unit, and CLI. The source provides three commands but no expected output, interpretation, or escalation criteria. |
| LDAPS | Repair the unclosed string in the source command, then validate the certificate requirements, names, trust chain, EKU, lifetime, private-key handling, and service selection. A locally created self-signed certificate alone does not establish a working, trusted LDAPS service. |

## Editorial and technical changes

- Added scope, approval gates, prerequisites, validation, rollback or escalation prompts, and completion records. These are review safeguards added during conversion, not claims that the original notes contained them.
- Reframed the CPU Ready guide as an investigation with a conditional topology proposal. Removed the unsupported assurances that the change will lower CPU Ready or cannot affect licensing.
- Preserved the RHEL recovery command sequence while marking SELinux handling as version dependent. Did not prescribe the source's blanket enforcement-disable workaround.
- Separated ASA transfer, integrity verification, boot configuration, save, and reload into distinct stages. The old boot entry must not be removed without an approved rollback plan.
- Preserved the ESXi UI sequence only as a review reference. The package-format mismatch remains unresolved.
- Corrected the LDAP example's missing closing quote and parameterized its duration and label. The example remains incomplete as a deployment design and requires PKI review.
- Replaced missing-image references with text. This is a text-only conversion, not a visually equivalent reproduction.

## Acceptance checklist

- [ ] Technical owner identifies supported product versions and verifies each procedure against current vendor guidance.
- [ ] Technical owner tests the procedure in an approved non-production environment.
- [ ] Change owner defines maintenance impact, backup, rollback, verification, and escalation.
- [x] Owner confirmed appropriate-authority approval for public release of the sanitized drafts.
- [ ] Reviewer confirms no identifiers or sensitive material were reintroduced when placeholders were filled.
- [x] Repository owner requested the sanitized guides in the appropriate repositories.
