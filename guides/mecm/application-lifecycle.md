# Package, deploy, update, and retire applications

Status: **Draft — target-system validation required.** Sources reviewed: **2026-11-26**.

## Service context

Use this runbook for Windows Installer (MSI) or vendor EXE applications managed by
Configuration Manager current branch on a disconnected network. Confirm exact
site/client/Windows builds and application support locally. It excludes packages,
task-sequence authoring, cloud stores, and software-update servicing. No operational
test or approval is implied by this draft.

An application contains deployment types that describe how to detect and install
it. A distribution point (DP) holds content; a deployment tells clients what to do.
Creating or distributing an application alone does not install it.

## Trigger and entry criteria

Start with an approved new application, configuration revision, version upgrade,
uninstall, or retirement request. Assign an application owner who can verify useful
operation and supply the vendor-supported installation, upgrade, and recovery
requirements. Stop if those requirements or software rights are unavailable.

## Prerequisites and private inputs

Record the owner, change, publisher/version, license entitlement, supported OS and
architecture, source provenance/hashes, transfer inspection, prerequisite versions,
silent install/uninstall commands, detection evidence, install context, return-code
meanings, restart behavior, maintenance agreement, and acceptance deadline privately.

Use the administration workstation and an RBAC identity permitted to create/edit
the scoped application, distribute its content, and deploy to the approved
collection. These can be different operators. Source-share write access belongs to
the approved packaging owners; grant the site/content-processing identity the
required read access. Confirm the actual topology and account with the site owner.

Prepare a permanent, protected UNC folder containing only this version's required
files. Keep working files, credentials, logs, and unrelated software elsewhere.
Offline installers must include all approved prerequisites and avoid unapproved
online bootstrap downloads. Do not overwrite a live version's source with an
unreviewed upgrade. Use [content operations](content-operations.md) for distribution
and [collection targeting](collection-targeting.md) for the pilot scope.

## Procedure tasks

### 1. Create the application and review its deployment type

1. Open **Software Library > Application Management > Applications > Create
   Application**. For MSI, choose automatic detection, **Windows Installer**, and
   the approved UNC MSI path; inspect imported metadata. For EXE, choose manual
   application information, then add a **Script Installer** deployment type using
   the vendor's commands. An EXE has no universal safe silent-install switch.
2. Enter the reviewed name, publisher, version, and user-facing information. Finish
   creation, select **Deployment Types**, choose the type, and open **Properties**.
