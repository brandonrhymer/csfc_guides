# Maintain MECM site health and handle incidents

**Status:** Draft. Source review: 2026-11-26. Target-system validation required.

[MECM guide index](README.md)

## Service context

This guide covers routine observation, supported maintenance review and incident
coordination for a disconnected, standalone Configuration Manager current-branch
primary site. Confirm installed-version support. Use the actual role map in
[site architecture](site-architecture.md); SQL, SMS Provider, MP, SUP and DP may
be on different hosts. A functioning console does not establish healthy clients
or content delivery.

## Trigger and entry conditions

Use the routine path at the locally approved cadence. Use the incident path for
a service failure, unexpected growth, repeated error, missed maintenance task
or client population that stops reporting. Record the affected service and last
known success before changing anything. Preserve the original symptom when a
temporary recovery masks it.

## Prerequisites and private inputs

Use an approved console host with read access to site/component status, alerts,
content and client health. Maintenance configuration requires the specific site
administration rights. Server event/log access, SQL work, certificate changes
and endpoint administration require separate authorization from those owners;
routine review is not permission to restart shared services.

Maintain a private operations sheet containing site/build and host-role map,
actual log/storage paths, authorized contacts, backup/recovery status, service
dependencies, change calendar, normal workload/queue trends, expected job times,
capacity/alert thresholds and escalation windows. Record **Unknown** where an
input is absent and have its owner resolve it. Keep credentials and private keys
in their approved stores, never in the sheet or this repository.

## Procedure tasks

### 1. Establish the operations calendar

The following is a planning baseline, not an imposed schedule. The service owner
must assign cadence, owner and retained evidence to each row before use.

| Checkpoint | Work and evidence |
| --- | --- |
| Each operating day or staffed shift | New alerts, Site/Component Status, failed scheduled tasks, role/event errors and unexpected queue/storage trends |
| Before and after each transfer/patch cycle | Metadata synchronization, required DP content, client scan/policy health, deployment changes and roster reconciliation |
| Weekly review | Capacity trend, repeated errors, maintenance execution history, SQL/WSUS owner reports and overdue exceptions |
| Each backup run | Verified completion and protected archive; follow the backup procedure's retention/recovery requirements |
| Approved periodic review | Retention settings, access/role map, certificate/CRL readiness, supported versions, recovery exercise and contact accuracy |
| Before site servicing | Approved servicing checklist, dependency checks and current recovery material |

Microsoft recommends scheduled maintenance and operational records; cadence
must fit the site's workload. [Microsoft: maintenance planning](https://learn.microsoft.com/en-us/intune/configmgr/core/servers/manage/maintenance-tasks)

### 2. Inspect status and alerts

1. Connect the console to the intended site. Open **Monitoring > System Status >
   Site Status**. Record each unhealthy site system, role, storage indication and
   observation time. Then inspect **Component Status** for the responsible
   component rather than treating every red icon as the same failure.
