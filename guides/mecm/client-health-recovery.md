# Diagnose and recover Windows client health

Status: **Draft — target-system validation required.** Sources reviewed: **2026-11-26**.

## Service context

Use this runbook for Configuration Manager current-branch Windows clients on an
internal network without internet access. Record the installed site/client builds
and supported Windows versions before use. Console labels and applicable checks
must be confirmed on those builds. This document has no live-site acceptance record.

An **online** client has a working client-notification connection to its management
point (MP). **Active** describes recent management activity over the configured
evaluation period. **Client check** reports local evaluation results. An online
icon alone proves neither inventory freshness nor successful application/update
management. Review these states separately. [Microsoft: monitor clients](https://learn.microsoft.com/en-us/intune/configmgr/core/clients/manage/monitor-clients)

## Trigger and entry criteria

Start for a failed client check, missing activity, repeated deployment failure, or
an owner-reported management outage. Identify one affected device before initiating
repair. If failures started together across an MP, boundary group, OS version, or
client version, investigate the shared service first; do not mass-reinstall clients.

## Prerequisites and private inputs

Record these in the authorized incident/change location, outside this repository:

- Device identity, owner, resource ID, assigned primary site, MP, recent changes,
  affected workload, last known success, and approved diagnostic window.
- Expected policy/inventory/check schedules, local timeout and escalation owner.
  Set a specific observation deadline; this guide does not prescribe a health SLA.
- Console access from the administration workstation, scoped device/collection read
  permissions, and **Read Client Status Settings** on the Site object for the
  dashboard. Additional modification rights require the relevant approved action.
- Local administrator access to the affected Windows client through an approved
  channel independent of MECM: physical/virtual console or approved remote access.
  Confirm this before changing a client that supplies the normal management channel.
- For reinstallation: approved complete client source, hashes/version, installation
  properties, free space, restart agreement, and the selected
  [communication-security prerequisites](https://github.com/brandonrhymer/csfc_scripts/blob/main/mecm/docs/communication-security.md).
  HTTPS clients need their intended certificate, private-key access, trusted chain,
  and reachable internal revocation data. A disconnected network still needs these.

## Procedure tasks

### 1. Establish the affected scope

1. On the administration workstation, open **Monitoring > Client Status > Client
   Health Dashboard**. Select the approved collection. Record the collection and
   the filter values, including **Include client health for offline clients**,
   activity days, and **Only show unhealthy client details**. Include offline
   clients when investigating devices absent from the default view.
2. Select the relevant failure segment. Compare the affected records with the
   actual managed-device list and planned outages. Use the OS/client-version tiles
   to identify a common pattern. Record counts with the denominator and timestamp.
3. Refresh and recheck the collection filter; a temporary collection choice can
   reset. Do not alter **Client Status Settings** to make a failure disappear.
   Those settings define evaluation periods, independently of inventory schedules.
   [Microsoft: dashboard filters and permissions](https://learn.microsoft.com/en-us/intune/configmgr/core/clients/manage/client-health-dashboard)
4. In **Assets and Compliance > Devices**, select one affected record. Inspect its
   activity and check details. Add online-status/time columns if needed. Match its
   name and resource ID to the actual machine, then record the last successful
   action and failed check. Stop if identity is ambiguous; use
   [client lifecycle](client-lifecycle.md#3-reassign-or-reconcile-client-identity).

Expected result: a bounded failure description, such as “one client has no recent
policy” or “all clients using one MP stopped reporting,” with preserved timestamps.

### 2. Follow the evidence to the failing layer

1. Connect to the client through the independent access channel. Confirm the
   machine identity, system clock, network address, usable disk space, and any
   pending restart. Check **Services > SMS Agent Host** and its current state.
   Observe first; a planned shutdown or approved service exception is not permission
   to override its owner.
2. Preserve relevant logs before remediation. Client logs normally reside in
   `%windir%\CCM\Logs`; setup logs reside in `%windir%\ccmsetup\Logs`. Use the
   installed CMTrace viewer or an approved text viewer. Correlate device, time,
   error, and preceding successful operation; a red line alone is not a diagnosis.
3. Select the matching branch below. If several apply, resolve setup/identity and
   communication before investigating an individual workload.

| Observation | Evidence and next action |
| --- | --- |
| Client missing or setup failed | Read `ccmsetup.log` and `client.msi.log`. Correct the identified prerequisite/source problem before task 4. |
| Client installed, wrong site or registration failure | Compare Control Panel **Configuration Manager > Site**, `ClientIDManagerStartup.log`, `ClientLocation.log`, and `LocationServices.log` with approved values. Use lifecycle reassignment; do not delete identity files. |
| Correct site, no policy or notification | Inspect `CcmMessaging.log`, `PolicyAgent.log`, and `CCMNotificationAgent.log`. Have network/PKI owners investigate DNS, time, authentication, ports, or CRL failures. A collection script cannot recover a disconnected channel. |
| Failed or overdue client check | Inspect `CcmEval.log` and `CcmEvalTask.log`; use task 3 for the identified failure. |
| Policy works; only inventory, content, or apps fail | Follow [inventory](inventory-operations.md), [content](content-operations.md), or [applications](application-lifecycle.md). Retain client-health evidence; do not assume reinstallation fixes workload configuration. |

Log roles: [Microsoft log reference](https://learn.microsoft.com/en-us/intune/configmgr/core/plan-design/hierarchy/log-files).

### 3. Perform one bounded local remediation

1. If the only issue is an overdue evaluation, open **Task Scheduler > Task
   Scheduler Library > Microsoft > Configuration Manager** on the client. Select
   the **Configuration Manager Health Evaluation** task and inspect its action,
   last run, and current state. Confirm that it invokes the installed `CcmEval.exe`.
   If absent, unexpected, disabled by an approved exception, or already running,
   stop and investigate rather than creating another task.
2. With the approved window active, select **Run** once. It can perform remediation;
   it is not a read-only diagnostic. Wait for completion and review fresh
   `CcmEval.log` results. A successful task-launch result is insufficient.
3. If evaluation identifies a stopped **SMS Agent Host** service, verify that no
   approved stop is in effect. Start it locally through **Services**, then inspect
   the client and Windows event logs. If it stops again, preserve the error and
   escalate. If the service is missing, or evaluation identifies a failure for
   which Microsoft prescribes reinstalling the client, continue to task 4.

Microsoft documents manual evaluation and specific remediation by failed check.
Do not generalize this into WMI repository, Windows Update, or network resets.
[Microsoft: client health checks](https://learn.microsoft.com/en-us/intune/configmgr/core/clients/manage/client-health-checks)

Never restart, repair, or uninstall the MECM agent through its own **Run Scripts**
execution. Any separate maintenance script retains its independent approval and
pilot requirements; this guide does not approve scripts or bypass author/approver
separation.

### 4. Uninstall and reinstall only the identified client

1. Obtain the owner-approved reinstall decision and preserve logs, site assignment,
   effective settings, and pending deployments. Confirm the complete staged source
   and independent access before removing the agent. Stop if source, PKI, or the
   permitted outage is unresolved.
2. On that client, open **Command Prompt as administrator**. Run the documented
   uninstall command:

   ```bat
   %windir%\ccmsetup\ccmsetup.exe /uninstall
   ```

3. Wait for setup to finish. Read `%windir%\ccmsetup\Logs\ccmsetup.log` for the
   completed uninstall; the command does not display an interactive success
   message. If uninstall fails, stop here with its error. Do not delete CCM folders,
   services, registry branches, or WMI namespaces manually.
   [Microsoft: uninstall the client](https://learn.microsoft.com/en-us/intune/configmgr/core/clients/manage/manage-clients#uninstall-the-client)
4. Follow the complete [manual pilot installation procedure](https://github.com/brandonrhymer/csfc_scripts/blob/main/mecm/docs/client-onboarding.md#manually-install-the-pilot-client)
   with the approved source and security branch. Keep all required source files;
   do not install `client.msi` directly. Confirm the final setup result and complete
   any approved restart before validating policy. Review actual site/MP assignment
   because previous installation configuration can affect reinstalls.
   [Microsoft: setup parameters and properties](https://learn.microsoft.com/en-us/intune/configmgr/core/clients/deploy/about-client-installation-properties)

Synthetic example: `LAB-WS01` remains online but reports an overdue check. Its
scheduled task exists and is idle. One approved local run passes; new inventory
and policy receipts also appear. Record recovery without reinstalling it. If the
same run instead identifies a missing client service, preserve evidence and use
the reinstall branch only after source/access checks pass.

## Verification

1. Confirm the intended site, MP, client version, running agent, and fresh successful
   health evaluation on the actual client.
2. Retrieve machine policy using **Control Panel > Configuration Manager > Actions >
   Machine Policy Retrieval & Evaluation Cycle > Run Now**. Verify fresh receipt.
3. Verify the originally failed workload through its linked guide. Refresh the
   console after the recorded reporting window and reconcile client/check/activity
   timestamps. Keep delayed or absent reporting open rather than declaring success.

## Exceptions, recovery, and escalation

Stop repeated repairs if the original failure returns. Shared failures go to the
site/MP/network/PKI owner with a healthy comparison client. WMI/OS corruption goes
to the Windows owner with logs; it is outside this runbook. A failed reinstall
leaves the device unmanaged: retain independent access, record that condition, and
escalate immediately under the locally defined response window.

## Closure records

Retain scope/filter values, before/after timestamps and versions, selected branch,
commands/actions, source hashes, log excerpts, restart result, workload verification,
exceptions, and owner acceptance privately. Record “recovered,” “awaiting reporting,”
or “escalated” explicitly. Public documentation review is not target-system acceptance.

## References

Microsoft references appear beside their procedures. Related operations:
[client settings](client-settings.md), [compliance reporting](compliance-reporting.md),
and [MECM guide index](README.md).
