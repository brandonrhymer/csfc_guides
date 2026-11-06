# Update ClamAV definitions offline

**Status:** Technical and release review required. Confirm the active database location and supported update method before replacing files.

## Scope

Transfer approved ClamAV signature databases from staged media to a managed Linux system. The source uses WinSCP with SCP and identifies `daily.cvd` and `main.cvd` as the files to replace.

## Prerequisites

- Obtain the definition files through the approved update process. Verify origin and integrity.
- Confirm compatibility with the installed ClamAV engine.
- Confirm the target host and approved credentials through the authorized access process.
- Identify the active database directory and whether local policy permits direct database replacement.
- Define backup, service reload, validation, and rollback procedures.

## Procedure

1. Locate the approved update files in the staging directory on the approved media.
2. Open WinSCP. Select a verified saved connection or create one for the approved target.
3. Select SCP if it is the permitted transfer protocol. Authenticate through the approved process.
4. Navigate to the active ClamAV database directory. The source uses `/usr/local/share/clamav`; confirm the installed configuration rather than assuming this path.
5. Transfer the approved `daily.cvd` and/or `main.cvd` files according to the approved replacement procedure.
6. Confirm replacement of the intended database files only. Do not create renamed duplicates as a workaround.
7. Apply the documented service reload or restart step, if required by the deployment.

## If the directory is missing

Stop and confirm the installation path and configuration. The source proposes the following commands, but creating an unused directory does not configure ClamAV to read it:

```bash
sudo mkdir /usr/local/share/clamav
sudo chmod 755 /usr/local/share/clamav
```

Run these only if the technical owner has confirmed that location, ownership, and permissions for this installation. Do not broaden permissions to resolve a transfer failure.

## Validate

The source does not supply a validation method. Before operational use, the technical owner must define how to:

- Verify the installed database version and integrity.
- Confirm the engine has loaded the new definitions.
- Check service health and update errors.
- Run an approved test and restore the previous databases if validation fails.

Record the database versions, completion time, and outcome without including credentials or raw logs in this guide.

## References for technical review

- [ClamAV database FAQ](https://docs.clamav.net/faq/faq-cvd.html) covers signature database files and inspection with `sigtool`.
- [ClamAV configuration guidance](https://docs.clamav.net/manual/Usage/Configuration.html) covers configuration and `clamconf`. Confirm the active database directory, ownership, service user, and applicable SELinux requirements before transferring files.

[Return to guide index](../README.md)
