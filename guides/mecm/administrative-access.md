# Grant, verify and retire MECM administrative access

**Status:** Draft; source reviewed 2026-11-26; target-system validation required.

## Service context

This guide controls console access to an offline Configuration Manager current
branch standalone primary site. Record the installed site and console builds
before use. It covers ordinary onboarding, changed duties, periodic review and
access removal. It does not grant Windows, SQL, AD or service-account privileges.

Configuration Manager combines a **security role** (permitted actions),
**security scope** (securable objects) and **collection** (users/devices).
Some objects, including boundaries and default client settings, cannot be
restricted to individual instances by a security scope. Review the
[RBAC model and built-in roles](https://learn.microsoft.com/en-us/intune/configmgr/core/understand/fundamentals-of-role-based-administration)
before promising that an administrator can affect only one site area.

## Trigger and entry criteria

Start with an approved task list, named access owner, expiry/review date and
intended resource population. The approving owner must identify allowed tasks
and forbidden tasks. "Needs MECM access" is not a sufficient request.

For urgent suspected misuse, follow the incident process and directory-owner
containment instructions. Do not wait for this routine onboarding workflow.

## Prerequisites

Use the approved console workstation. The implementing account must have
authority to manage administrative users and the assigned objects; the
**Security administrator** role is the relevant built-in starting point for
access administration. Its own scope limits what it can assign. Keep an
independent, authorized administrator available before changing your own access.

Obtain these private inputs:

- `<domain>\<security-group>`, its directory owner, approved members and group nesting.
- Task-to-permission matrix, selected role, object scopes and collection IDs.
- Named positive and negative test objects, test identity and expected outcomes.
- Existing direct and group-derived access, review date and rollback record.
- Directory change authorization if group membership must change.

Prefer a task-specific domain security group with controlled membership. Have
the directory administrator create or modify it through the local AD procedure.
Verify its identity and members before assigning it. Do not create a group from
an ambiguous display name or assume that an email distribution list is suitable.

Infrastructure identities have separate lifecycles. Use
[SQL and installation accounts](https://github.com/brandonrhymer/csfc_scripts/blob/main/mecm/docs/sql-and-accounts.md)
for site-server and setup rights. Console access does not justify making an
operator a domain administrator, SQL sysadmin or local server administrator.

## Procedure

### 1. Select and test the task permissions

Use the following as candidate roles, not a claim that every permission they
contain is required. Inspect **Administration > Security > Security Roles >
selected role > Properties > Permissions** on the installed build. Record each
required object/action and reject unrelated write permissions.

| Required task | Candidate to inspect | Evidence required before assignment |
| --- | --- | --- |
| Read operational objects | Read-only analyst | Intended objects visible; editing unavailable |
| Author applications | Application author | Authoring allowed; deployment authority separately justified |
| Deploy approved applications | Application deployment manager | Required application read and target collection deployment rights |
| Manage patch deployments | Software update manager | Update tasks and collections match the request |
| Administer access | Security administrator | Security duties expressly approved |
| Notify clients to retrieve policy | Custom role | Collection **Notify Resource**, plus necessary read access |

Microsoft identifies **Notify Resource** as the permission for
[client notification actions](https://learn.microsoft.com/en-us/intune/configmgr/core/clients/manage/client-notification).
It is broader than one policy-download command. If that breadth is unacceptable,
use an authorized operator for the action instead of silently adding the grant.

1. Build a row for each requested task: object type, action, scope, collection,
   test case and owner. Include a negative case for each material boundary.
2. If a built-in role fits, record its additional permissions and the owner's
   decision. If it is too broad, use **Security Roles > source role > Copy**.
   Name the custom role, expand **Permissions**, and retain only the approved
   grants. Do not edit a shared custom role without checking all its assignees.
3. Have the security owner test the complete role in a bounded test scope. The
   [RBA Viewer](https://learn.microsoft.com/en-us/intune/configmgr/core/support/rbaviewer)
   can model permissions and audit a user, including group-derived grants. It
   requires site-server execution and broad audit visibility; have an existing
   authorized auditor use it rather than elevating the applicant for testing.
4. Treat multiple assignments as a combined effective permission set. In
   particular, deployment permission can interact with object read rights from
   another role. Review the [RBAC configuration caution](https://learn.microsoft.com/en-us/intune/configmgr/core/servers/deploy/configure/configure-role-based-administration)
   before considering role-to-scope associations an isolation boundary.

### 2. Prepare objects and add the group

Synthetic example: `EXAMPLE\CM-Pilot-Readers` may inspect the `Pilot Objects`
scope and `Pilot Devices` collection. It may not edit those objects, inspect
an unrelated production application, or operate on production devices. These
names are examples; the local matrix must identify actual objects.

1. In **Administration > Security > Security Scopes**, create the approved
   scope if needed. On each intended securable object, use **Set Security Scopes**
   to assign it. Inspect existing scope memberships before removing any.
2. Confirm the target collection and its limiting relationships using
   [collection targeting](collection-targeting.md). A scope on an application
   does not constrain the device collection by itself.
3. Open **Administration > Security > Administrative Users > Add User or Group**.
   Use **Browse** to resolve the approved domain security group. Add the selected
   security role.
4. Choose the option limiting access to specified scopes and collections. Remove
   unintended defaults, including broad collections, and add the approved scope
   and collection. Check the complete list, then save.
5. Reopen **Properties**. Inspect **Security Roles** and **Security Scopes** and
   compare them with the matrix. For approved role-specific associations, select
   **Associate assigned security roles with specific security scopes and
   collections** here and verify every association before saving.

These console actions follow Microsoft's [RBAC configuration procedure](https://learn.microsoft.com/en-us/intune/configmgr/core/servers/deploy/configure/configure-role-based-administration).
Expected result: one identifiable assignment with only the intended roles,
scopes and collection relationships. Stop if the requested object is unavailable
to the implementing administrator; do not substitute **All** to continue.

### 3. Verify allowed and denied access

1. Close the test user's existing console and start a new session under that
   identity. After group-membership changes, have the directory owner confirm
   replication and refresh the user's sign-in session as required.
2. Open the positive test collection and object. Confirm identity, membership
   visibility and expected permitted action. For a reader, inspect properties
   without saving. For a writer, perform only the approved reversible test on
   a disposable test object and restore it afterward.
3. Attempt to locate the negative test object. Confirm it is unavailable as
   designed. Verify forbidden actions are absent or denied. Do not submit an
   actual production deployment as a negative test.
4. Test a child collection limited by the assigned collection. Collection
   permissions extend to limited descendants; the directly assigned collection
   itself has special Modify/Delete restrictions. Record both observations.
5. Reconcile the result with direct user assignments and every relevant group.
   A successful positive test with an unexpectedly visible negative object is
   a failure. Capture minimal evidence privately and stop rollout.

If reporting is installed and authorized, use **Monitoring > Reporting >
Reports > Administrative Security** reports to support the review. Microsoft's
[report catalog](https://learn.microsoft.com/en-us/intune/configmgr/core/servers/manage/list-of-reports)
describes **Administrative users security assignments** and **Administration
activity log**. Reporting is supplementary; unavailable reports do not excuse
skipping console and effective-access checks.

### 4. Change or remove access

1. For a duty change, capture the previous matrix. Open the selected administrative
   user's **Properties** and revise **Security Roles** and **Security Scopes**.
   Remove obsolete associations as well as adding new ones. Repeat all access
   tests; changing a shared group assignment affects every member.
2. For one departing member of a continuing group, have the directory owner
   remove that membership. Check for direct assignments and other access-bearing
   groups. Do not delete a shared MECM group assignment to remove one person.
3. For a retired administrative-user/group assignment, select the exact entry in
   **Administrative Users**, select **Delete**, and inspect the confirmation
   before accepting it. If that action is unavailable on the installed console,
   stop and have the security owner reconcile permissions/build behavior. The
   supported [Remove-CMAdministrativeUser](https://learn.microsoft.com/en-us/powershell/module/configurationmanager/remove-cmadministrativeuser?view=sccm-ps)
   operation removes the MECM assignment; it does not delete the AD identity.
4. Test a fresh session after removal. Confirm the former access is absent,
   surviving authorized administrators still work, and no alternate assignment
   recreates the grant. Keep shared scopes/roles until their remaining consumers
   have been reviewed separately.

## Verification and expected results

Acceptance requires an approved task matrix, exact group identity, reviewed
effective memberships, successful allowed tests and successful denied tests.
For removal, demonstrate loss of the retired access in a new session. Record
any persistent session or propagation limitation and its follow-up owner.

Keep script authoring, approval and execution distinct. Microsoft separates
**SMS Scripts Create**, **SMS Scripts Approve**, and **Collection Run Script**
permissions in [Run Scripts guidance](https://learn.microsoft.com/en-us/intune/configmgr/apps/deploy-use/create-deploy-scripts).
Use the existing [Run Scripts approval walkthrough](https://github.com/brandonrhymer/csfc_scripts/blob/main/mecm/client-maintenance/run-scripts.md)
before granting those duties. A generic access test must not run a script.

## Exceptions, recovery and escalation

On unexpected access, pause the assignment and notify the security owner.
Revert only the recorded assignment or membership change through its owner;
do not overwrite concurrent access changes. If a required task fails, preserve
the object, action and error and investigate the missing permission. Do not
grant Full administrator as a diagnostic shortcut. For lockout, use the
independent authorized administrator and retain the incident record.

## Closure and records

Retain request, task matrix, group owner, before/after assignments, tester,
allowed/denied results, change time, expiry and next review privately. Identify
who will remove temporary access. Do not publish user lists or access exports.

## References

Microsoft references appear beside each applicable task. See
[site architecture](site-architecture.md) for host dependencies and
[client settings](client-settings.md) for a scoped policy-change workflow.

[MECM guide index](README.md)
