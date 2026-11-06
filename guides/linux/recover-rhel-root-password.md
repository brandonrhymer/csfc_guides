# Recover a RHEL root password

**Status:** Technical and release review required. This procedure changes authentication state and requires an approved maintenance window.

## Scope

Recover access to a RHEL virtual machine through its console when an authorized administrator cannot use the existing root password. The exact recovery and SELinux steps depend on the RHEL release and system configuration.

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

The source provides this command sequence. Confirm the filesystem layout before using it:

```bash
mount -o remount,rw /sysroot
chroot /sysroot
passwd
```

Enter the new password only at the password prompt. Stop if the root filesystem is unavailable, cannot be mounted, or is not the expected system.

### 3 Complete SELinux recovery

The source next provides:

```bash
load_policy -i
chcon -t shadow_t /etc/shadow
```

**Review required:** Confirm whether this sequence is supported for the installed RHEL release. It is not an instruction to apply universally. The approved procedure may require filesystem relabeling or other release-specific steps.

The source also suggests temporarily disabling SELinux enforcement after an authentication-token error. Do not apply that workaround automatically. Such errors have multiple possible causes. Diagnose the filesystem state, permissions, security context, and release-specific recovery requirements, then escalate if the approved procedure does not resolve the error.

### 4 Resume normal boot

1. Exit the recovery environment as directed by the approved release-specific procedure. The source uses `Ctrl+D`.
2. Reboot if required.
3. Confirm that `rd.break` is absent from the normal boot command line. Remove it if it was persistently added.
4. Allow any required relabel operation to finish without interruption.

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
