# Reconcile MECM update and application compliance

**Status:** Draft. Source review: 2026-11-26. Target-system validation required.

[MECM guide index](README.md)

## Service context

Use this guide to produce an evidence-backed operational report for a
disconnected, standalone Configuration Manager current-branch site. Confirm the
installed build, client/OS support and report availability before use. The main
workflow covers software updates; the application branch records deployment and
detection state. Neither result establishes compliance with an external security
standard by itself.

## Trigger and entry conditions

Start at patch-ring promotion, cycle closure, an application rollout review or
an agreed reporting interval. The report owner must identify the question, target
population, required update/application set and evidence cutoff. A percentage
without those definitions is not an acceptable deliverable.

## Prerequisites and private inputs

Use the approved console host with read rights to the relevant collections,
updates, applications and deployments. Security scopes can limit visible data;
have the site administrator confirm that the reporting identity can see the
whole requested population. Local log inspection requires separately authorized
endpoint access. A client notification is an operational action, not a read-only
report refresh.

For built-in reports, confirm a configured reporting services point and access
to its approved internal endpoint. Microsoft requires **Read** for Site and
**Run Report** for the relevant objects. If these are absent, use the console
branch and disclose its limits; do not install a reporting role to finish a
report. [Microsoft: reporting operations](https://learn.microsoft.com/en-us/intune/configmgr/core/servers/manage/operations-and-maintenance-for-reporting)

Keep these inputs and outputs in an authorized private record:

| Input | Required definition |
| --- | --- |
| Expected roster | Owner-approved asset list with stable identifiers, inclusion/exclusion reasons and as-of time, including unmanaged/missing devices |
| Technical scope | Site/build; collection ID; deployment IDs; update IDs/revisions and group membership, or application/revision/deployment type |
| Freshness | Reporting cutoff, acceptable scan/contact age, synchronization source/coverage/time and agreed observation limit |
| Success rule | Treatment of Not Required, pending reboot, unsupported OS, unavailable devices and approved exceptions |
| Governance | Report recipient, evidence location/retention, exception owner/expiry and reviewer |

The organization supplies freshness and acceptance thresholds. Do not replace
unknown values with a convenient local default.

## Procedure tasks

### 1. Fix the denominator and scope

1. Preserve a dated copy of the expected roster. In **Assets and Compliance >
   Device Collections**, select the target collection and **Show Members**.
   Record the collection ID, evaluated membership and collection evidence time.
2. Reconcile expected devices to resource records. Put absent, duplicate,
   unmanaged, wrongly targeted and intentionally excluded devices into named
   categories. Keep absent devices in the report even though they cannot appear
   in a deployment's client results. Resolve identity using approved inventory,
   not hostname similarity alone; see [inventory operations](inventory-operations.md).
3. In **Software Library > Software Updates > Software Update Groups**, select
   the group and **Show Members**. Match it to the approved update list. Record
   additions, revisions, expirations and exclusions before interpreting results.
   For applications, record whether the assignment is Install or Uninstall and
   Available or Required. Preserve the exact deployment identity.
4. Separate these counts: expected assets, targeted resources, current reporters,
   successful outcomes and unresolved/exempted assets. If a percentage is needed,
   name its numerator and denominator. Example: “80 of 100 expected devices meet
   the defined update outcome; 10 require updates, 5 are stale, 5 are absent.”
   This is a synthetic format, not a site result.

**Expected result:** a reproducible scope that cannot silently shrink when clients
stop reporting. See [collection targeting](collection-targeting.md) for corrections.

### 2. Collect software-update state

1. Open **Monitoring > Deployments**, select the exact update deployment, and
   choose **View Status**. Record category counts and device-level results,
   including Unknown and errors. Match deployment/collection IDs and the evidence
   time to the worksheet. [Microsoft: monitor updates](https://learn.microsoft.com/en-us/intune/configmgr/sum/deploy-use/monitor-software-updates)
2. Read update state separately from deployment progress:

   | State | Meaning for this report |
   | --- | --- |
   | Installed | Applicable update is reported installed; verify required restart and freshness |
   | Not Required | Update is not applicable; this does not prove an installation occurred |
   | Required | Update is needed, or reporting/restart completion has not yet resolved the state |
   | Unknown | No usable state has arrived; investigate scan, communication or processing |

   A successful scan is not a successful installation. Record scan outcome/time,
   installation state and reboot state separately.
   [Microsoft: compliance states](https://learn.microsoft.com/en-us/intune/configmgr/sum/understand/software-updates-introduction#software-updates-compliance-assessment)
3. Confirm catalog coverage and the latest accepted disconnected synchronization.
   A new scan against incomplete or old metadata cannot establish coverage of
   updates missing from that catalog. Reconcile missing products, classifications,
   languages and update identities with the acquisition/SUP owner.
4. Identify unsupported operating systems independently. Microsoft warns these
   can appear compliant when no applicable updates remain. Report them as a
   support/security exception requiring owner disposition, not proof of current
   protection. [Microsoft: reporting caveat](https://learn.microsoft.com/en-us/intune/configmgr/sum/deploy-use/monitor-software-updates#software-updates-reports)

### 3. Use built-in reports when available

1. Go to **Monitoring > Reporting > Reports**. Locate the required report by
   name, select **Run**, enter its required parameters and choose **View Report**.
   Record those parameters and generation time with the retained output.
2. Use **Compliance 9 - Overall health and compliance** for the selected group
   and collection. Its compliance overview concerns active clients; reconcile
   it with the independent roster and client-health counts.
3. Use **Management 2 - Updates required but not deployed** to find deployment
   gaps, and **Troubleshooting 2 - Deployment errors** to group failure codes.
   These report names and purposes are documented by
   [Microsoft](https://learn.microsoft.com/en-us/intune/configmgr/sum/deploy-use/monitor-software-updates#recommended-software-updates-reports).
4. If a report or parameter differs in the installed release, record the actual
   supported report/version and its scope. Do not substitute a dashboard image
   for a missing device-level export without disclosing the limitation.

### 4. Resolve stale or conflicting evidence

1. Compare local and site timestamps for one representative affected client.
   Inspect `%windir%\CCM\Logs\WUAHandler.log` and `ScanAgent.log` for the scan,
   `UpdatesStore.log` for update state, and `StateMessage.log` for reporting.
2. Ask the site owner to trace the same identifiers through the management point
   and site State System (`StateSys.log`). State messages must reach the database
   before summarization can change the displayed result. If receipt is current
   but summary is old, select the specific deployment's **Run Summarization**,
   then refresh and record the new result. Repeated summarization cannot produce
   missing client data. [Microsoft: compliance assessment flow](https://learn.microsoft.com/en-us/troubleshoot/mem/configmgr/update-management/track-software-update-compliance-assessment)
3. If an approved diagnostic refresh is needed, the endpoint owner retrieves
   machine policy and runs **Software Updates Scan Cycle**, then **Software
   Updates Deployment Evaluation Cycle** in the client's Configuration Manager
   control panel. Deployment evaluation can enforce due assignments: review
   impact/window first. Preserve pre-action evidence and wait for actual results.
4. Stop at the agreed reporting cutoff. Mark unresolved state stale/unknown and
   assign an owner; do not keep refreshing until a favorable number appears.

### 5. Application and external-scanner branches

For applications, select the deployment in **Monitoring > Deployments > View
Status**. Distinguish Success, In Progress, Error, Requirements Not Met and Unknown.
Success may mean already detected and can describe an uninstall assignment.
Requirements Not Met is not a successful installation. Unknown is not applicable
to application deployments with a purpose of Available. Inspect deployment-type
detail and the approved detection rule with the application owner before declaring
the intended version present. [Microsoft: application monitoring](https://learn.microsoft.com/en-us/intune/configmgr/apps/deploy-use/monitor-applications-from-the-console)

If Nessus or another scanner is an approved local evidence source, reconcile
asset identity, scan time, authenticated-scan coverage, plugin/feed currency,
finding identity and applicability with its owner. Compare the exact remediation
requirement, including supersedence and required restart. Preserve disagreements
as investigation items; neither system's percentage automatically overrides the
other. This branch supplies reconciliation inputs, not scanner-specific commands.

## Verification and acceptance

A reviewer must reproduce the population, update/application set, report
parameters and cutoff; reconcile every expected asset once; and distinguish
current results from stale, absent, unsupported and exempted devices. Spot-check
representative successful and failed outcomes against endpoint evidence. A
reporting deliverable can be complete while remediation remains open.

## Exceptions, recovery and escalation

Route scan/client failures to [client health and recovery](client-health-recovery.md),
shared reporting/state-processing failures to [site operations](site-operations.md),
content failures to [content operations](content-operations.md), and detection
problems to [application lifecycle](application-lifecycle.md). Retain error codes,
object/device IDs, observation times and last known successful state privately.
Do not edit site database rows, delete clients, or alter collections to correct
the report. Record exception owner, rationale, risk decision, expiry, remediation
action and recheck date; expired exceptions return to unresolved work.

## Closure records

Retain the roster reconciliation, exact numerator/denominator, source coverage,
reports/parameters, evidence timestamps, exceptions, discrepancy decisions and
reviewer signoff privately. Send only the approved summary to its intended
recipient. Feed remaining actions into the
[production patch cycle](production-patch-cycle.md) or application change record.

## References

References are linked beside their use. For end-to-end evidence requirements,
see the existing [verification checkpoints](https://github.com/brandonrhymer/csfc_scripts/blob/main/mecm/docs/verification-checkpoints.md).
