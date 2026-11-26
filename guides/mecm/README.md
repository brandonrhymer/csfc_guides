# Configuration Manager operating guides

Thirteen guides cover the operation of Microsoft Configuration Manager (also called MECM). Twelve apply to the core operating lifecycle. The certificate guide applies when the site uses PKI. These are **procedure drafts**, source reviewed on **2026-11-26**; target-system validation and operational approval are still required.

## Start at your current stage

| Your task | Read next | Result to establish locally |
| --- | --- | --- |
| Plan or document the service | [Site architecture](site-architecture.md) | An agreed role layout, communication map, capacity plan and as-built record |
| Grant an operator access | [Administrative access](administrative-access.md) | Allowed tasks work and prohibited scope remains inaccessible |
| Select a controlled set of devices | [Collection targeting](collection-targeting.md) | Membership and attached policy match the approved target list |
| Control client behavior | [Client settings](client-settings.md) | The pilot receives the intended effective settings |
| Repair a management failure | [Client health and recovery](client-health-recovery.md) | The failed function recovers and reports current evidence |
| Deliver or remove content | [Content operations](content-operations.md) | The correct content revision is available on the intended DPs |
| Run a production patch cycle | [Production patch cycle](production-patch-cycle.md) | Approved waves complete, with exceptions and workload checks recorded |
| Deploy or retire an application | [Application lifecycle](application-lifecycle.md) | Detection, install, upgrade and removal behave as intended |
| Add, upgrade, reassign or retire clients | [Client lifecycle](client-lifecycle.md) | Each endpoint reaches its recorded management state |
| Report results and exceptions | [Compliance reporting](compliance-reporting.md) | The report accounts for the expected fleet, stale data and exclusions |
| Maintain or troubleshoot the site | [Site operations](site-operations.md) | Health trends, maintenance results and incident ownership are recorded |
| Collect and troubleshoot inventory | [Inventory operations](inventory-operations.md) | Required inventory arrives with understood scope and freshness |
| Renew or troubleshoot PKI certificates | [Certificate lifecycle](certificate-lifecycle.md) | The affected roles and clients work with the intended certificate chain |

For a new service, establish architecture and access before expanding collections or deployments. For an existing service, inspect and record its current state; do not reinstall it to follow a guide. Every procedure needs the installed build, approved local values, assigned operators, maintenance impact, evidence destination and recovery criteria in its private operational copy.

## Setup and implementation handoffs

The [MECM toolkit](https://github.com/brandonrhymer/csfc_scripts/blob/main/mecm/README.md) retains installation guidance and documentation for its scripts. Use these existing procedures where applicable:

| Need | Existing procedure |
| --- | --- |
| Prerequisites, installation and first pilot | [Toolkit stages](https://github.com/brandonrhymer/csfc_scripts/blob/main/mecm/README.md#work-through-these-stages) |
| AD publishing versus discovery | [System Management container and publishing](https://github.com/brandonrhymer/csfc_scripts/blob/main/mecm/docs/ad-publishing.md), then [client onboarding](https://github.com/brandonrhymer/csfc_scripts/blob/main/mecm/docs/client-onboarding.md) |
| Initial HTTPS or Enhanced HTTP configuration | [Communication security](https://github.com/brandonrhymer/csfc_scripts/blob/main/mecm/docs/communication-security.md) |
| Initial SUP and disconnected update transfer | [WSUS/SUP setup](https://github.com/brandonrhymer/csfc_scripts/blob/main/mecm/docs/wsus-sup-setup.md) and [offline acquisition](https://github.com/brandonrhymer/csfc_scripts/blob/main/mecm/docs/offline-content-acquisition.md) |
| Backup, recovery and site upgrades | [Backup/recovery](https://github.com/brandonrhymer/csfc_scripts/blob/main/mecm/docs/backup-recovery.md) and [offline site servicing](https://github.com/brandonrhymer/csfc_scripts/blob/main/mecm/docs/offline-site-servicing.md) |
| Optional maintenance scripts and Run Scripts approval | [Client maintenance](https://github.com/brandonrhymer/csfc_scripts/blob/main/mecm/client-maintenance/README.md) |

In the disconnected toolkit scenario, the connected WSUS source also serves another network. Its owner controls its configuration. These guides do not authorize changing that shared source to satisfy a local deployment. Missing content requires an agreed acquisition route.

## Boundaries and acceptance

This set focuses on an on-premises Windows service using Configuration Manager current branch. Confirm support for the exact site, console, client, SQL, Windows and WSUS versions before use. A source review date is not a recommendation to retain a particular build. Product support and console labels can change.

OS deployment/task sequences, CMG, co-management, tenant attachment, third-party update publishing, Endpoint Protection and multi-site hierarchy expansion are separate workstreams. If your service uses one, add its design and operational runbooks before relying on this set for that workflow. PKI issuance policy belongs to the PKI owner; the certificate guide covers its Configuration Manager handoff.

Keep site identities, completed worksheets, approvals, logs, screenshots, client exports and secrets outside this public repository. The guides specify checks to perform; they do not assert that those checks have passed. See [review requirements](../REVIEW.md) for the remaining environment acceptance requirements.

[All technical guides](../README.md)
