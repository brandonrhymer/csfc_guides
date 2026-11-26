# Run an approved MECM production patch cycle

**Status:** Draft. Source review: 2026-11-26. Target-system validation required.

[MECM guide index](README.md)

## Service context

This procedure extends a proven first-patch path into a controlled production
cycle for a disconnected, standalone Configuration Manager current-branch site.
Confirm installed-version support and applicability. Initial setup and transfer
commands remain in [first patches](https://github.com/brandonrhymer/csfc_scripts/blob/main/mecm/docs/03-first-patches.md).
Do not start production until its scan, download, installation, restart and
reporting checkpoints have been demonstrated on the target system.

## Trigger and entry conditions

Start for an approved monthly or emergency update set. The patch owner must
define the required outcome, risk decision and completion date. An emergency
changes the approved schedule; it does not prove content availability or remove
workload recovery requirements. Stop expansion when acceptance is unmet.

## Prerequisites and private inputs

Use the approved console host and an account authorized to manage updates,
deployment packages and deployments for the selected security scopes and
collections. Collection and client-setting changes need their own effective
rights. Endpoint/workload owners provide local access and service checks;
acquisition, SUP/DP and recovery owners provide their respective evidence.

Before deployment, record:

- Change ID, site/build, fixed update IDs/revisions/KBs, products, languages,
  architectures, prerequisites, known issues and vendor recovery restrictions.
- Expected device roster, collection IDs and reviewed membership; pilot and
  production rings, workload order, exclusions and exception owners/expiry.
- Per-ring availability, deadline, time basis, effective maintenance windows,
  restart settings, user communication, observation period and promotion criteria.
- Metadata export/import/synchronization evidence, payload manifest/transfer
  acceptance, deployment package/source/DPs, capacity and successful scan evidence.
- Backup/recovery readiness, workload-owner checks, failure threshold and decision
  authority. No universal percentage, waiting period or deadline is supplied here.

Use [collection targeting](collection-targeting.md),
[client settings](client-settings.md) and
[compliance reporting](compliance-reporting.md) to resolve unknown inputs.

## Procedure tasks

### 1. Freeze and acquire the approved set

1. Confirm the connected source covers the necessary metadata, languages and
   files. Complete approved metadata import and ConfigMgr synchronization;
   retain the matching import and `wsyncmgr.log` results. Do not alter the shared
   connected WSUS to resolve a local gap.
   [Microsoft: disconnected synchronization](https://learn.microsoft.com/en-us/intune/configmgr/sum/get-started/synchronize-software-updates-disconnected)
2. In **Software Library > Software Updates > All Software Updates**, filter by
   approved identities. Remove default date/classification filters if they hide
   required updates. Check supersedence, expiry and prerequisites against the
   update owner's decision; do not silently substitute a newer revision.
3. Select the agreed updates, choose **Create Software Update Group**, and name
   it for this change/cycle. In **Software Update Groups**, select **Show Members**
   and reconcile every member with the fixed list.
   [Microsoft: manual deployment](https://learn.microsoft.com/en-us/intune/configmgr/sum/deploy-use/manually-deploy-software-updates)
4. Review existing deployments before reusing any group. Adding updates to an
   already deployed group can extend deployment automatically; adding devices
   to a targeted collection can also expand exposure. Do not use an ADR or
   mutable shared group as an unreviewed promotion mechanism.
   [Microsoft: deployment behavior](https://learn.microsoft.com/en-us/intune/configmgr/sum/deploy-use/deploy-software-updates)
5. Follow [offline content acquisition](https://github.com/brandonrhymer/csfc_scripts/blob/main/mecm/docs/offline-content-acquisition.md)
   for missing files, dependencies or UUP content. Run the group's **Download**
   wizard using **Download software updates from a location on my network**.
   Require completion for every selected update and successful distribution to
   all intended DPs through [content operations](content-operations.md).

**Expected result:** a reviewed, reproducible update set with both metadata and
installation files. A console listing or successful media copy alone fails this
gate. Preserve the fixed set if acquisition is incomplete and escalate the gap.

### 2. Review windows and workload sequencing

1. In **Assets and Compliance > Device Collections**, select each ring's
   **Properties > Maintenance Windows**. Check effective dates, recurrence,
   start/end, type and UTC/local-time choice. Review other collections containing
   each representative client: overlapping windows can broaden the allowed time.
   If no applicable maintenance window exists, Required updates can install and
   restart after their deadline. Confirm that behavior is approved; leaving
   outside-window override boxes unchecked does not create a window restriction.
2. Confirm update maximum runtime and restart time fit the effective window.
   Inspect the effective client setting controlling use of **All deployments**
   windows when a dedicated **Software updates** window exists. Downloads may
   occur outside windows. Validate received windows in client
   `ServiceWindowManager.log`. [Microsoft: maintenance windows](https://learn.microsoft.com/en-us/intune/configmgr/core/clients/manage/collections/use-maintenance-windows)
3. Have the workload owner define drain/failover, dependency order, minimum
   available capacity and post-restart checks for clustered or redundant services.
   Collection membership alone does not establish a safe server sequence.
4. If approved orchestration groups are already used, verify membership, rules,
   approved pre/post scripts and monitoring with their administrator. They require
   their documented feature/client/permission prerequisites. **User-initiated
   Software Center updates bypass orchestration**, as do Definition updates.
   Do not use a manual Software Center install to prove server orchestration.
   [Microsoft: orchestration groups](https://learn.microsoft.com/en-us/intune/configmgr/sum/deploy-use/orchestration-groups)

### 3. Pilot, then deploy one production ring

1. Start with the pilot only. In **Software Library > Software Updates > Software
   Update Groups**, select the fixed group and **Deploy**. Keep this wizard
   targeted to that pilot through completion. Use **Available** for an approved
   manual test, or **Required** to test scheduled enforcement. An Available test
   must be followed by an accepted Required pilot before relying on scheduled
   enforcement. Deployment type cannot be changed through the wizard afterward.
2. In **Deploy Software Updates**, verify the collection and purpose for this
   ring. On **Scheduling**, enter approved availability, deadline where applicable
   and time basis. Review grace-period and deadline-randomization effects with
   effective client settings. Production rings require **Required** deployments.
3. On **User Experience**, review notifications, installation/restart outside
   maintenance windows and restart suppression for servers/workstations. Leave
   outside-window overrides off unless the change explicitly requires them.
   Suppressing a restart requires an owned later restart; it does not complete
   servicing. On **Download Settings**, ensure reachable internal sources;
   Microsoft Update fallback cannot satisfy a disconnected client.
   [Microsoft: deployment wizard](https://learn.microsoft.com/en-us/intune/configmgr/sum/deploy-use/manually-deploy-software-updates)
4. Review the final summary against the private ring worksheet before completing
   the wizard. Record deployment ID and settings. A template is only a starting
   point; inspect inherited values. If the target or schedule differs, cancel.
5. On representative pilot/ring devices, retrieve Configuration Manager machine
   policy. Confirm scan, content, installation, restart and workload checks using
   the [observable checkpoints](https://github.com/brandonrhymer/csfc_scripts/blob/main/mecm/docs/verification-checkpoints.md).
   Record actual events and times, including pending reboots and failures.
6. Open **Monitoring > Deployments**, select this deployment and **View Status**.
   Reconcile the whole expected roster, including Unknown and missing devices.
   After the approved observation period, the owner records promote, hold or
   recover. Only after pilot acceptance/promotion, repeat steps 1-6 in a new
   wizard for the next approved production ring. Preserve the completed pilot
   deployment and evidence; do not retarget it to promote production.

## Verification and acceptance

Each ring requires the same update identities, correct membership/policy,
successful internal content access, completed applicable installations/restarts,
workload acceptance and current reported results. Record Not Required separately
from Installed. Use [compliance reporting](compliance-reporting.md) for roster,
freshness, exception and final reporting checks. No ring is accepted by this
documentation change.

## Exceptions, recovery and escalation

For a suspected bad update:

1. Stop further rings and notify the patch/workload decision owners. Record the
   first failure, affected update/deployment IDs, clients, symptoms and times.
   Check other groups/deployments containing the update so that another assignment
   cannot continue the same exposure.
2. Have the authorized site operator disable the exact deployment. The following
   **untested procedure example** uses Microsoft's documented cmdlets rather than
   assuming an application-specific console action. Open the console menu's
   **Connect via Windows PowerShell** and confirm the intended site drive before
   running it. Replace the placeholder with the recorded `AssignmentUniqueID`
   GUID, not the numeric assignment ID. [Microsoft: console PowerShell](https://learn.microsoft.com/en-us/powershell/sccm/overview)

   ```powershell
   $patchDeploymentId = '<deployment-GUID>'
   $patchDeployment = @(Get-CMSoftwareUpdateDeployment -DeploymentId $patchDeploymentId -ErrorAction Stop)
   if ($patchDeployment.Count -ne 1) { throw 'Expected exactly one deployment. Stop and reconcile identity.' }
   $patchDeployment[0] | Format-List AssignmentName, AssignmentUniqueID, TargetCollectionID, Enabled
   ```

   Compare the returned identity and collection with the change record. Stop on
   any mismatch. Preview, review the stated target, then execute the approved
   change and reread its state:

   ```powershell
   Set-CMSoftwareUpdateDeployment -InputObject $patchDeployment[0] -Enable $false -WhatIf
   Set-CMSoftwareUpdateDeployment -InputObject $patchDeployment[0] -Enable $false -Confirm
   Get-CMSoftwareUpdateDeployment -DeploymentId $patchDeploymentId | Format-List AssignmentName, AssignmentUniqueID, TargetCollectionID, Enabled
   ```

   Require `Enabled: False`. This disables one assignment, not software updates
   globally. [Microsoft: get deployment](https://learn.microsoft.com/en-us/powershell/module/configurationmanager/get-cmsoftwareupdatedeployment?view=sccm-ps)
   and [modify deployment](https://learn.microsoft.com/en-us/powershell/module/configurationmanager/set-cmsoftwareupdatedeployment?view=sccm-ps)
3. Select the affected device(s) under **Assets and Compliance > Devices > Client
   Notification > Download Computer Policy**, or run **Machine Policy Retrieval
   & Evaluation Cycle** from the client's Configuration Manager control panel.
   Check `PolicyAgent.log`/`PolicyEvaluator.log` and current assignment state on
   representative clients. `gpupdate` refreshes Group Policy, not Configuration
   Manager assignment policy. [Microsoft: client policy retrieval](https://learn.microsoft.com/en-us/intune/configmgr/core/clients/manage/manage-clients#start-client-policy-retrieval-with-client-notification)
4. Track offline/unreachable clients separately; a console change is not proof
   they received it. Disabling does not guarantee cancellation of work already
   executing and does not uninstall completed updates.
   [Microsoft: disable and policy behavior](https://learn.microsoft.com/en-us/intune/configmgr/apps/deploy-use/disable-delete-deployments)
   The update/workload owner
   selects vendor-supported uninstall, mitigation or recovery for each affected
   system; some updates cannot be removed. Preserve logs before recovery.
5. Reconcile actual installed/reboot states and service health. Retest a corrected
   update/recovery on the approved pilot before authorizing further deployment.

For content failures, return to content operations. For scan/policy failures,
use [client health and recovery](client-health-recovery.md); widespread failures
also enter [site operations](site-operations.md). Defer individual devices only
with an owner, reason, expiry, risk decision and next action. Do not remove them
from the expected roster merely to improve the percentage.

## Closure records

The patch owner signs the final roster reconciliation and workload results.
Retain the approved update set, transfer/package evidence, per-ring deployment
settings, promotion decisions, restart evidence, report timestamp, unresolved
devices, exceptions and follow-up dates privately. Mark a cycle **complete with
accepted exceptions** only if the local owner permits that outcome; otherwise
leave it open. Carry unresolved work into the next cycle explicitly.

## References

Microsoft sources are linked at the relevant decisions. Supporting procedures:
[routine operations](https://github.com/brandonrhymer/csfc_scripts/blob/main/mecm/docs/04-maintain.md),
[backup and recovery](https://github.com/brandonrhymer/csfc_scripts/blob/main/mecm/docs/backup-recovery.md),
and [compliance reporting](compliance-reporting.md).