2. Select the affected system/component and **Show Messages > All** for a period
   covering the failure and preceding success. Read message IDs, severity,
   description, source and times. Save the relevant evidence privately before
   resetting counters or changing summarizers. An old error count alone does not
   prove the failure is still active. [Microsoft: status system](https://learn.microsoft.com/en-us/intune/configmgr/core/servers/manage/use-status-system)
3. Open **Monitoring > Alerts > Active Alerts**, then **All Alerts**. Check whether
   expected conditions are monitored and whether an alert is postponed or
   disabled. Record ownership and next action. Do not suppress it simply to
   obtain a green screen. If approved email notification is configured, verify
   delivery with its owner; a disconnected site needs a reachable approved
   mail path. [Microsoft: alerts](https://learn.microsoft.com/en-us/intune/configmgr/core/servers/manage/configure-alerts)
4. Check **Monitoring > Software Update Point Synchronization Status** and
   **Monitoring > Distribution Status > Content Status** for current jobs and
   errors. Confirm timing against the authorized disconnected import and package
   changes. Review client-health and deployment trends for the affected population.

**Expected result:** each exception has evidence, an owner and a next observation
or action. Healthy summaries must agree with recent service transactions.

### 3. Verify maintenance execution and capacity

1. Open **Administration > Site Configuration > Sites**, select the site, then
   **Maintenance Tasks** in the details pane. For each expected task, record
   enabled state, schedule, last start, last completion and success result.
   A configured schedule without a completed run is not passing evidence.
   [Microsoft: maintenance-task controls](https://learn.microsoft.com/en-us/intune/configmgr/core/servers/manage/maintenance-tasks#set-up-maintenance-tasks)
2. If a task is late, failed or continuously running, capture its evidence and
   ask the site/DB owner to investigate overlap, resource pressure and logs.
   Do not repeatedly toggle it to force execution. For an approved settings
   change, record original values, use **Edit**, and verify the next execution.
3. Before changing a deletion/retention task, identify dependent discovery,
   reporting and evidence requirements. Check its exact purpose and site-type
   availability in Microsoft's [maintenance-task reference](https://learn.microsoft.com/en-us/intune/configmgr/core/servers/manage/reference-for-maintenance-tasks).
   A shorter retention setting can remove information still needed operationally.
4. On each role host, use the approved storage-monitoring method to record free
   bytes and total size for OS, database/log, WSUS content, source, library and
   backup volumes. Compare with prior measurements and upcoming transfer needs.
   Ask the DBA for database/log growth and job outcomes; do not shrink databases
   as a generic space repair.
5. Observe site-system inbox counts/oldest-file age through the established
   monitoring method. Record path, time, age/count and whether progress continues.
   Correlate a growing backlog with its owning component and recent traffic.
   Do not delete, move or rename inbox files to reduce a counter. Microsoft
   identifies backlogs, SQL/event errors and long-unreported clients as diagnostic
   signals. [Microsoft: monitor the hierarchy](https://learn.microsoft.com/en-us/intune/configmgr/core/servers/manage/monitor-hierarchy)

### 4. Scope an incident and route evidence

1. Record start/detection time, impact, recent changes, site/client versions,
   common subnet/role/update/content identity and actions already attempted.
2. Compare one affected client with a working representative of the same intended
   configuration. If isolated, enter [client health and recovery](client-health-recovery.md).
   If several clients share a failing MP, SUP, DP, trust path or policy change,
   investigate that shared dependency before reinstalling clients.
3. Assign a lead and use the table below. Record actual host/log paths from the
   operations sheet; do not assume every log is on the console host.

| Symptom | Evidence and owner |
| --- | --- |
| Console/provider or database failure | Provider-host `SMSProv.log`, site status, server events; Provider/site owner and DBA |
| MP/policy communication failure | `MPControl.log`, affected client policy/messaging logs; MP/network/PKI owners |
| SUP configuration, health or import failure | Site-server `WCM.log` and `wsyncmgr.log`, SUP-host `WSUSCtrl.log`, import log on the importing WSUS host; SUP/WSUS owner and DBA where relevant |
| Content distribution/download failure | Package ID/version, site-server `distmgr.log` and `PkgXferMgr.log`, DP status and client transfer evidence; source/DP/network owners |
| Missing state/report updates | Client `StateMessage.log`, site `StateSys.log`, affected IDs and times; site/reporting owner |
| Certificate or CRL failure | Endpoint, certificate identity/validity, chain/revocation error and time; PKI/role owner |
| Backup or site-servicing failure | `Smsbkup.log` or `CMUpdate.log`, task/stage status and recovery-set identity; site/DB/recovery owners |

These log roles are documented in Microsoft's
[log-file reference](https://learn.microsoft.com/en-us/intune/configmgr/core/plan-design/hierarchy/log-files).
Read logs with CMTrace or another approved viewer and preserve relevant rollover
files before evidence is overwritten. [Microsoft: log handling](https://learn.microsoft.com/en-us/intune/configmgr/core/plan-design/hierarchy/about-log-files)

## Verification and return to service

1. Record the supported corrective action, executing owner, time and result.
   Verify the original failing operation succeeds; a restarted service is only
   an intermediate observation.
2. Confirm current role checks, status messages and queue/capacity trends. For
   client-facing incidents, prove representative policy receipt, update scan,
   approved content download and state return as applicable using the
   [verification checkpoints](https://github.com/brandonrhymer/csfc_scripts/blob/main/mecm/docs/verification-checkpoints.md).
3. Reconcile affected devices and outstanding deployments with
   [compliance reporting](compliance-reporting.md). Observe for the agreed period.
   The service/workload owner records restored, partially restored or unresolved;
   do not close solely because the console refreshed.

## Exceptions, recovery and escalation

Escalate when the agreed impact/time limit is crossed, evidence stops progressing,
ownership is unclear, corruption is suspected, or a proposed action exceeds the
authorized scope. Supply a private evidence bundle: impact/roster, chronology,
build/topology, exact errors/IDs, logs around the event, recent changes, attempted
actions and recovery readiness. Redact sensitive material before any external
support transfer through the approved release process.

Use the existing [backup/recovery procedure](https://github.com/brandonrhymer/csfc_scripts/blob/main/mecm/docs/backup-recovery.md)
for disaster recovery, [WSUS/SUP maintenance readiness](https://github.com/brandonrhymer/csfc_scripts/blob/main/docs/maintenance/wsus-sup.md)
for database/cleanup decisions, [DP cleanup readiness](https://github.com/brandonrhymer/csfc_scripts/blob/main/docs/maintenance/distribution-point.md)
for orphan assessment, and [offline site servicing](https://github.com/brandonrhymer/csfc_scripts/blob/main/mecm/docs/offline-site-servicing.md)
for site updates. Do not bypass trust, edit the site database, remove library/inbox
files or change the shared connected WSUS as an improvised recovery.

## Closure records

Record completed/missed checks, maintenance execution evidence, trend changes,
incident cause or remaining uncertainty, recovery verification, owners and next
due actions. Update the private role/calendar sheet after authorized changes.
Keep unresolved maintenance and lab acceptance open; source review of this guide
does not establish operational readiness.

## References

Microsoft sources are linked beside their use. Related procedures:
[content operations](content-operations.md), [production patch cycle](production-patch-cycle.md)
and [compliance reporting](compliance-reporting.md).
