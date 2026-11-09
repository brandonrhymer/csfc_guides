# Import and link a Group Policy Object

**Status:** Technical review draft. Lab validation required. Incorrect scope can apply policy to unintended systems.

## Scope

Import an approved policy backup into a new Group Policy Object, then link it to an approved organizational unit. The source refers to DISA STIG policy packages.

## Prerequisites

- Obtain the correct package from the approved publisher or distribution process. Confirm product, version, and package integrity.
- Identify the applicable requirements and affected systems through the approved assessment process.
- Confirm that the package includes the correct GPO backup/import format. A STIG document or an arbitrary folder is not necessarily an importable GPO backup.
- Review the settings, conflicts, inheritance, security filtering, test scope, and rollback plan.
- Use an authorized administration account and approved management workstation or server.
- Back up any policy that may be affected.

## 1 Stage the policy backup

1. Copy the approved package to the authorized staging location.
2. Extract it if required.
3. Identify the specific policy backup to import. Do not assume the outer package folder is the correct import target.

## 2 Create and import the GPO

1. Open Group Policy Management.
2. Navigate to the approved domain and **Group Policy Objects**.
3. Create a new GPO using the approved naming convention.
4. Right-click that GPO and select **Import Settings** if available for the installed management tools.
5. Follow the wizard, selecting the correct backup location and policy backup.
6. Review any migration-table or reference-mapping requirements before completing the import.
7. Confirm the import summary reports success. Import copies policy settings, not the source GPO's links or security filtering. Inspect the imported settings. Do not link the GPO until the technical owner approves the scope.

## 3 Link to the approved scope

1. Confirm the target organizational unit and affected computer or user objects.
2. Review inheritance, link order, enforcement, security filtering, and any WMI filter under the approved design.
3. Right-click the approved target and select **Link an Existing GPO**.
4. Select the newly imported GPO and confirm the intended link.

Do not infer the target from a generic folder name or apply the GPO broadly merely because its import succeeded.

## Validate

The source ends after linking. Before operational use, define how to:

- Verify the GPO link, scope, and settings.
- Confirm application to the intended test systems after policy refresh.
- Use Group Policy Results for the intended test user and computer to check resultant policy and application behavior. Verify the imported GPO is applied rather than denied by filtering. Keep reports in the approved environment.
- Confirm that unintended systems were not affected.
- Roll back the link or settings if acceptance criteria fail. Removing a link does not guarantee that every previously applied setting is undone. Test the setting-specific rollback and policy refresh before production use.

Record the package version, approved scope, validation outcome, and exceptions in the authorized change record. Keep actual domain and organizational-unit identifiers out of this collection.

## Reference for technical review

[Microsoft guidance on GPO backup and import](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/group-policy/group-policy-backup-restore) describes backup selection, import results, and migration tables.

[Return to guide index](../README.md)