3. Review **Content**, **Programs**, **Detection Method**, **User Experience**,
   **Requirements**, **Dependencies**, and **Return Codes** against the input record.
   Use **Detection Method > Add Clause** for a tested MSI product, file/version,
   or registry rule; **Requirements > Add** for applicability; and **Dependencies >
   Add** for prerequisites. Verify dependency **Auto Install** intent. Return-code
   editing is in the existing deployment type's properties.
   [Microsoft: application creation and deployment-type settings](https://learn.microsoft.com/en-us/intune/configmgr/apps/deploy-use/create-applications)

Use this review worksheet before saving. Its pass conditions are local test goals,
not assumed properties of the supplied installer:

| Decision | Evidence required before deployment |
| --- | --- |
| Install/uninstall command | Vendor reference, required transforms/arguments, successful silent operation, and working uninstall content. No credentials embedded in arguments. |
| User or system context | Application owner confirms per-user versus per-machine intent; test with a standard user and the actual deployment context. |
| Detection | False before install, true after install, false after uninstall; old version must not falsely satisfy a new-version rule. Check 32/64-bit location and per-user scope. |
| Requirements/dependencies | Supported OS/architecture and prerequisites actually present or installed; blocked devices remain explained exceptions. |
| Exit codes/restarts | Observed result matches vendor meaning and configured behavior; no failure code relabeled as success to suppress errors. |
| Runtime/user interaction | Installer finishes within the approved runtime/window and does not wait for hidden prompts; owner accepts application-close and restart effects. |

For the repository's MSI helper, follow its existing
[MSI application instructions](https://github.com/brandonrhymer/csfc_scripts/blob/main/mecm/docs/04-maintain.md#add-an-msi-application).
The helper does not supply licensing, transforms, EXE switches, or these acceptance
decisions. Independently review any detection/installation script under the site's
script process; this guide does not grant script approval.

### 2. Distribute and deploy to the pilot

1. Select the application and choose **Distribute Content**. Include required
   dependency content and the approved DPs. Follow content operations until
   **Monitoring > Distribution Status > Content Status** confirms the intended
   content on every needed DP. Stop for failed, missing, or wrong-version content.
2. Recheck pilot membership and all existing deployments of the application or
   earlier versions. Select **Deploy**. On **General**, choose the reviewed
   collection. On **Content**, verify the approved DPs. On **Deployment Settings**,
   choose **Install** and the approved purpose: **Available** permits a Software
   Center request; **Required** enforces the configured schedule. For an Available
   manual pilot, leave **Automatically upgrade any superseded versions of this
   application** disabled unless that automatic behavior is part of the approved
   test. Available alone does not guarantee manual-only installation.
3. Review availability/deadline/time basis on **Scheduling** and notifications,
   installation/restart overrides on **User Experience**. Leave out-of-window
   overrides and implicit uninstall unselected unless explicitly included in the
   change. Review **Summary** against the approved scope before finishing.
   [Microsoft: deploy applications](https://learn.microsoft.com/en-us/intune/configmgr/apps/deploy-use/deploy-applications)
4. On the pilot client, retrieve policy. For the initial Available test, open
   **Software Center > Applications**, select the exact application/version, and
   select **Install** within the agreed test window. A user-initiated Available
   installation can occur outside collection maintenance windows; coordinate it.
5. Verify installation, detection, useful application operation, and the expected
   restart outcome. Test Required scheduling separately if that is the production
   intent; an Available test does not validate deadlines or unattended execution.

Synthetic example: `Example Viewer 2.0` is a fictional machine-wide MSI with an
owner-supplied offline installer. On `LAB-WS01`, record absent detection, successful
install, reported version `2.0`, standard-user launch, and successful uninstall.
On `LAB-WS02` with version `1.0`, verify the agreed upgrade preserves its sample
settings. If the detector only checks for any viewer executable, reject it because
the older executable can falsely pass the new deployment.

### 3. Diagnose before expanding

1. Open **Monitoring > Deployments**, select the deployment, and choose **View
   Status**. Inspect individual assets and details, not only the summary. Record
   successes, in-progress, errors, and requirements-not-met separately. Available
   deployments do not use the Unknown compliance state like Required deployments;
   an unrequested Available app is not a failed install.
   [Microsoft: application monitoring](https://learn.microsoft.com/en-us/intune/configmgr/apps/deploy-use/monitor-applications-from-the-console)
2. On the affected client, inspect `%windir%\CCM\Logs` and the vendor installer log.
   Correlate application/deployment-type identity, revision, and action timestamp.

   | Symptom | Next check |
   | --- | --- |
   | No applicable policy | Membership, deployment purpose/availability, policy receipt, and client health. |
   | Requirements not met | `AppIntentEval.log`, requirements, context, and prerequisites; do not remove a requirement solely to force installation. |
   | Download failure | DP content status, boundary group, `CAS.log`, and transfer logs through content operations. |
   | Already installed or missing after success | `AppDiscovery.log`; compare the actual installed state with detection rules. |
   | Installer failed or needs restart | `AppEnforce.log`, recorded command/context/exit code, and vendor log; compare with return-code and reboot settings. |

   [Microsoft: troubleshooting tips](https://learn.microsoft.com/en-us/troubleshoot/mem/configmgr/app-management/troubleshoot/troubleshoot-application-deployment)
   and [application enforcement](https://learn.microsoft.com/en-us/troubleshoot/mem/configmgr/os-deployment/troubleshoot-install-application-step#step-8-application-deployment-type-is-enforced-by-executing-the-command-line).
3. Correct only the identified cause and repeat the affected test. Keep production
   expansion held until the application owner accepts functionality, target
   exclusions, scheduling, restart behavior, and recovery evidence.

### 4. Select the right change or removal branch

1. **Configuration revision:** retain a record of existing settings and deployments.
   Review **Revision History** before editing. A restored application revision
   restores its configuration; do not treat it as an automatic downgrade of software
   already installed. If changing a live application affects existing deployments,
   test an isolated application copy first.
2. **Source-content correction:** preserve the old source/version, approve the new
   files, and select the deployment type's **Update Content** action. Verify the
   resulting revision/content distribution and retest. Replacing files on the UNC
   share alone does not complete this operation.
   [Microsoft: application management tasks](https://learn.microsoft.com/en-us/intune/configmgr/apps/deploy-use/management-tasks-applications)
3. **New version/supersedence:** create and test the new application. Open its
   **Properties > Supersedence > Add**, select the old application and replacement
   deployment type, and decide whether **Uninstall** is needed before the new
   installation. Base that decision on the vendor's supported upgrade path. Test
   both clean and existing-version clients, dependency relationships, and data
   preservation. Review old/new deployment overlap before release. Distribute
   and deploy the superseding application using task 2; record its collection,
   availability/deadline and the automatic supersedence-upgrade decision. Test
   the chosen automatic or manual behavior on clients with the old version.
   [Microsoft: revisions and supersedence](https://learn.microsoft.com/en-us/intune/configmgr/apps/deploy-use/revise-and-supersede-applications)
4. **Uninstall:** verify the tested uninstall program/content in deployment-type
   properties. Under the approved change, remove existing application, simulated,
   and task-sequence deployments that would reinstall this application before
   creating the removal deployment. If a shared deployment cannot safely be
   removed, stop for the deployment owner's revised targeting plan. Select
   **Deploy > Deployment Settings > Uninstall** for the approved
   removal collection; its purpose is Required. Requirements do not gate an
   uninstall, and dependencies are not automatically removed. Pilot removal and
   verify retained user data as agreed.
5. **Collection removal:** only a deployment explicitly configured for supported
   implicit uninstall uses membership loss to remove its application. Inspect that
   setting before changing membership; do not assume removal either preserves or
   uninstalls the software.
   [Microsoft: uninstall behavior and implicit uninstall](https://learn.microsoft.com/en-us/intune/configmgr/apps/deploy-use/uninstall-applications)
6. **Retirement:** after resolving deployment/removal intent and retaining recovery
   material, select the application and **Retire**. Retirement prevents new
   deployments but does not remove installed software or existing deployments.
   Inspect references before any later deletion; retain required revision/export
   records under the local retention decision.
   [Microsoft: retire and delete behavior](https://learn.microsoft.com/en-us/intune/configmgr/apps/deploy-use/management-tasks-applications#retire)

## Verification

Accept only when the approved manifest reconciles with actual application versions,
detection, owner-tested operation, deployment status, and restart evidence. Require
separate clean-install, upgrade, uninstall, and Required-schedule results when those
paths are in scope. Keep unexplained states and offline clients open.

## Exceptions, recovery, and escalation

For unexpected installs or damage, stop expansion and have the deployment owner
remove/disable the affected intent as supported on the installed version. This
does not undo completed installation or guarantee cancellation of work in progress.
Use the tested vendor recovery/reinstall plan, preserve user data, and verify each
affected device. Escalate missing sources/licensing to the application owner,
distribution failures to the content owner, and client failures to
[client health recovery](client-health-recovery.md).

## Closure records

Keep source hashes, vendor instructions, application/deployment-type IDs and
revisions, deployment/collection IDs, settings, private pilot results, exclusions,
recovery evidence, and owner acceptance. Record retained old-source location and
cleanup decisions. Never attach live logs or license material to this public guide.

## References

Primary references appear at the relevant action. Related guidance:
[compliance reporting](compliance-reporting.md),
[content operations](content-operations.md), and [MECM guide index](README.md).
