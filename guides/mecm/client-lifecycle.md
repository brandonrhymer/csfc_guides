# Operate the Windows client lifecycle

Status: **Draft — target-system validation required.** Sources reviewed: **2026-11-26**.

## Service context

This runbook covers batch onboarding, client upgrades, site reassignment, identity
exceptions, and retirement for Configuration Manager current-branch Windows
clients on a disconnected internal network. Record exact site/client builds and
Windows support before use. Cloud enrollment and operating-system deployment are
outside scope. These procedures have not been exercised against the target site.

Start with the existing [one-client onboarding procedure](https://github.com/brandonrhymer/csfc_scripts/blob/main/mecm/docs/client-onboarding.md).
This runbook adds lifecycle control around it; it does not repeat boundaries,
discovery, certificates, or setup commands.

## Trigger and entry criteria

Use one branch for an approved intake batch, a site update containing a new client,
a documented site move, an identity conflict, or an asset retirement. Client repair
uses [client health recovery](client-health-recovery.md). Do not combine an upgrade
and reassignment merely because they are technically possible; record and validate
each intended change.

## Prerequisites and private inputs

- Record the change/asset references, owners, approved device manifest, identities,
  current and intended site/version, distribution points (DPs), maintenance and
  restart windows, observation deadline, exceptions, and acceptance decision-maker.
- Use an administration workstation with the matching console. The operator needs
  RBAC permissions for the intended collections and client operation. Client push
  requires its approved account/dependencies; local installation needs administrator
  rights on each client. Confirm independent access for failed installations.
- Keep a complete approved client source with provenance, version, transfer hashes,
  and required installation properties. Revalidate the
  [offline security prerequisites](https://github.com/brandonrhymer/csfc_scripts/blob/main/mecm/docs/communication-security.md).
  Certificates, trust, revocation access, DNS, time, and role connectivity must work
  inside the disconnected network; an internet download is not a recovery plan.
- Before changing membership, inspect deployments, maintenance windows, exclusions,
  client settings, and application implicit-uninstall behavior. Use
  [collection targeting](collection-targeting.md) and [client settings](client-settings.md).
  Keep actual device lists and evidence outside public source control.

## Procedure tasks

### 1. Onboard a bounded batch

1. Reconcile the intake manifest with **Assets and Compliance > Devices**. Match
   computer/domain and hardware identity with the asset owner. Record each resource
   ID and whether the device is discovered, client-installed, and site-assigned.
   Stop duplicate or ambiguous rows before selecting targets.
2. Create or verify the approved pilot collection and its limiting collection.
   Evaluate membership and compare every member to the manifest. A discovery record
   alone is not evidence that a client is installed or managed.
3. Choose one installation route in the change record:

   | Route | Use when | Required control |
   | --- | --- | --- |
   | Local manual installation | Small batch, workgroup device, or remote push unavailable | Complete staged source and local administrator access; follow the linked onboarding procedure per device. |
   | Selected-device/collection client push | Trust, firewall, administrative share, authentication, and account prerequisites have been verified | Target the reviewed pilot; do not enable automatic site-wide push as a side effect. |
   | Existing approved installation distribution | A separately maintained deployment process already exists | Verify its actual source/properties and scope; do not invent a new logon script or GPO during intake. |

4. For selected client push, open **Assets and Compliance > Devices** or the reviewed
   collection, select the intended records, and choose **Install Client**. Review
   the wizard's target, installation options, and properties against the change.
   Finish only when that scope matches. Observe site-server `CCM.log` and each
   client's `%windir%\ccmsetup\Logs\ccmsetup.log`; push can retry after initial
   failure, so a failed first attempt is not cancellation.
   [Microsoft: client deployment routes and push](https://learn.microsoft.com/en-us/intune/configmgr/core/clients/deploy/deploy-clients-to-windows-computers)
5. For each device, complete onboarding verification: final setup result, intended
   site/MP, policy receipt, health evaluation, inventory receipt, and the agreed
   workload test. Record failures separately. Expand to the next approved batch
   only after the owner reviews the pilot and confirms membership again.

Expected result: the manifest accounts for every device as accepted, excluded with
reason, or failed with an owner. A collection-level percentage does not close
missing or duplicate devices.

### 2. Test, promote, and roll out a new client

1. Before site servicing, record production/pre-production versions and the test
   collection membership. Use representative supported devices with independent
   access. Workgroup clients are not supported by pre-production deployment; give
   them a separate approved test/rollout plan after production promotion.
2. At **Administration > Site Configuration > Sites > Hierarchy Settings > Client
   Upgrade**, configure the approved pre-production collection and automatic
   pre-production upgrade. During the separately approved
   [offline site update](https://github.com/brandonrhymer/csfc_scripts/blob/main/mecm/docs/offline-site-servicing.md),
   select **Test in pre-production collection** on **Client Options**.
3. Inspect **Monitoring > Client Status > Pre-production Client Deployment**.
   Verify the installed version locally, policy, health, inventory, Software Center,
   and an approved application/update workflow. Capture failures and restart effects.
   Site-system hosts can report **Not compliant** in pre-production despite a
   successful client upgrade; verify locally before classifying that exception.
   [Microsoft: deployment-status monitoring](https://learn.microsoft.com/en-us/intune/configmgr/core/clients/deploy/monitor-client-deployment-status)
4. **Before promotion**, inspect **Hierarchy Settings > Client Upgrade** and record
   the existing production automatic-upgrade setting, server handling, interval
   and exclusion collection. Review client push and other installation mechanisms
   with their owners. An already-enabled production upgrade policy can start
   broad rollout after promotion makes the new version production. Approve that
   scope first. If a hold or exclusion is required, apply the approved restriction
   and verify affected clients receive it before promotion; newly configured
   exclusions depend on updated client policy. Stop if rollout restrictions are
   unverified. [Microsoft: upgrade exclusions](https://learn.microsoft.com/en-us/intune/configmgr/core/clients/manage/upgrade/exclude-clients-windows)
5. After pilot acceptance and rollout-scope approval, an authorized hierarchy administrator selects **Administration >
   Updates and Servicing > Promote Pre-production Client**, compares both versions
   and the collection, then confirms **Promote**. Microsoft requires Full
   Administrator with All scope for promotion; verify the documented Update
   Packages permissions and connection to the CAS or standalone primary site.
   Promotion changes the hierarchy's production client source, not just the pilot.
   [Microsoft: pre-production testing and promotion](https://learn.microsoft.com/en-us/intune/configmgr/core/clients/manage/upgrade/test-client-upgrades)
6. Recheck **Hierarchy Settings > Client Upgrade** after promotion against the
   approved pre-promotion record. If a rollout was held, enable **Upgrade all
   clients in the hierarchy using the production client** only when its scope,
   server handling, interval and exclusions are approved. This is hierarchy-wide;
   use a reviewed installation method if the approved scope is narrower.
7. Monitor **Monitoring > Client Status > Production Client Deployment** and compare
   actual client versions with the manifest. Automatic upgrades are randomized
   within the configured interval and honor applicable maintenance windows. Account
   for offline clients and exclusions rather than marking them successful.
   [Microsoft: Windows client upgrades](https://learn.microsoft.com/en-us/intune/configmgr/core/clients/manage/upgrade/upgrade-clients-for-windows-computers)
8. If the pilot fails, hold promotion. If production rollout fails, stop expansion
   and have the site owner review the active upgrade/install mechanisms. Do not
   assume clearing a checkbox cancels work already received. Use the health guide
   per failed device; arbitrary client downgrade is not the rollback procedure.

If the topology includes a passive site server, assess Microsoft's documented
[pre-production/failover issue](https://learn.microsoft.com/en-us/intune/configmgr/core/clients/manage/upgrade/test-client-upgrades#pre-production-client-and-site-server-high-availability)
with the site owner before servicing or failover. It can change the production
client source independently of this planned promotion sequence.

Synthetic example: a reviewed batch contains `LAB-WS01`, `LAB-WS02`, and `LAB-WG01`.
The first two are domain members; the third is a workgroup client. Test the domain
clients in pre-production. Record `LAB-WG01` as outside that mechanism, with its
own post-promotion local-install test. “Two passed” is not “all three upgraded.”

### 3. Reassign or reconcile client identity

1. For a move, confirm the destination **primary** site supports the client version,
   has the intended boundaries/roles, trusts its authentication, and can deliver
   content and policy. Record the source-site assignment and memberships for recovery.
   Roaming or changing an IP address does not automatically reassign an already
   assigned client. [Microsoft: site assignment](https://learn.microsoft.com/en-us/intune/configmgr/core/clients/deploy/assign-clients-to-a-site)
2. For another primary site **within the same hierarchy**, select the verified
   device in **Assets and Compliance > Devices**, choose **Reassign Site**, and
   select the approved destination. Check local **Configuration Manager > Site**,
   `ClientLocation.log`, policy receipt, and console assignment afterward. A move
   between hierarchies needs a separate migration/installation plan; stop here.
3. For a conflicting identity, compare hardware identifiers, asset history,
   reinstall/reimage dates, and current client registration before any deletion or
   merge. Open **Monitoring > System Status > Conflicting Records**. With the
   identity decision recorded, select **Conflicting Record** and the appropriate
   **Merge**, **New**, or **Block** action. Do not change the hierarchy-wide conflict
   policy to resolve one unexplained record.
   [Microsoft: client operations and conflicting records](https://learn.microsoft.com/en-us/intune/configmgr/core/clients/manage/manage-clients)
4. For duplicate SMBIOS IDs/shared network adapters, escalate to the imaging/site
   owner to confirm the documented duplicate-hardware-identifier mechanism. Do not
   merge two physical devices merely because their names or an adapter match.

### 4. Retire a device without automatic re-enrollment

1. Confirm asset-owner authorization and retain required history before changes.
   Review all deployment paths, not just one collection. Have the directory and
   deployment owners remove/exclude the device from relevant discovery and automatic
   installation sources under their approved procedures. Avoid broad OU exclusions
   that remove unrelated managed systems.
2. Select the operation that matches the intended outcome:

   | Operation | Effect and limitation |
   | --- | --- |
   | Remove collection membership | Changes targeting after evaluation/policy; does not uninstall the MECM agent. Review other collections and any configured application implicit uninstall. |
   | Uninstall the Windows client | Removes its management agent through CCMSetup; does not itself retire the asset or remove discovery sources. |
   | Block | Stops MECM trust/communication; it is not general network isolation or software removal. Use the security owner's decision for an untrusted device. |
   | Delete device record | Removes database history/associations; a communicating client can recreate the record. Microsoft describes this as a troubleshooting action, not uninstall. |

   [Microsoft: block, delete, and uninstall](https://learn.microsoft.com/en-us/intune/configmgr/core/clients/manage/manage-clients)
3. If a clean agent removal is authorized, connect independently to the client and
   open **Command Prompt as administrator**. Run
   `%windir%\ccmsetup\ccmsetup.exe /uninstall`. Wait for completion and verify
   `%windir%\ccmsetup\Logs\ccmsetup.log`; the command has no on-screen completion
   result. Stop on an uninstall error and preserve its logs. For a security
   incident, follow containment direction before attempting routine removal.
4. Verify the client is absent locally and that no pending push/GPO/other installer
   reintroduces it. Observe through the locally agreed discovery/install cycles.
   Preserve records under retention policy; use reviewed maintenance cleanup for
   stale records. If a record returns, identify its discovery source before deleting
   it again. Handle AD removal, certificates, data sanitization, and asset disposal
   through their owning procedures.

## Verification

Reconcile every manifest entry with actual version, assignment, health, policy, and
inventory evidence. For retirement, require local uninstall evidence plus absence
of unintended rediscovery/reinstallation over the agreed observation period. For a
site move, verify both destination management and source-record disposition.

## Exceptions, recovery, and escalation

Hold expansion for identity ambiguity, unexpected members, failed PKI/source
validation, or missing management. Keep failed devices assigned to an owner with
independent access. Reverting collection settings or reinstalling an agent does not
undo applications already installed. If reassignment fails, the site owner must
validate the previous site before any reassignment back.

## Closure records

Retain approved manifests, before/after versions and assignments, source hashes,
collection and deployment IDs, promotion/rollout decisions, installer evidence,
exceptions, and asset-owner acceptance privately. Record deferred/offline devices
explicitly. Do not claim production readiness from document review alone.

## References

Primary references are linked beside their procedures. Continue with
[client health recovery](client-health-recovery.md),
[inventory operations](inventory-operations.md), or the [MECM guide index](README.md).
