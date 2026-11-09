# Recover a RHEL 9 root password

**Status:** Technical review draft. Lab validation required. This procedure changes authentication state and requires an approved maintenance window.

## Scope

Recover access to a RHEL 9 virtual machine through its console when an authorized administrator cannot use the existing root password. This revision uses Red Hat's RHEL 9 relabel procedure. For another release, obtain its matching recovery procedure before proceeding.

## Prerequisites

- Confirm ownership, authorization, console access, and the supported recovery procedure for the installed RHEL release.
- Confirm the service can be interrupted. Identify dependencies and the maintenance window.
- Take an approved backup or VM snapshot where the workload supports it. A snapshot is not a substitute for a recovery plan.
- Confirm any bootloader protection or disk encryption requirements before rebooting.
- Plan secure handling of the replacement password. Do not put it in this guide, a ticket, or a transcript.

## Procedure

### 1 Open the recovery shell

1. In vSphere, select the VM and open its console.
2. Restart the guest through the approved maintenance procedure.
3. At the boot menu, select the appropriate entry and press `e` to edit it.
4. Append `rd.break` to the kernel command line if the release-specific recovery procedure permits it.
5. Press `Ctrl+X` to continue to the recovery shell.

### 2 Change the password

Confirm that the installed system is RHEL 9 and that `/sysroot` is its root filesystem before using this sequence:

```bash
mount -o remount,rw /sysroot
chroot /sysroot
passwd
```

Enter the new password only at the password prompt. Stop if the root filesystem is unavailable, cannot be mounted, or is not the expected system.

### 3 Schedule SELinux relabeling

After `passwd` reports success, remain inside the changed-root environment. Create the relabel marker there:

```bash
touch /.autorelabel
```

Stop if the marker cannot be created. Do not substitute the original note's `load_policy` and `chcon` commands or disable enforcement to work around a password error. Diagnose filesystem, permission, and recovery-state errors first.

### 4 Resume normal boot

1. Run `exit` to leave the changed-root environment.
2. Run `exit` again to leave the recovery shell and continue boot.
3. Wait for relabeling to finish. It can take time, and the system can restart as part of the process. Do not interrupt it.
4. Confirm that normal boot no longer uses `rd.break`. A one-time GRUB edit does not require changing the saved boot configuration.

## Validate

- Confirm that normal boot completes.
- Verify authorized root access without recording the password.
- Confirm SELinux is in the required operating mode.
- Confirm the VM's services and monitoring recover.
- Record completion, exceptions, and snapshot or backup follow-up under the approved retention policy.

## Stop and escalate

Stop for an unsupported RHEL version, missing recovery credentials, unexpected encrypted storage, filesystem errors, or uncertainty about SELinux recovery. Do not reduce security controls merely to force the password change to succeed.

## Reference for technical review

[Red Hat Enterprise Linux 9 user and group management](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/9/html/configuring_basic_system_settings/managing-users-and-groups_configuring-basic-system-settings) includes a root-password recovery procedure. Its documented relabel route includes `touch /.autorelabel` in the changed-root environment and exiting the recovery shells. Confirm that the installed system is RHEL 9 and use the complete applicable procedure rather than combining fragments from different releases.

[Return to guide index](../README.md)
