# Update ClamAV definitions offline

**Status:** Technical review draft. Lab validation required. Confirm the active database location and supported update method before replacing files.

## Scope

Transfer approved ClamAV signature databases from staged media to a managed Linux system. The source uses WinSCP with SCP and names `daily.cvd` and `main.cvd`. These names do not define the complete database set required by every installation. Use the approved update manifest for the target engine.

## Prerequisites

- Obtain the definition files through the approved update process. ClamAV recommends FreshClam or CVDUpdate for acquisition instead of ad hoc manual downloads. For disconnected systems, use the approved connected staging and media-transfer process. Verify origin, integrity, and database versions.
- Confirm compatibility with the installed ClamAV engine.
- Confirm the target host and approved credentials through the authorized access process.
- Identify the active database directory and whether local policy permits direct database replacement.
- Define backup, service reload, validation, and rollback procedures. Coordinate with any running updater so it does not overwrite or race the replacement. Do not disable scanning without an approved maintenance plan.

## Procedure

1. Locate the approved update files in the staging directory on the approved media.
2. Open WinSCP. Select a verified saved connection or create one for the approved target.
3. Select SCP if it is the permitted transfer protocol. Authenticate through the approved process.
4. Navigate to the active ClamAV database directory. The source uses `/usr/local/share/clamav`; confirm the installed configuration rather than assuming this path.
5. Transfer the approved database set to a staging location first. Validate the transfer before replacing active files. Use the deployment's approved replacement method so the engine cannot load a partially copied file.
6. Confirm replacement of the intended database files only. Preserve required ownership and security context. Account for existing CVD or CLD files in the approved plan; do not leave stale copies or create renamed duplicates as a workaround.
7. Apply the documented service reload or restart step, if required by the deployment.

## If the directory is missing

Stop and confirm the installation path and active configuration. Creating `/usr/local/share/clamav` does not make an engine use that directory. Do not create directories or broaden permissions merely to resolve a transfer error.

## Validate

1. Inspect the active configuration with `clamconf` if available. Confirm the database directory used by each scanner and updater. Keep its output in the approved environment because it can include operational details.
2. Inspect each staged and installed CVD database with the installed `sigtool`, for example:

   ```bash
   sigtool --info '<APPROVED_DATABASE_FILE>'
   ```

   Check the reported signature verification, version, and build time against the approved update manifest. Stop on an error, failed verification, or an unexpected version. Confirm the installed tool's behavior for any other database format.
3. Use the deployment's approved reload procedure and inspect its service status and logs. A correct file on disk does not prove a running daemon loaded it.
4. Run an approved harmless test with the actual scanning service. Confirm the expected result and service health. If validation fails, follow the approved rollback and escalation plan.

Record the database versions, completion time, and outcome without including credentials or raw logs in this guide.

## References for technical review

- [ClamAV database FAQ](https://docs.clamav.net/faq/faq-cvd.html) explains supported acquisition tools and database-directory checks.
- [ClamAV signature documentation](https://docs.clamav.net/manual/Signatures.html) describes CVD inspection and signature verification with `sigtool --info`.
- [ClamAV configuration guidance](https://docs.clamav.net/manual/Usage/Configuration.html) covers configuration and `clamconf`. Confirm the active database directory, ownership, service user, and applicable SELinux requirements before transferring files.

[Return to guide index](../README.md)
