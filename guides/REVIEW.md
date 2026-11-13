# Review requirements

## Operational status

These guides are procedure drafts, not approved operational runbooks. No target-system testing has been performed. The requirements below must be resolved before operational use; they are not completed acceptance evidence. See the [guide index](README.md) for the eight guides.

## Release boundaries

Public release of sanitized drafts does not establish technical validation, operational approval, or release clearance for original source material. Redaction alone is not a security audit, classification determination, declassification action, or public-release approval.

Replace site-specific placeholders only in an approved operational copy. Keep credentials, private keys, live environment identifiers, raw source files, source screenshots, logs, and environment-specific evidence out of this repository. Public vendor documentation links may be included as review references.

## Technical review required

| Guide | Required resolution |
| --- | --- |
| RHEL root recovery | Confirm RHEL version, bootloader access, filesystem state, encryption requirements, and supported SELinux relabel/recovery procedure. This guide uses the RHEL 9 relabel-marker route. Do not substitute policy-load or manual-context commands, or disable enforcement to bypass an error. Confirm the backup or snapshot policy. |
| Cisco ASA | Verify model, running release, supported upgrade path, package integrity, command modes, high-availability sequence, boot selection, and rollback. Obtain the image and authoritative integrity value from an approved, verified source. Do not treat a fixed version as an upgrade recommendation. Verify the new image's first-boot priority and preserve the approved fallback; do not remove the old boot entry without an approved rollback plan. |
| ClamAV | Confirm the active database directory, engine compatibility, permitted update mechanism, ownership, service behavior, and how the engine reloads the new databases. Configuration, database-signature, reload, and scan checks are planned requirements, not completed tests. |
| CPU Ready | Validate units, sampling interval, vCPU normalization, and workload symptoms before using thresholds. Determine whether virtual socket topology is relevant to this ESXi and guest configuration. The proposed one-socket change is not a general fix or a licensing assurance. |
| ESXi | Resolve the mismatch between an offline depot ZIP and a dialog requesting a VIB. Confirm the supported update method for the exact host release and hardware, maintenance requirements, compatibility, and rollback. Do not execute the UI sequence with the depot ZIP on assumption. Profile listing does not establish compatibility, and profile update and install are different operations. |
| GPO | Confirm the package, backup/import format, target OU, inheritance, security filtering, conflicts, test scope, and rollback. An imported GPO has no intended effect until its scope is correctly established and policy refresh occurs. |
| GP 2.0 | Identify the product, operating system, approved test destination, service unit, and CLI. Expected output, interpretation, and escalation criteria remain unspecified. Those product-specific inputs must be supplied before use; do not guess them. |
| LDAPS | Resolve the target-specific issuance method, cryptographic provider, and key specification before use. The cmdlet's default provider differs from the cited Schannel CSP workflow. Validate certificate requirements, names, trust chain, EKU, lifetime, private-key handling, and service selection under the approved PKI design. A locally created self-signed certificate alone does not establish a working, trusted LDAPS service. |

## Acceptance checklist

- [ ] Technical owner identifies supported product versions and verifies each procedure against current vendor guidance.
- [ ] Technical owner tests the procedure in an approved non-production environment.
- [ ] Change owner defines maintenance impact, backup, rollback, verification, and escalation.
- [ ] Reviewer confirms the release status of any new material and checks content and metadata for sensitive information.
- [ ] Reviewer confirms that operational placeholders are resolved in the approved copy without committing live identifiers or sensitive material to this repository.
