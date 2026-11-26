# Build, verify and retire MECM collection targets

**Status:** Draft; source reviewed 2026-11-26; target-system validation required.

## Service context

Collections determine which users or devices can receive deployments, client
settings and other management actions. They also participate in administrative
access. This guide covers Configuration Manager current branch on an offline
standalone primary site. Record the installed site/console build and validate
available controls before use. A collection contains devices or users, not both.
See Microsoft's [collection introduction](https://learn.microsoft.com/en-us/intune/configmgr/core/clients/manage/collections/introduction-to-collections).

Use an empty, undeployed collection to develop new targeting. Changing membership
of an existing collection can affect attached work without creating a new
deployment. A membership count is insufficient evidence: compare the actual
resource identities with the approved roster.

## Trigger and entry criteria

Use this procedure for a new pilot, a rule change, controlled rollout expansion,
withdrawal or collection retirement. Enter with an approved population, a named
collection owner, the intended management purpose and an observation period.
Obtain a separate change for any deployment or client-setting changes.

## Prerequisites

Work from the approved console workstation under an account with collection
read/create/modify rights for the intended population. Deletion and deployment
rights are separate. Verify them through [administrative access](administrative-access.md).
If dependent objects are outside your visibility, their owner must complete the
audit before membership changes proceed.

Record privately: collection name/ID, device or user type, limiting collection,
resource IDs and domain identities, rule logic, data source and freshness,
evaluation schedule, exclusion owner, attached deployments/settings/windows,
before/after roster and recovery owner. Treat real membership exports as
environment data; do not commit them to this repository.

## Procedure

### 1. Calculate the intended membership

| Rule/control | Decision to make |
| --- | --- |
| Limiting collection | The maximum eligible population; direct and include rules cannot bypass it |
| Direct rule | Explicit discovered/imported resources maintained by an operator |
| Query rule | Conditions evaluated against site data; dependent on discovery/inventory freshness |
| Include collection | A reusable population whose later changes affect this target |
| Exclude collection | A removal set; exclusion wins when inclusion conflicts |

