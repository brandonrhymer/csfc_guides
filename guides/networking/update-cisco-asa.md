# Update Cisco ASA software

**Status:** Technical and release review required. An ASA reload can interrupt connectivity. Do not execute this draft without an approved device-specific upgrade and rollback plan.

## Scope

Stage an approved ASA image, copy it to device storage, verify it, select it for boot, and validate the device after the approved reload. The source uses a TFTP server and the ASA CLI.

## Prerequisites

- Confirm the ASA model, running release, supported upgrade path, image family, storage capacity, and licensing requirements.
- For a failover pair or cluster, obtain the supported upgrade sequence. This source is not a high-availability runbook.
- Obtain the approved image and authoritative integrity value. The source's private download link and checksum values are omitted.
- Back up the configuration and record the current boot settings through the approved process.
- Confirm console or out-of-band recovery access, maintenance impact, and rollback.
- Use TFTP only if approved for this network. It does not provide authentication or encryption.

## 1 Stage the image

1. Place `<ASA_IMAGE_FILENAME>` in the approved TFTP staging directory.
2. Start the approved TFTP service and select the correct local interface.
3. Verify that the intended device can reach the server and that the server exposes only the intended files.
4. Confirm the image appears in the server's file listing.

## 2 Copy and verify

From the approved management session, enter the required command mode. Replace placeholders before use and confirm command syntax for the target release.

```text
enable
copy tftp: flash:
```

At the prompts, supply `<TFTP_SERVER_ADDRESS>` and `<ASA_IMAGE_FILENAME>`. Confirm the intended destination and wait for transfer completion.

The source verifies the copied file with:

```text
show flash:
verify /md5 disk0:/<ASA_IMAGE_FILENAME>
```

Confirm that `disk0:` is the correct filesystem on this device. Compare the result with the authoritative value for the exact image. Use a stronger vendor-supported verification method where available and required. Stop if verification fails or the integrity value cannot be verified.

## 3 Review and change the boot selection

Record the current selection:

```text
show running-config boot
```

After the approved change owner confirms the boot order and rollback plan, configure the new image:

```text
configure terminal
boot system disk0:/<ASA_IMAGE_FILENAME>
```

The source removes the old boot entry with the following command:

```text
no boot system disk0:/<OLD_ASA_IMAGE_FILENAME>
```

Adding a new boot entry may leave the old image first in the boot order. Retaining the old entry alone does not ensure the new image will boot. Use the supported procedure for this device to make the intended new image the first boot choice, while preserving the approved fallback. Do not remove the old entry unless the approved plan calls for it.

Before saving and again before reload, inspect the effective boot configuration and explicitly verify that the new image has the intended first-boot priority, all referenced files exist, and the remaining order matches the approved rollback plan. Stop if the order cannot be verified. No model-specific reordering command is assumed by this guide.

## 4 Save and reload

Use the release-appropriate commands and approved reload sequence. The source's save-and-reload stage is represented here:

```text
end
write memory
reload
```

Do not reload if the configuration, image, integrity check, new-image boot priority, maintenance window, or recovery path is unresolved.

## 5 Validate

The source checks:

```text
show version
show boot
```

Confirm that the intended release booted and the saved boot settings are correct. Validate interfaces, routing, permitted traffic, management access, and failover or cluster health as applicable. Use the approved rollback plan if validation fails.

## Completion record

Record the approved image, before-and-after versions, integrity-check result, validation outcome, and exceptions in the authorized change record. Do not copy credentials or unrestricted device logs into this guide.

[Return to guide index](../README.md)
