# Pilot, verify and revert MECM client settings

**Status:** Draft; source reviewed 2026-11-26; target-system validation required.

## Service context

Use this procedure to change client settings in an offline Configuration Manager
current branch standalone primary site. Record the installed site, console and
client builds. Settings control client behavior such as inventory, policy,
Software Center and update management. They are distinct from a deployment,
collection maintenance window, configuration baseline or domain Group Policy.

Default client settings apply throughout the hierarchy. Custom device settings
target device collections; custom user settings target user collections and
offer the settings applicable to users. Custom settings override defaults. When
applicable custom values conflict, the lower order number has higher priority.
Use [Microsoft's configuration procedure](https://learn.microsoft.com/en-us/intune/configmgr/core/clients/deploy/configure-client-settings)
and verify the resultant settings; do not assume a single object supplies the
client's entire configuration.

## Trigger and entry criteria

Enter for an approved behavior change or a diagnosed settings conflict. Define
the exact setting, old and intended values, affected users/devices, business
effect, observation period and rollback outcome. Start with a reviewed pilot
collection. A default-settings edit needs a separate assessment of every client
affected by that hierarchy-wide change.

## Prerequisites

Use an approved console workstation and an account with the necessary Client
Settings read/create/modify/deploy rights and access to the target collection.
Verify the actual permissions using [administrative access](administrative-access.md).
Client notification additionally requires **Notify Resource** on collections.
Obtain approved access to the pilot for local observation and log reading;
console permission does not grant that host access.

Record privately:

- Target collection ID and verified roster, plus a negative-control device outside that target.
- Current default/custom objects, order numbers and their deployed collections.
- Every value in the settings group to be selected, with intended differences.
- User discovery/policy prerequisites for user settings and the test sign-in identity.
- Existing GPO/other management authorities, workload owner and conflict resolution.
- Restore values, observation deadline, client access and evidence storage location.

Use [collection targeting](collection-targeting.md) before deployment. Select a
healthy, communicating pilot; use [client health and recovery](client-health-recovery.md)
if policy receipt is already failing.

## Procedure

### 1. Establish the effective baseline

1. Open **Administration > Client Settings**. Record **Default Client Settings**
   and all custom objects potentially applying to the pilot. Inspect each
   object's **Properties** and each custom object's **Deployments** tab. Defaults
   apply without a collection deployment. Include overlapping collections;
   their names do not prove their actual membership.
2. Open **Assets and Compliance > Devices**, select the pilot and choose
   **Client Settings > Resultant Client Settings**. For a user-settings change,
   use **Users** and the intended user. Record the current effective values.
3. Inspect the same settings for the negative control. Resolve any unexpected
   baseline before continuing. Resultant Client Settings is a read-only console
   calculation; it does not prove that an offline endpoint has received policy.
4. Check Microsoft's [setting-specific definitions](https://learn.microsoft.com/en-us/intune/configmgr/core/clients/deploy/about-client-settings)
   for applicability, dependencies and combination behavior. For example,
   software inventory file types can merge across applicable settings. The
   lower-number rule is not a reason to skip resultant-value inspection.

### 2. Work through an observable conflict example

The following is a synthetic lab exercise with no production deployments. It
changes Software Center branding so the result is observable without installing
software. The order numbers and names are illustrative, not required site values.

| Object | Target | Order | Software Center company name |
| --- | --- | --- | --- |
| Existing default | All clients | Default | `Example Service` |
| `Lab Branding A` | Reviewed lab devices | 2 | `Example Lab A` |
| `Lab Branding B` | One-device pilot, also in the lab collection | 1 | `Example Lab B` |

Expected: the pilot displays `Example Lab B`; a lab control outside the pilot
displays `Example Lab A`; a device outside both retains its prior default.
Removing B's deployment makes A the expected pilot result after policy processing.
Do not edit real default branding just to reproduce this table.

Before performing the exercise, inspect the whole Software Center settings group
and preserve approved existing choices. Adding a group can introduce other
values, including visibility/customization options; a one-field intention is
not evidence that only one field will change. If the full group cannot be made
safe for the test, stop and select a locally approved alternative.

### 3. Create and deploy the pilot settings

1. In **Administration > Client Settings**, choose **Create Custom Client
   Settings**, then **Create Custom Client Device Settings** or **Create Custom
   Client User Settings** for the request. Enter a unique name and description
   identifying owner, purpose and change record.
2. Select only the required settings groups. Enter reviewed values in each group
   and compare every field with the approved worksheet. For the branding example,
   select **Software Center**, set **Select these new settings to specify company
   information** to **Yes**, open **Customize**, and set **General > Company
   name**. Preserve the other reviewed customization choices, then save.
3. Inspect the new object's order. Use **Move Item Up/Move Item Down** if required
   by the approved precedence plan. Reordering affects every target of that
   object; record any other objects whose relative precedence changes.
4. Select the object, choose **Deploy**, and select the exact reviewed collection.
   Inspect the object's **Deployments** tab immediately. Expected result: only the
   intended assignment exists. Stop and withdraw an incorrect assignment before
   requesting any client action.
5. Repeat Resultant Client Settings for pilot and controls. Compare all selected
   groups, not just the requested field. Do not proceed if a higher-priority
   object prevents the intended change or an unrelated effective value changes.

### 4. Retrieve policy and observe the endpoint

1. Record the request time. For one selected pilot device, use **Assets and
   Compliance > Devices > Client Notification > Download Computer Policy**.
   For the user-policy branch, use **Download User Policy** with the intended
   test user signed in and user policy enabled. These are distinct
   [notification actions](https://learn.microsoft.com/en-us/intune/configmgr/core/clients/manage/client-notification).
2. Alternatively, on the authorized pilot open **Control Panel > Configuration
   Manager > Actions**. Select **Machine Policy Retrieval & Evaluation Cycle >
   Run Now**; use **User Policy Retrieval & Evaluation Cycle** for the user
   branch. These are Microsoft's [local policy retrieval steps](https://learn.microsoft.com/en-us/intune/configmgr/core/clients/manage/manage-clients#start-policy-retrieval).
   Choose one method; do not repeatedly trigger both because the UI has not changed.
3. Inspect `%windir%\CCM\Logs\PolicyAgent.log` for retrieval and
   `PolicyEvaluator.log` for evaluation around the recorded time. If location or
   connectivity is suspect, inspect `LocationServices.log`. Interpret entries
   using the [log reference](https://learn.microsoft.com/en-us/intune/configmgr/core/plan-design/hierarchy/log-files);
   an accepted notification is not evidence of endpoint application.
4. Verify the setting's actual behavior. For branding, close and reopen Software
   Center and inspect the displayed company name. For an inventory change, verify
   the intended inventory data arrives. For update settings, use the
   [patch-cycle checks](production-patch-cycle.md). Record time and outcome.
5. Repeat the observation on the negative control. Wait through the locally
   approved period for the relevant behavior. Record offline/unreachable devices
   separately instead of marking them passed from console configuration alone.

### 5. Resolve conflicts, revert or expand

1. If resultant values are wrong, compare collection membership, assignment type,
   priority and every overlapping settings object. Correct the reviewed cause;
   raising priority globally can change other clients.
2. If resultant values are correct but endpoint behavior is wrong, inspect policy
   processing, build applicability and other management authorities. Configuration
   Manager policy retrieval does not refresh domain GPO. For software updates,
   domain GPO can override equivalent local policy used by the client and cause
   scan failures. Have the GPO and patch owners reconcile it using
   [software update settings guidance](https://learn.microsoft.com/en-us/intune/configmgr/sum/get-started/manage-settings-for-software-updates).
   Do not remove a GPO or rewrite policy registry values as an ad hoc fix.
3. To withdraw the pilot override, select its custom settings object and its
   **Deployments** tab. Select only the pilot assignment and choose **Delete**;
   inspect the confirmation before accepting it. Keep the settings
   object for investigation until remaining consumers are known. If the console
   action is unavailable, stop for the site owner rather than deleting the object.
   Microsoft's [deployment-removal operation](https://learn.microsoft.com/en-us/powershell/module/configurationmanager/remove-cmclientsettingdeployment?view=sccm-ps)
   also distinguishes removing one assignment from deleting the settings object.
4. Recalculate resultant values, retrieve policy once and observe the restored
   behavior. In the example, B's withdrawal must restore A. If a previously
   existing object was edited, restore its recorded values/order after checking
   for concurrent changes. Removing policy is not a universal undo for every
   setting's side effects; verify the setting-specific recovery outcome.
5. Expand only after pilot acceptance. Recheck the next collection's membership,
   overlaps and workload impact, then deploy in the approved waves. Do not move
   the pilot value into defaults solely to simplify administration.

## Verification and expected results

Acceptance needs all three layers: the intended assignment and order, correct
resultant values, and observed client behavior. Controls must retain their
expected behavior. Demonstrate reversal before broader rollout. Record unresolved
offline clients with a follow-up owner and planned check; do not treat them as
successful because a settings object exists.

## Exceptions, recovery and escalation

Stop expansion for unexpected targets, altered unrelated settings, policy errors,
GPO conflicts or an unproven restore path. Preserve timestamps, selected groups,
old/new values and minimal relevant logs. Escalate scope to the collection owner,
policy delivery to the client/site owner, and domain policy to the GPO owner.
Do not disable security policy, reset WMI, reinstall every client or introduce an
Internet path to make a pilot pass.

## Closure and records

Store before/after settings, priorities, assignments, pilot/control identities,
policy/log observations, behavioral results and rollback evidence privately.
Record owner acceptance, remaining clients, next review date and the cleanup
decision for temporary objects. Keep change-specific screenshots out of public Git.

## References

Microsoft references are linked beside the relevant operations. See
[inventory operations](inventory-operations.md) for inventory outcomes and
[compliance reporting](compliance-reporting.md) for update and application
deployment results. Configuration-baseline assessment is a separate workflow.

[MECM guide index](README.md)
