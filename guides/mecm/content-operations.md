# Operate and repair MECM content distribution

**Status:** Draft. Source review: 2026-11-26. Target-system validation required.

[MECM guide index](README.md)

## Service context

Use this guide for application and software-update content in a disconnected,
standalone Configuration Manager current-branch primary site. Confirm the
installed build is supported and the referenced controls exist before use.
Content distribution makes files available; it does not authorize installation.
Use [application lifecycle](application-lifecycle.md) or the
[production patch cycle](production-patch-cycle.md) for deployment decisions.

Keep these locations distinct:

| Location | Purpose and ownership |
| --- | --- |
| Transfer/staging folder | Holds accepted inbound files and their private manifest until ingestion is verified; it is not the permanent package source |
| Permanent source | Protected, retained source files at the object's configured UNC path; the source owner controls changes and recovery |
| Site content library | Configuration Manager's managed, deduplicated content store |
| DP content library | Managed files from which clients obtain distributed content; a DP colocated with the site uses its shared library |
| Client cache | Temporary installation content on each endpoint; it is not a source backup |

Never add, edit, or delete files inside `SCCMContentLib` to repair a package.
Use supported validation and redistribution. Retain source files independently
of the library. [Microsoft: content library](https://learn.microsoft.com/en-us/intune/configmgr/core/plan-design/hierarchy/the-content-library)
Client-cache administration has its own supported controls; Explorer deletion
does not reconcile the client's cache records.
[Microsoft: configure client cache](https://learn.microsoft.com/en-us/intune/configmgr/core/clients/manage/configure-client-cache)

## Trigger and entry conditions

Start for approved new content, a source revision, a failed transfer, a reported
hash failure, or retirement from a particular DP. Stop if the object, source,
destination, or recovery owner is unknown. First confirm that the symptom is
content-related; wrong collection membership or missing policy belongs in
[collection targeting](collection-targeting.md) or
[client health and recovery](client-health-recovery.md).

## Prerequisites and private inputs

Operate from the approved console host with RBAC rights to read/modify the
specific content objects and distribute to the approved DPs. The site owner
must verify security scopes and effective permissions; console visibility alone
does not grant source-share or server-log access. Obtain separate read access
to server/client logs and approved endpoint access for the download test.

Record privately:

- Change ID, operator, source owner, DP owner, observation window and escalation contact.
- Site/build, object type/name/ID, source version or application content ID,
  permanent source path, intended DPs and relevant boundary groups.
- Approved file manifest, source provenance, transfer acceptance, current
  distribution state, available storage and competing transfers.
- Referencing deployments, dependencies and retention/recovery requirements;
  the approved pilot and its expected internal content source.

For software-update ingestion, both the downloading operator and SMS Provider
computer require the documented write permissions at the package destination.
Use a separate source directory for each update deployment package; restrict
other write access. [Microsoft: download updates](https://learn.microsoft.com/en-us/intune/configmgr/sum/deploy-use/download-software-updates)
For application sources, confirm the site server can read the retained source.
Do not broaden permissions as a generic response to access denied.

## Procedure tasks

### 1. Establish content identity

1. Open **Software Library > Application Management > Applications** for an
   application, or **Software Library > Software Updates > Deployment Packages**
   for update content. Select the exact approved object, not a similarly named
   predecessor. Record its source and existing destinations.
2. For an application, inspect the intended deployment type, content path and
   dependencies. For updates, reconcile the package against the approved update
   IDs, revisions, languages and required files. If input is missing, use the
   [offline acquisition procedure](https://github.com/brandonrhymer/csfc_scripts/blob/main/mecm/docs/offline-content-acquisition.md).
3. Compare accepted source files with the recorded manifest. Record every
   intentional difference before publishing changed content. An unexplained
   difference is a stop condition, even if the DP currently reports success.

**Expected result:** one unambiguous object/version and an intact, recoverable
source. A filename or KB title alone is insufficient identity.

### 2. Choose and perform one supported action

Use the action matching the problem; do not cycle through all actions.

| Need | Console action |
| --- | --- |
| First delivery to a destination | Select the object, **Distribute Content**; review dependencies, choose the approved DP, and review the summary |
| Publish approved changed source | Packages: **Update Distribution Points**. Applications: select the deployment type, then **Update Content** |
| Repair/resend existing content | **Administration > Distribution Points > Properties > Content**; select the object, **Redistribute** |
| Check integrity | In the same DP **Content** tab, select the object, **Validate** |
| Retire from this DP | In that tab, select the object, **Remove**, only after the retirement checks below |

Updating rereads the source and propagates changes to the object's existing
distribution locations; review all affected destinations and deployments before
that action. It is not a repair limited to one selected DP.

Updating package content changes its version. Application package version stays
at 1; updating a deployment type creates a new content ID. Redistribution resends
existing content; validation checks integrity and reports failures. These actions
are not interchangeable. [Microsoft: manage distributed content](https://learn.microsoft.com/en-us/intune/configmgr/core/servers/deploy/configure/deploy-and-manage-content)

Before confirming any action, compare the displayed object and destination with
the private record. Cancel on a mismatch. Record the action and start time.
For retirement, reconcile all referencing deployments and dependencies, confirm
an approved alternative source where still required, and preserve recovery
material. Removing a DP copy does not uninstall endpoints or retire an application.

### 3. Verify distribution, then prove a client download

1. Open **Monitoring > Distribution Status > Content Status**. Select the exact
   package and **View Status**. Inspect the intended DP under Success, In Progress,
   Error or Unknown; capture its detailed messages and timestamp. Confirm the
   displayed source version matches the intended change. Monitor until the
   agreed observation limit, not until an arbitrary universal timeout.
   [Microsoft: monitor content](https://learn.microsoft.com/en-us/intune/configmgr/core/servers/deploy/configure/monitor-content-you-have-distributed)
2. For an application, additionally correlate the deployment type/content ID;
   package version 1 cannot distinguish its revisions. After repair, request
   validation for the affected object and review the new result.
3. On the approved pilot, use the approved deployment to request the content.
   Record `LocationServices.log`, `CAS.log`, `ContentTransferManager.log` and
   `DataTransferService.log` in `%windir%\CCM\Logs`. Correlate content identity,
   selected internal source and completed transfer. If files were already
   cached, record a cache hit; it does not prove a new DP download. Select a
   suitable uncached pilot/content test with the endpoint owner instead of
   deleting the whole cache. [Microsoft: log reference](https://learn.microsoft.com/en-us/intune/configmgr/core/plan-design/hierarchy/log-files)
4. If peer caching is enabled, distinguish the actual peer from the DP. Verify
   the approved source path required by the test. A successful install without
   a recorded source does not establish the DP-to-client route.

### 4. Conditional prestaging

Use prestaging only when the DP owner has chosen a documented bandwidth/media
workflow. Record file/version identity, media approval, extraction outcome and
status acknowledgment. Follow Microsoft's topology-specific prestage procedure;
do not simply copy files into the library. Site-server DPs and pull DPs have
different restrictions. A package revision requires matching prestaged content.
[Microsoft: prestaged content](https://learn.microsoft.com/en-us/intune/configmgr/core/servers/deploy/configure/deploy-and-manage-content#use-prestaged-content)

## Verification and acceptance

Accept distribution/repair only when the requested object/version is present on
each intended DP, relevant validation succeeds, and the pilot demonstrates the
required internal download. Accept removal when the DP assignment is absent and
remaining required deployments still have a proven source. These are required
observations; this draft establishes none of them.

## Exceptions, recovery and escalation

- **Source missing or access denied:** preserve the error and route to the
  source/site owner. Restore the approved source/permissions before retrying.
- **Distribution stalls or fails:** correlate package ID/time with site
  `distmgr.log` and `PkgXferMgr.log`; use the recorded role-host logs for DP
  installation/health. Check destination storage and connectivity with its owner.
- **DP hash failure:** preserve evidence, confirm trusted source identity, then
  redistribute that object and revalidate. Repeated corruption requires storage
  or security investigation, not repeated blind redistribution.
- **DP healthy, one client fails:** compare boundary/source selection, endpoint
  space, trust and cache state with a working peer; use client recovery guidance.
- **Unapproved source revision:** stop expansion. The application/update owner
  decides whether to republish retained content or create a corrected version;
  content reversal does not undo completed installations.

For suspected orphaned DP storage, use the existing
[cleanup readiness procedure](https://github.com/brandonrhymer/csfc_scripts/blob/main/docs/maintenance/distribution-point.md).
This guide does not authorize library deletion or a new cleanup job.

## Closure records

Retain before/after identity, action/destinations, source-manifest reference,
status/log timestamps, pilot source and transfer result, exception owner and
recovery decision in the approved private record. Keep operational names,
manifests and logs outside this public repository.

## References

Microsoft references are linked beside the applicable steps. Also use the
[end-to-end patch checkpoints](https://github.com/brandonrhymer/csfc_scripts/blob/main/mecm/docs/verification-checkpoints.md)
and [site architecture](site-architecture.md) for role and boundary context.
