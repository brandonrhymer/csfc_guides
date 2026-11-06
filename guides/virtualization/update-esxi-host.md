# Review an ESXi host update

**Status:** Blocked for operational use pending package-format and update-method review. Technical and release approval required.

## Scope

The source describes an ESXi Host Client workflow for selecting an update file from a datastore and installing it through **Manage > Packages > Install update**. It does not provide a complete host-upgrade runbook.

## Source mismatch

The source screenshot shows an offline depot ZIP selected in the datastore browser, while the installation dialog requests a VIB. These are different package concepts. The source does not establish that the selected package is supported by that dialog for the target ESXi release.

Do not paste an offline depot ZIP into that dialog on the assumption that the procedure is valid. The technical owner must verify the supported method for the exact release and package. This finding does not claim that every Host Client version rejects every ZIP package.

## Prerequisites for an approved update

- Confirm the hardware, drivers, firmware, current ESXi release, target release, and management-platform compatibility.
- Obtain the approved image or package and verify its integrity and origin.
- Confirm the supported upgrade path and whether an OEM image is required.
- Back up the host configuration through the supported process.
- Confirm workload migration or shutdown, maintenance mode, recovery access, reboot impact, and rollback requirements.
- Obtain a version-specific procedure from the technical owner before proceeding.

## Source UI sequence for review

The sequence below preserves the source's intended navigation. It is not authorization to install an unresolved package.

1. Open the approved host management address.
2. Authenticate through the approved access process.
3. Select **Storage > Datastores > Datastore browser**.
4. Locate the intended update file and record its datastore path in the authorized change record.
5. Select **Manage > Packages > Install update**.
6. Compare the dialog's accepted package type with the actual file. Stop if they do not match or support is uncertain.
7. Only after the update method is verified and all prerequisites are satisfied, follow the approved procedure for installation and any reboot.

## Validate

The source does not define post-update validation. The technical owner must supply checks for:

- Expected ESXi version and build.
- Host management, networking, storage, and hardware health.
- Correct driver and package state.
- Cluster and management-platform connectivity, where applicable.
- VM availability and workload health after return to service.

## Reference for technical review

[Broadcom guidance on patching an ESXi host using the command line](https://knowledge.broadcom.com/external/article/343840/patching-esxi-host-using-command-line.html) describes a profile-based workflow for an offline depot. It is a review reference, not a replacement procedure tailored to this host. Confirm the exact profile, release applicability, backup, and maintenance requirements before use.

## Conversion note

All five embedded screenshots were omitted. Some contain a real host identifier and inventory details. The screenshots have not been cleared for release. The six text instructions and their intended sequence are represented above, with the package-format blocker made explicit.

[Return to guide index](../README.md)
