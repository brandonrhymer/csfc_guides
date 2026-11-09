# Review an LDAPS certificate request

**Status:** Incomplete source. PKI and directory-service approval, followed by lab validation, required before deployment.

## Scope

The source proposes creating a self-signed certificate on a domain controller using elevated PowerShell. It does not describe a complete certificate request, issuance, trust, service-selection, or LDAPS-validation process.

The original command has an unclosed quoted string. The example below repairs that syntax and removes the source-specific label. It is a review example, not a verified production configuration.

## Prerequisites

- Confirm the intended relying service and domain controller names.
- Have the PKI owner determine whether to use an approved certificate authority or an explicitly permitted self-signed certificate.
- Confirm the controller's Active Directory fully qualified domain name (FQDN) and Server Authentication enhanced key usage (OID `1.3.6.1.5.5.7.3.1`), key algorithm and length, private-key properties, validity period, and renewal process.
- Confirm the required certificate store (Local Computer Personal or the NTDS service store) and directory-service selection behavior for the supported Windows Server release.
- Define client trust, deployment, validation, and rollback. Do not assume a friendly name selects a certificate for LDAPS.

## Source example for review

Open an approved elevated PowerShell session on the authorized domain controller only after the certificate design is approved.

```powershell
$fqdn = [System.Net.Dns]::GetHostByName($env:COMPUTERNAME).HostName

# Set these from the approved certificate design before running the command.
$validityYears = <APPROVED_VALIDITY_YEARS>
$friendlyName = '<APPROVED_CERTIFICATE_LABEL>'

New-SelfSignedCertificate `
    -DnsName $fqdn `
    -CertStoreLocation 'Cert:\LocalMachine\My' `
    -NotAfter (Get-Date).AddYears($validityYears) `
    -FriendlyName $friendlyName
```

`<APPROVED_VALIDITY_YEARS>` is a placeholder, not valid PowerShell input. Replace it only in an approved operational copy. The source's fixed five-year duration is not endorsed by this draft.

Verify that name resolution returns the intended controller name. The example does not configure every certificate requirement or establish client trust. The PKI owner must replace or extend it as appropriate for the approved design.

## Validate before service use

The approved deployment procedure must verify:

- Certificate names and intended server identity.
- Validity dates and appropriate enhanced key usage.
- Private-key presence, protection, and access.
- Certificate chain and trust on each intended client.
- Correct certificate selection by the directory service.
- Successful LDAPS connection and authentication under the organization's test procedure.
- Renewal, rollback, and removal of an incorrect certificate without disrupting other services.

After approved deployment, test from an intended client with `ldp.exe`: select **Connection > Connect**, enter the approved controller FQDN, use port **636**, and select **SSL**. Use **3269** only when testing the global catalog service. Successful RootDSE output checks the LDAPS connection, not application authorization. Test the required bind and application operation separately. Confirm which certificate the service presents and whether the intended clients trust it.

Do not weaken certificate validation to make the connection succeed. Do not export a private key or distribute certificate material without the required authorization.

## Reference for technical review

[Microsoft guidance on LDAP signing and certificates](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/configure-ldap-signing-certificates) describes certificate requirements and deployment considerations. Validate server identity, Server Authentication usage, a usable private key, and client trust. A syntax-correct certificate-creation command does not by itself satisfy these requirements.

[Return to guide index](../README.md)
