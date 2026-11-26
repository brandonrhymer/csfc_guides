# Maintain Configuration Manager PKI certificates

**Status:** Draft; source reviewed 2026-11-26; target-system validation required.

## Service context and scope

Use this runbook when an existing Configuration Manager current branch deployment uses PKI certificates and a certificate approaches renewal, its issuing chain changes, or authentication fails. The Configuration Manager administrator owns the service checks. The PKI owner approves issuance, renewal and revocation. The Windows/IIS owner applies approved bindings. Assign named owners and escalation contacts in a private operational copy.

This is a certificate maintenance procedure, not a CA installation or HTTPS migration procedure. Record the installed Configuration Manager and Windows versions. Confirm their support and the role-specific certificate requirements before use. For initial configuration, use the [communication security guide](https://github.com/brandonrhymer/csfc_scripts/blob/main/mecm/docs/communication-security.md).

Enhanced HTTP uses Configuration Manager-generated certificates. Identify that mode before selecting a renewal procedure. Do not replace its generated certificates with a PKI renewal procedure. A site can contain both PKI HTTPS roles and Enhanced HTTP roles. Inspect each role separately. See [Microsoft's Enhanced HTTP guidance](https://learn.microsoft.com/en-us/intune/configmgr/core/plan-design/hierarchy/enhanced-http).

## Trigger and entry criteria

Start a planned renewal early enough to complete issuance, disconnected transfer, pilot verification and recovery before expiry. The PKI owner supplies that lead time; there is no universal renewal interval in this guide. For suspected key compromise or a revoked certificate, enter the security incident process immediately. Do not restore a compromised certificate as a recovery measure.

Before proceeding, have:

- A change record identifying the exact role, host, certificate purpose and maintenance window.
- Authorized local administrator access to the affected Windows server for its certificate store and IIS binding; separate Configuration Manager permissions to modify the affected role when needed.
- A working out-of-band access method if the client management channel fails.
- The approved CA template or issuance method, names, EKUs, key provider and protection requirements.
- A protected copy of the existing binding and role configuration, and an approved recovery certificate if recovery is feasible.
- A pilot client in each affected trust and network path, plus expected policy, content or scan results.

Stop if the certificate owner, affected services, approved replacement, or recovery route is unknown. A certificate store can serve several products; this runbook does not authorize changing unrelated bindings.

## Procedure

### 1. Record the certificate dependencies

1. On the affected server, open `certlm.msc`. Inspect **Personal > Certificates**. Open the candidate certificate and record its thumbprint, subject/SAN, issuer, validity and EKUs in the private change record. Confirm that its private key is present where required. A matching friendly name alone is insufficient.
2. In IIS Manager, select the actual website used by the role and open **Bindings**. Record the HTTPS IP, port, hostname and selected certificate. Inspect any other bindings that use the same certificate before changing it.
3. From the Configuration Manager console, inspect **Administration > Site Configuration > Servers and Site System Roles**. Record the role's connection settings. For a distribution point, also inspect **Administration > Distribution Points > Properties > Communication**.
4. Complete the purpose table below for every affected certificate. A server certificate and a client authentication certificate are separate dependencies even when they are on the same host.

| Purpose | What to inspect | Maintenance destination |
| --- | --- | --- |
| IIS server identity for HTTPS MP, DP or SUP | Server Authentication EKU; names used by callers; local computer Personal store | Correct website's HTTPS binding |
| Windows client identity | Client Authentication EKU; computer identity; accessible private key; applicable selection criteria | Client computer certificate store and enrollment process |
| MP site-system monitoring identity | Required client authentication certificate, including when no Configuration Manager client is installed | MP computer Personal store |
| DP authentication to HTTPS MP | Client authentication certificate exported as protected PKCS #12/PFX | DP Communication properties import |
| Imported trusted root | Approved root certificate without its private key | Windows trust stores and applicable site certificate issuer configuration |

Use the [Microsoft PKI requirements](https://learn.microsoft.com/en-us/intune/configmgr/core/plan-design/network/pki-certificate-requirements) for the exact role's supported key characteristics and identity requirements. Do not apply one generic certificate template to every row. Task-sequence media certificates, CMG certificates and SQL certificates require their own role-specific maintenance plans if present.

### 2. Prepare and inspect the replacement

1. Have the PKI owner issue or renew the certificate using the approved process. For an enterprise enrollment workflow, the authorized server operator opens **Personal > Certificates > All Tasks > Request New Certificate** in `certlm.msc`, selects the approved template, supplies approved identity fields, and completes enrollment. If the template is missing or denied, stop and resolve enrollment rights with the PKI owner; do not substitute another template.
2. Reopen the resulting certificate. Compare its thumbprint, identity, purpose and expiry with the change record. Confirm the intended private key exists and the service can use it. Investigate unexpected duplicate eligible client certificates before rollout.
3. Use the PKI owner's approved validation procedure to check the chain and revocation endpoints from each affected network. Before testing, record its procedure/version, validation host, execution identity and expected endpoints. Retain the certificate thumbprint, chain result, CRL publication/next-update times, retrieval result and test time. In a disconnected environment, confirm the approved CRL publication/transfer process supplies current data before it expires. Test the actual service context: a successful browser request by an administrator does not establish LOCAL SYSTEM access. Stop if no approved service-context test is available; request one from the PKI owner.
4. If a root CA certificate previously imported into Configuration Manager was renewed, import the renewed root through the approved site configuration procedure. Recheck the client's eligible issuer and selection settings. Do not export a root private key for this task.

Microsoft documents [revocation, trust and client selection behavior](https://learn.microsoft.com/en-us/intune/configmgr/core/plan-design/security/plan-for-certificates). An inaccessible required CRL can prevent certificate-dependent connections. Resolve access or publication failures with the PKI and network owners; disabling checks is not a recovery step in this runbook.

### 3. Apply only the relevant role change

**IIS server certificate renewal**

1. On the server, open IIS Manager and select the recorded website, then **Bindings**.
2. Select the existing HTTPS binding and choose **Edit**. Select the replacement certificate. Use **View** to check its identity before saving. Preserve the recorded address, port and hostname unless a separate approved change covers them.
3. Save the binding. Repeat for another binding only if it is explicitly in scope. Capture the resulting binding and certificate identity in the private record.

**Distribution point client authentication certificate**

1. Obtain the approved replacement client authentication PFX using the protected transfer process. Keep its password out of tickets, transcripts and Git.
2. In **Administration > Distribution Points**, select the correct DP and open **Properties > Communication**. Select **Import certificate**, choose the replacement PFX, and provide its password through the UI.
3. Save and verify DP status reporting to the HTTPS management point. Updating the DP's IIS server binding does not update this separate certificate. Follow [Microsoft's DP communication settings](https://learn.microsoft.com/en-us/intune/configmgr/core/servers/deploy/configure/install-and-configure-distribution-points#communication).

**Windows client renewal**

1. Use the approved client certificate enrollment or provisioning method on the pilot device. Inspect the resulting certificate in the computer store.
2. Check **Control Panel > Configuration Manager > General** for the client certificate state. Read `ClientIDManagerStartup.log` and `CcmMessaging.log` under the client log directory, normally `%windir%\CCM\Logs`, for selection and communication errors.
3. Request machine policy through the client control panel's **Actions** tab. Verify policy retrieval and the relevant workload. If selection is ambiguous, refer the observed certificates and configured criteria to the PKI and site owners before expanding deployment.

**SUP/WSUS renewal**

1. Before changing the binding, inspect the effective client setting **Enforce TLS certificate pinning for Windows Update client for detecting updates** and clients' local-computer `WindowsServerUpdateServices` stores. If pinning applies, have the certificate provisioning owner distribute and verify the approved replacement **public certificate, without its private key**, in that store on affected clients. Record coverage and unresolved devices before scheduling cutover. Keep the previous valid pin for the approved rollback period. Do not disable pinning to make renewal pass. See [Microsoft's WSUS certificate-pinning requirements](https://learn.microsoft.com/en-us/intune/configmgr/sum/get-started/software-update-point-ssl#tls-certificate-pinning-for-devices-scanning-https-configured-wsus-servers).
2. Replace the certificate on the existing **WSUS Administration** HTTPS binding as above. Preserve HTTP bindings and the established virtual-directory configuration.
3. Check the SUP's existing SSL setting and connect the WSUS console over its configured SSL port from the site server. Verify `WCM.log` on the site server and `WSUSCtrl.log` on the SUP, then confirm a fresh pilot update scan. Include a pilot with the effective pinning configuration when applicable; verify the replacement pin and scan evidence together.
4. If the hostname, port or SSL mode is changing, stop the renewal-only procedure and use the complete [Microsoft SUP TLS configuration tutorial](https://learn.microsoft.com/en-us/intune/configmgr/sum/get-started/software-update-point-ssl). That workflow coordinates IIS, WSUS and the SUP. Do not require SSL on the whole WSUS website or treat a certificate replacement as authority to change the site's synchronization settings.

## Verification and recovery

| Check after the change | Expected evidence | If it fails |
| --- | --- | --- |
| Certificate and endpoint identity | Recorded new thumbprint and correct binding; valid chain and required revocation data | Stop rollout; inspect wrong binding, name, trust, key access or CRL path |
| MP communication | Fresh policy retrieval on pilot and successful current MP health checks in `mpcontrol.log` | Route to site/PKI owner with timestamps and certificate identity |
| DP service | Current successful DP status and pilot content download | Separate DP client-certificate status failures from IIS download failures |
| SUP service | Successful WSUS connection and fresh pilot scan in `WUAHandler.log` | Check WSUS binding, SUP SSL settings, trust and connectivity |
| Client selection | Eligible new certificate and successful authenticated workload | Check duplicate eligible certificates and site selection criteria |

Use the [Microsoft log reference](https://learn.microsoft.com/en-us/intune/configmgr/core/plan-design/hierarchy/log-files) to confirm log placement for the installed role. Correlate events after the recorded change time; an old success is not acceptance.

If a planned renewal breaks service, pause rollout and use the approved recovery path: restore the recorded binding to the previous certificate only if it is still valid, trusted and uncompromised. A DP PFX import has a separate recovery import. Recheck the same workload after recovery. If no usable previous certificate exists, escalate for replacement issuance and service restoration. Do not remove the role or relax authentication as an improvised fix.

For compromise, the security/PKI owners control revocation, key replacement and publication. Verify affected peers obtain the updated revocation information. Preserve relevant evidence outside this repository.

## Closure and records

Close only after the affected role and pilot paths pass, the owner accepts the observation period, and the certificate inventory shows the new expiry and next review date. Record failures, recovery actions and any untested network segments. Retire old certificates and transfer copies only through the PKI retention/disposal process after checking dependencies. Store private evidence and secrets in their approved systems.

Operational acceptance still requires a dated renewal rehearsal on the actual certificate architecture, including unavailable-CRL and recovery scenarios where approved. Source review does not establish those results.

[MECM guide index](README.md)
