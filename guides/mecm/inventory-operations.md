# Configure, verify, and troubleshoot client inventory

Status: **Draft — target-system validation required.** Sources reviewed: **2026-11-26**.

## Service context

Use this runbook for Configuration Manager current-branch Windows client inventory
on a disconnected internal network. Record the actual site/client builds, supported
Windows versions, and enabled inventory classes. This draft has no target-system
test record. It covers native console operations; custom inventory providers,
database schema changes, and license-compliance decisions require separate work.

Choose the data mechanism before changing settings:

| Information needed | Mechanism and limitation |
| --- | --- |
| Hardware, OS, or installed-program registrations | Hardware inventory collects selected WMI/registry-backed classes. An installed-program class can contain software information despite the feature's name. |
| Executable/file metadata | Software inventory scans the approved file types/paths. It is not a complete installed-program catalog or an application detection rule. |
| Copies of files | File collection copies specified files from clients. It requires separate data-owner review; inventorying metadata does not require collecting file contents. |
| Application install/deployment state | Use [application lifecycle](application-lifecycle.md) and deployment status rather than inferring success from file inventory. |

[Microsoft: inventory mechanisms](https://learn.microsoft.com/en-us/intune/configmgr/develop/core/clients/inventory/about-configuration-manager-inventory)
and [installed-software enumeration limits](https://learn.microsoft.com/en-us/powershell/scripting/samples/working-with-software-installations).

## Trigger and entry criteria

Start for a justified new inventory field, an approved collection/report requirement,
missing or stale client data, or an inventory-scope review. Define the decision the
data supports and an acceptable freshness interval with its owner. Inventory age
must account for the configured schedule and processing delay; this guide does not
set a fleet-wide freshness SLA.

## Prerequisites and private inputs

- Use the administration workstation with the matching console. Obtain RBAC read
  access to the relevant devices/collections, resource data, and client settings.
  Configuration changes require the scoped settings/deployment permissions. Client
  diagnostics need approved local access; server-log review needs the site owner.
- Record the pilot manifest/resource IDs, required classes and properties or file
  patterns/paths, business purpose, exclusions, approved schedule, owner, observation
  deadline, retention, and recovery plan in the authorized private record.
- Verify client assignment, policy communication, and health first. Confirm access
  to the MP/site logs if troubleshooting server processing. Know which host owns
  each role; do not assume they are on one server.
- Review collected data, who can view/export it, and its storage before enabling
  collection. Do not collect credential stores, registry hives, private keys, or
  user documents as troubleshooting shortcuts. Offline storage still needs access
  controls. Inventory is client-reported data, not tamper-proof evidence.
  [Microsoft: software inventory privacy](https://learn.microsoft.com/en-us/intune/configmgr/core/clients/manage/inventory/security-and-privacy-for-software-inventory)
  and [hardware inventory privacy](https://learn.microsoft.com/en-us/intune/configmgr/core/clients/manage/inventory/security-and-privacy-for-hardware-inventory).

## Procedure tasks

### 1. Capture effective settings and existing evidence

1. In **Assets and Compliance > Devices**, select the verified pilot and choose
   **Client Settings > Resultant Client Settings**. Record effective hardware and
   software inventory enablement, schedule, classes, file patterns, and collection
   configuration. Review contributing settings in **Administration > Client
   Settings** using [client settings](client-settings.md).
   [Microsoft: resultant settings](https://learn.microsoft.com/en-us/intune/configmgr/core/clients/deploy/configure-client-settings#view-client-settings)
2. From the device, choose **Start > Resource Explorer**. Under **Hardware**, inspect
   **Workstation Status** and the required classes. Record last hardware inventory
   time, values, and any missing class/property. **Hardware History** can show prior
   changed values; it is not a live query of the client.
   [Microsoft: hardware Resource Explorer](https://learn.microsoft.com/en-us/intune/configmgr/core/clients/manage/inventory/use-resource-explorer-to-view-hardware-inventory)
3. If software inventory is in scope, inspect **Software > Last Software Scan**,
   **File Details**, and **Product Details**. Use **Collected Files** only when
   approved file collection exists. A current hardware timestamp says nothing
   about the software scan's freshness.
   [Microsoft: software Resource Explorer](https://learn.microsoft.com/en-us/intune/configmgr/core/clients/manage/inventory/use-resource-explorer-to-view-software-inventory)

Expected result: a baseline showing what the site already receives, its age, and
the precise missing data. Stop if the selected resource cannot be matched to the
actual pilot; reconcile [client identity](client-lifecycle.md) first.

### 2. Configure only the justified scope

1. For a pilot change, open **Administration > Client Settings > Create Custom
   Client Settings > Create Custom Client Device Settings**. Name it according to
   the site's convention and select the required inventory group. Preserve the
   original settings and priority.
   Avoid changing **Default Client Settings** for a single pilot.
2. For hardware inventory, select **Hardware Inventory**, enable it, set the
   approved schedule, and open **Set Classes**. Select only the reviewed classes
   and properties; record their exact displayed names. For installed programs,
   inspect the available installed-application classes for both relevant
   architectures and compare their fields with a known installed product.
   [Microsoft: inventory settings](https://learn.microsoft.com/en-us/intune/configmgr/core/clients/deploy/about-client-settings#hardware-inventory)
3. Review all contributing policies before expecting a class to disappear. Hardware
   classes are merged across settings; unchecking one in a higher-priority custom
   setting does not disable a class collected elsewhere. A selective reduction can
   require an approved default-policy change and explicitly scoped enablement.
   [Microsoft: hardware inventory configuration](https://learn.microsoft.com/en-us/intune/configmgr/core/clients/manage/inventory/configure-hardware-inventory)
4. For a justified file scan, select **Software Inventory**, enable it, set its
   schedule, and open **Set Types**. Add the approved filename pattern and bounded
   path; decide subfolder and exclusion behavior explicitly. Leave **Collect Files**
   empty unless file-content collection is separately approved. Review size limits
   before any approved collection. Do not scan every file on every disk to locate
   one known executable. Software inventory file lists merge across applicable
   custom policies; an empty list in this policy does not cancel collection
   requested by another. Reconcile all effective policies before reducing scope.
   [Microsoft: software inventory settings](https://learn.microsoft.com/en-us/intune/configmgr/core/clients/deploy/about-client-settings#software-inventory)
5. Save the settings, choose **Deploy**, and select the reviewed pilot collection.
   Confirm the **Deployments** tab, priority, and membership. Retrieve client policy
   on one pilot, then reopen its resultant settings and confirm the intended values.
   Stop if an unrelated device is in scope or a competing policy changes the result.

Custom classes/provider extensions are a separate reviewed change. If a required
class is unavailable, do not import an arbitrary MOF or modify WMI to make it appear.
[Microsoft: extend hardware inventory](https://learn.microsoft.com/en-us/intune/configmgr/core/clients/manage/inventory/extend-hardware-inventory)

### 3. Trace one inventory cycle from client to database

1. On the pilot Windows client, open **Control Panel > Configuration Manager >
   Actions**. Run **Machine Policy Retrieval & Evaluation Cycle** once and verify
   policy receipt. Then run **Hardware Inventory Cycle** or **Software Inventory
   Cycle**, matching the data under test. Record the request time and time zone.
2. Review `%windir%\CCM\Logs\InventoryAgent.log` for the matching action's start,
   completion, and report submission. Use `InventoryProvider.log` for collection
   errors. If another inventory is running, allow it to finish; do not create a
   series of overlapping requests.
3. If the client reports a submission but the site view does not advance, ask the
   MP owner to correlate the client/time in `MP_Hinv.log` for hardware or
   `MP_Sinv.log` for software. The site owner then checks `dataldr.log` or
   `sinvproc.log`, respectively. Client logs normally use `%windir%\CCM\Logs`;
   site logs normally use `<site-install>\Logs`. Confirm the MP role's actual log
   directory with its owner rather than assuming the site-server path.
   [Microsoft: inventory log locations and roles](https://learn.microsoft.com/en-us/intune/configmgr/core/plan-design/hierarchy/log-files)
4. Refresh Resource Explorer and compare its timestamp and required fields with
   the baseline. Hardware data travels client → MP → site server → site database;
   successful collection is only the first stage. Initial hardware inventory is
   full, later reports are normally deltas; the site can request resynchronization
   when a delta is missing.
   [Microsoft: inventory flow](https://learn.microsoft.com/en-us/intune/configmgr/core/clients/manage/inventory/introduction-to-hardware-inventory)

### 4. Resolve the missing or stale-data branch

| Observation | Action and stop condition |
| --- | --- |
| No client cycle starts | Verify health, policy receipt, effective enablement, and schedule. Use [client health recovery](client-health-recovery.md) if these fail. |
| Cycle completes but a class is absent | Check the exact class/property in effective settings and its presence on the supported OS. Escalate provider errors; do not reset WMI. |
| File scan omits the expected file | Check pattern, path, subfolders, attributes, and exclusions. Check for a hidden `Skpswi.dat` exclusion with the owner; do not remove an approved exclusion. |
| Provider reports `80041006` | Review scan scope with the owner. Microsoft identifies this as a provider memory-quota condition; reduce unnecessary scope before specialist tuning. |
| Submission succeeds but MP/site processing fails | Preserve the matching timestamps/errors and escalate to the role owner. Do not delete inbox queues or edit database rows. |
| Data is current but the report/collection is stale | Verify report parameters, resource identity, query fields, and collection evaluation separately. Inventory collection success does not force collection membership evaluation. |

Software-scan exceptions: [Microsoft configuration guidance](https://learn.microsoft.com/en-us/intune/configmgr/core/clients/manage/inventory/configure-software-inventory).

If evidence identifies a resynchronization problem, the site owner can authorize
one full inventory through an already approved **Support Center Client Tools** installation:
**Inventory > Invoke trigger > Advanced > Hardware inventory cycle (full
resynchronization)**, or the corresponding software action. Confirm the connected
client before triggering and trace it again. Stop if that UI/tool is unavailable;
do not replace it with undocumented inventory-history deletion scripts.
[Microsoft: Support Center inventory actions](https://learn.microsoft.com/en-us/intune/configmgr/core/support/support-center-ui-reference#inventory-tab)

## Verification

1. Compare a known positive pilot and a known negative pilot against the actual
   local state. For example, fictional `LAB-WS01` has `Example Viewer 2.0` while
   `LAB-WS02` does not. Confirm the intended class and version discriminate them
   after fresh inventory; an absent row on a stale client is **unknown**, not proof
   of absence.
2. Test the read-only report/query and review the returned device list before
   connecting it to a deployment collection. Use [collection targeting](collection-targeting.md)
   for limiting scope, evaluation, exclusions, and acceptance.
3. Verify collection duration/load and data scope remain acceptable to the site
   and data owners. Record any stale/failed clients and their effect on the result.
   Do not convert an incomplete denominator into a compliance claim.

## Exceptions, recovery, and escalation

Stop expansion for unexpected collected data, excessive load, or wrong membership.
Restore the approved settings/deployment scope, retrieve policy, and confirm the
effective result. Disabling collection does not remove data already stored. Have
the data/site owners apply retention or incident handling; do not delete database
rows or collected-file directories manually. Escalate unsupported classes and
recurring provider failures with the smallest relevant private evidence set.

## Closure records

Retain business purpose, approved fields/patterns, settings and scope before/after,
pilot identities, timestamps for each processing stage, local comparisons,
query/collection acceptance, exceptions, and owner sign-off privately. State
whether inventory is accepted for the particular downstream use or remains partial.

## References

Primary references are linked at each task. See also
[compliance reporting](compliance-reporting.md), [client settings](client-settings.md),
and [MECM guide index](README.md).