Microsoft documents these [membership rules](https://learn.microsoft.com/en-us/intune/configmgr/core/clients/manage/collections/create-collections)
and their [evaluation order](https://learn.microsoft.com/en-us/intune/configmgr/core/clients/manage/collections/collection-evaluation).
Use this reasoning aid: eligible query/direct/included members, intersected with
the limit, minus exclusions. It describes expected membership, not a script to run.

Synthetic example:

| Input | Members |
| --- | --- |
| Limit: `Approved Lab Devices` | `LAB-01`, `LAB-02`, `LAB-03`, `LAB-04` |
| Query result | `LAB-01`, `LAB-02` |
| Direct rule | `LAB-04` |
| Included collection | `LAB-03`, `LAB-05` |
| Excluded hold collection | `LAB-02` |
| Expected final target | `LAB-01`, `LAB-03`, `LAB-04` |

`LAB-05` is outside the limit; `LAB-02` is excluded. Adding `LAB-02` directly
does not override the exclusion. If the limit later admits `LAB-05`, the include
rule can add it. Record this future expansion path before approving the design.

### 2. Audit existing effects and dependencies

1. In **Assets and Compliance > Device Collections** or **User Collections**,
   locate the exact collection. Record its ID, then select **Show Members** and
   capture the current roster. Investigate duplicate names by resource identity.
2. Open **Properties > Membership Rules**. Record the limit, every rule and the
   update settings. Use **View Relationships** to inspect collections that limit,
   include or exclude this collection and those it depends on.
3. Inspect the collection's **Deployments** details and reconcile them with
   **Monitoring > Deployments**, filtering to the target collection. Have each
   application, patch, baseline or task-sequence owner identify required actions,
   deadlines, uninstall behavior and any automatic rule that can recreate work.
4. In **Administration > Client Settings**, inspect applicable custom settings'
   **Deployments** tabs for this target. Record priority and affected groups;
   follow [client settings](client-settings.md) when changing their assignments.
5. For device collections, inspect **Properties > Maintenance Windows** and
   identify the devices' other applicable windows. Record window type, time basis
   and recurrence. Applicable windows from multiple collections combine; adding
   membership can broaden permitted execution time. Removing membership can
   remove a window. Review Microsoft's [maintenance-window behavior](https://learn.microsoft.com/en-us/intune/configmgr/core/clients/manage/collections/use-maintenance-windows)
   and the actual deployment overrides; a window does not prevent content download.
6. Ask the access owner whether this collection is assigned to administrators.
   Stop if the dependency or attached-work inventory is incomplete.

For a new collection, record these checks as empty after creation and before
adding a deployment. Use [collection management](https://learn.microsoft.com/en-us/intune/configmgr/core/clients/manage/collections/manage-collections)
for the console actions and dependency view.

### 3. Create or revise the bounded target

1. Select **Create Device Collection** or **Create User Collection** from the
   appropriate node. Enter the approved name and a comment with purpose, owner
   and change reference. **Browse** to the reviewed limiting collection.
2. On **Membership Rules**, select **Add Rule**. For a direct device pilot, choose
   **Direct Rule**, search **System Resource** by **Name**, and select only the
   verified resources. Inspect domain/resource identity if names are ambiguous.
   Complete the rule wizard; expected result: the selected resources appear as
   explicit rules, subject to the limit and exclusions.
3. For a query, select **Query Rule > Edit Query Statement > Criteria**. Add the
   approved attribute/operator/value through the query editor. Record the exact
   resulting expression and independently inspect its candidate population.
   Confirm whether its attributes come from discovery or inventory. Do not add
   a broad wildcard simply because the expected device is missing.
4. For an include or exclusion, choose that rule type and select the reviewed
   source collection. Confirm its ID and owner. Compare the combined result with
   the worksheet, rather than assuming one rule replaces other rules.
5. Set the approved full-update schedule. Enable incremental evaluation only
   after the site owner confirms suitability of the query classes and dependency
   chain. It is not a substitute for full evaluation or fresh inventory.
6. Complete the wizard. For an existing collection, make the reviewed changes in
   **Properties > Membership Rules**, preserving unrelated rules and schedules.
   Record a recoverable copy of the original definition first.

### 4. Evaluate and reconcile

1. Request **Update Membership** once for the selected collection. Wait for the
   evaluation to complete; **Refresh** updates the displayed results and does not
   itself evaluate membership.
2. Inspect its evaluation completion time and **Show Members**. Use
   **Monitoring > Collection Evaluation > Collection Evaluation Queue** when
   completion is delayed. Review queue/run information using Microsoft's
   [evaluation monitoring procedure](https://learn.microsoft.com/en-us/intune/configmgr/core/clients/manage/collections/collection-evaluation-view).
3. Compare the full final roster against the approved set. Record added, removed,
   missing and unexpected identities separately. For the synthetic example,
   verify all three expected members plus the absence of `LAB-02` and `LAB-05`.
4. If a query is wrong, inspect the source attribute and its timestamp in the
   resource record/Resource Explorer. Follow [inventory operations](inventory-operations.md)
   or the existing [discovery procedure](https://github.com/brandonrhymer/csfc_scripts/blob/main/mecm/docs/client-onboarding.md).
   Re-evaluation cannot manufacture missing inventory. Check upstream limiting,
   include and exclusion collections before changing the target's rules.
5. If evaluation remains queued or fails beyond the locally agreed observation
   period, retain collection ID, timestamps and queue evidence. Have the site
   owner inspect `colleval.log`; do not repeatedly force large evaluations.

### 5. Expand, withdraw or retire

1. Before expansion, rerun the attached-work audit and compare only the proposed
   additions with the authorized wave. Existing required deployments may act
   after new members receive policy. Apply the bounded change, reevaluate and
   verify both the collection and actual client outcomes under the owning
   [patch](production-patch-cycle.md) or [application](application-lifecycle.md) guide.
2. For withdrawal, identify every rule that can admit the resource. Remove the
   relevant direct rule, revise the approved query/source collection, or use an
   approved exclusion with an owner and expiry. A direct-rule removal alone does
   not defeat an include/query rule. Verify absence after evaluation.
3. Separately verify client policy withdrawal and any already-started action.
   Removing membership is not evidence that an installation stopped or that an
   application was uninstalled. Use workload-specific recovery when needed.
4. To retire a collection, have owners remove/reassign attached work and resolve
   all limiting/include/exclude and RBAC references. Export the definition if
   required; collection export does not include every referenced collection or
   the full attached-work configuration.
5. Select **Delete** only for the approved custom collection. Inspect the exact
   collection/deployment list. Cancel if deletion expands to dependencies not
   approved for retirement. Do not select **Delete each collection member from
   the database**. Refresh and confirm only the intended collection was removed.

## Verification and expected results

Accept targeting only after a completed evaluation yields the exact approved
members and negative examples remain absent. Confirm no unintended deployment,
settings, maintenance-window or RBAC change. For active-target changes, retain
separate evidence of client effects; a correct console roster alone is not
deployment acceptance. For retirement, verify remaining collections and devices
still exist and replacement assignments work.

## Exceptions, recovery and escalation

On unexpected members, halt further rollout and notify every attached-work owner.
They must contain pending deployments while the collection owner restores the
recorded rule/limit configuration and reevaluates. Do not assume a corrected
roster reverses completed actions. Escalate data freshness to the inventory
owner, evaluation failures to the site owner, and identity ambiguity to the
device-lifecycle owner. Do not delete resource records to make a count match.

## Closure and records

Retain rule versions, limits/dependencies, before/after rosters, evaluation times,
attached-work audit, client observations, exceptions and owner acceptance
privately. Record the next review date and cleanup owner for temporary pilots
and exclusions. Publish only synthetic examples.

## References

The Microsoft references beside each task define product behavior. Related
operations: [client lifecycle](client-lifecycle.md) and
[site operations](site-operations.md).

[MECM guide index](README.md)
