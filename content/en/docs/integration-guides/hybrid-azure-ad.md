---
title: "Hybrid Azure Active Directory"
description: ""
lead: "This section will provide instructions on how to enable passwordless authentication using the Fast Identity Online 2 (FIDO2) standard in hybrid Azure AD environments with the help of IDmelon Passwordless Orchestration Platform (IPOP). This step-by-step guide provided in this document will help organizations configure and enable FIDO2 authentication on thei Hybrid Azure AD setup easily."
date: 2023-04-01T18:14:02-08:00
lastmod: 2023-04-01T18:14:02-08:00
draft: false
images: []
menu:
  docs:
    parent: "directory-services-integration"
weight: 100
toc: true
---

<div id="_modal" class="modal">
  <span class="close">&times;</span>
  <img class="modal-content" id="img01">
</div>

<div align="center">
    <img src="/images/vendor/DirectoryServicesIntegration/HybridAzureAD/hybridDiagram.png" class="doc-img-frame">
</div>

## Requirements

- Windows Server 2016 ([patch KB4534307](https://support.microsoft.com/en-us/topic/january-23-2020-kb4534307-os-build-14393-3474-b181594e-2c6a-14ea-e75b-678efea9d27e)), 2019 ([patch KB4534321](https://support.microsoft.com/en-us/topic/january-23-2020-kb4534321-os-build-17763-1012-023e84c3-f9aa-3b55-8aff-d512911c459f)), and 2022
- Windows 10 or 11 clients (version 2004 or later)
- Azure AD Connect (version 1.4.32.0 or later)
- FIDO2 Security Keys

## Azure AD Connect

### Prerequisites

[Prerequisites for Azure AD Connect](https://learn.microsoft.com/en-us/azure/active-directory/hybrid/how-to-connect-install-prerequisites#before-you-install-azure-ad-connect)

### Configuration

1. Sign in as _Local Administrator_ on the server you want to install _Azure AD Connect_ on the server you sign in on will be the sync server.
2. Go to `AzureADConnect.msi` and double-click to open the installation file.
3. In welcome, select the checkbox to agree to the licensing terms, and then select _Continue_.
4. In _Express settings_, select _Use express settings_.

    <div align="center">
        <img src="/images/vendor/DirectoryServicesIntegration/HybridAzureAD/express.png" class="doc-img-frame">
    </div>

5. In `Connect to Azure AD`, enter the _username_ and _password_ of the Hybrid Identity Administrator account, and then select Next.

    <div align="center">
        <img src="/images/vendor/DirectoryServicesIntegration/HybridAzureAD/connectaad.png" class="doc-img-frame">
    </div>

6. In Connect to AD DS, enter the username and password for an Enterprise Admin account. You can enter the domain part in either NetBIOS or FQDN format, like **IDMELON\administrator** or **idmelon.com\administrator**. Then, elect Next.

    <div align="center">
        <img src="/images/vendor/DirectoryServicesIntegration/HybridAzureAD/connectad.png" class="doc-img-frame">
    </div>

7. The [Azure AD sign-in configuration](https://learn.microsoft.com/en-us/azure/active-directory/hybrid/plan-connect-user-signin#azure-ad-sign-in-configuration) page appears only if you didn't complete the step to [verify your domains](https://learn.microsoft.com/en-us/azure/active-directory/fundamentals/add-custom-domain) in the [prerequisites](https://learn.microsoft.com/en-us/azure/active-directory/hybrid/how-to-connect-install-prerequisites).

    <div align="center">
        <img src="/images/vendor/DirectoryServicesIntegration/HybridAzureAD/unverifieddomain.png" class="doc-img-frame">
    </div>

    If you see this page, review each domain that is marked Not Added or Not Verified. Make sure that those domains have been verified in Azure AD. When you have verified your domains, select the Refresh icon.

    In Ready to configure, select Install.

    - Optionally in **Ready to configure**, you can clear the **Start the synchronization process as soon as configuration completed** checkbox. You should clear this checkbox if you want to do more configuration, such as to add [filtering](https://learn.microsoft.com/en-us/azure/active-directory/hybrid/how-to-connect-sync-configure-filtering). If you clear this option, the wizard configures syncs but leaves the scheduler disabled. The scheduler doesn't run until you enable it manually by [rerunning the installation wizard](https://learn.microsoft.com/en-us/azure/active-directory/hybrid/how-to-connect-installation-wizard).
    - If you leave the **Start the synchronization process when configuration completes** checkbox selected, a full sync of all users, groups, and contacts to Azure AD begins immediately.
    - If you have Exchange in your instance of Windows Server Active Directory, you also have the option to enable [Exchange Hybrid deployment](https://learn.microsoft.com/en-us/exchange/exchange-hybrid). Enable this option if you plan to have Exchange mailboxes both in the cloud and on-premises at the same time.

    <div align="center">
        <img src="/images/vendor/DirectoryServicesIntegration/HybridAzureAD/readytoconfigure.png" class="doc-img-frame">
    </div>

8. When the installation completed, select Exit.
9. Before you use Synchronization Service Manager or Synchronization Rule Editor, sign out, and then sign in again.

After these steps, you can now check the Azure portal to ensure that the local users are available on the Azure active directory users list.

<div align="center">
    <img src="/images/vendor/DirectoryServicesIntegration/HybridAzureAD/azureusers.png" class="doc-img-frame">
</div>

## Enable FIDO2 security key method

1. Sign in to the [Azure portal](https://portal.azure.com/)
2. Browse to **Azure Active Directory > Security > Authentication methods > Authentication method policy**.
3. Under the method **FIDO2 Security Key**, click **All users**, or click **Add groups** to select specific groups. _Only security groups are supported._
4. Save the configuration.

<div align="center">
    <img src="/images/vendor/DirectoryServicesIntegration/HybridAzureAD/authenticationmethods.png" class="doc-img-frame">
</div>

<div align="center">
    <img src="/images/vendor/DirectoryServicesIntegration/HybridAzureAD/securitykeyconfig.png" class="doc-img-frame">
</div>

## Enable passwordless security key sign-in to on-premises resources by using Azure AD

To enable the **security key sign-in option** on clients you need to install the Azure AD Kerberos module.
The [Azure AD Kerberos PowerShell module](https://www.powershellgallery.com/packages/AzureADHybridAuthenticationManagement) provides FIDO2 management features for administrators.

### Install the Azure AD Kerberos PowerShell module

1. Open a PowerShell prompt using the Run as administrator option.
2. Install the Azure AD Kerberos PowerShell module:

```powershell
# First, ensure TLS 1.2 for PowerShell gallery access.
[Net.ServicePointManager]::SecurityProtocol = [Net.ServicePointManager]::SecurityProtocol -bor [Net.SecurityProtocolType]::Tls12

# Install the Azure AD Kerberos PowerShell Module.
Install-Module -Name AzureADHybridAuthenticationManagement -AllowClobber
```

### Create a Kerberos Server object

Administrators use the Azure AD Kerberos PowerShell module to create an Azure AD Kerberos Server object in their on-premises directory.

Run the following steps in each domain and forest in your organization that contain Azure AD users:

1. Open a PowerShell prompt using the Run as administrator option.
2. Run the following PowerShell commands to create a new Azure AD Kerberos Server object both in your on-premises Active Directory domain and in your Azure Active Directory tenant.

_Replace admin@idmelon.com in the following example with your on-premises Active Directory domain name._

```powershell
# Specify the on-premises Active Directory domain. A new Azure AD
# Kerberos Server object will be created in this Active Directory domain.
$domain = $env:USERDNSDOMAIN

# Enter a UPN of an Azure Active Directory global administrator
$userPrincipalName = "admin@idmelon.com"

# Enter a domain administrator username and password.
$domainCred = Get-Credential

# Create the new Azure AD Kerberos Server object in Active Directory
# and then publish it to Azure Active Directory.
# Open an interactive sign-in prompt with given username to access the Azure AD.
Set-AzureADKerberosServer -Domain $domain -UserPrincipalName $userPrincipalName -DomainCredential $domainCred
```

**Note:** _If you're working on a domain-joined machine with an account that has domain administrator privileges, you can skip the "-DomainCredential" parameter. If the "-DomainCredential" parameter isn't provided, the current Windows login credential is used to access your on-premises Active Directory Domain Controller._

```powershell
# Specify the on-premises Active Directory domain. A new Azure AD
# Kerberos Server object will be created in this Active Directory domain.
$domain = $env:USERDNSDOMAIN

# Enter a UPN of an Azure Active Directory global administrator
$userPrincipalName = "admin@idmelon.com"

# Create the new Azure AD Kerberos Server object in Active Directory
# and then publish it to Azure Active Directory.
# Open an interactive sign-in prompt with given username to access the Azure AD.
Set-AzureADKerberosServer -Domain $domain -UserPrincipalName $userPrincipalName
```

### View and verify the Azure AD Kerberos Server

You can view and verify the newly created Azure AD Kerberos Server by using the following command:

```powershell
Get-AzureADKerberosServer -Domain $domain -CloudCredential $cloudCred -DomainCredential $domainCred
```

This command outputs the properties of the Azure AD Kerberos Server. You can review the properties to verify that everything is in good order.

### Remove the Azure AD Kerberos Server

If you want to revert the scenario and remove the Azure AD Kerberos Server from both the on-premises Active Directory and Azure AD, run the following command:

```powershell
Remove-AzureADKerberosServer -Domain $domain -CloudCredential $cloudCred -DomainCredential $domainCred
```

**Warning:** _If your password has expired, signing in with FIDO is blocked. The expectation is that users reset their passwords before they can log in by using FIDO._

### What can I do if I'm unable to use the FIDO security key immediately after I create a hybrid Azure AD-joined machine?

If you're clean-installing a hybrid Azure AD-joined machine, after the domain join and restart process, you must sign in with a password and wait for the policy to sync before you can use the FIDO security key to sign in.

- Check your current status by running **dsregcmd /status** in a Command Prompt window, and check to ensure that both the **AzureAdJoined** and **DomainJoined** statuses are showing as **YES**.
- This delay in syncing is a known limitation of domain-joined devices and isn't FIDO-specific.

```shell
+---------------------------------------------------+
| Device State                                      |
+---------------------------------------------------+
             AzureAdJoined : YES
          EnterpriseJoined : NO
              DomainJoined : YES
                DomainName : IDMELON
+---------------------------------------------------+
```

## Enabling login permission with security key in Active Directory

To enable FIDO security key sign-in. The setting can be found under **Computer Configuration > Administrative Templates > System > Logon > Turn on security key sign-in**

- Setting this policy to **Enabled** allows users to sign in with security keys.
- Setting this policy to **Disabled** or **Not Configured** stops users from signing in with security keys.

<div align="center">
    <img src="/images/vendor/DirectoryServicesIntegration/HybridAzureAD/enableSecuritykeyLogin.png" class="doc-img-frame">
</div>

## Register a Security Key

1. Go to the My Profile page at [My Account](https://myaccount.microsoft.com/) and sign in if you haven't already done so.
2. Select **Security Info**, select **Add method**, and then select **Security key** from the **Add a method** list.
3. Select **Add**, and then select the type of security key you have, either **USB** device or **NFC** device.

## Log in with the Security Key

If you have followed all the above steps correctly, now the **sign in option** should appear on the lock screen of the client system and you can enter by selecting the **FIDO security key** option.

<div align="center">
    <img src="/images/vendor/DirectoryServicesIntegration/HybridAzureAD/signinOptions.png" class="doc-img-frame">
</div>

## Registered Devices

Refer to **Users > [select user] > Devices** to make sure that the client PCs are registered in the Azure portal.

<div align="center">
    <img src="/images/vendor/DirectoryServicesIntegration/HybridAzureAD/userdevices.png" class="doc-img-frame">
</div>

## References

- [Azure AD Connect: Accounts and permissions](https://learn.microsoft.com/en-us/azure/active-directory/hybrid/reference-connect-accounts-permissions)
- [Enable passwordless security key sign-in to on-premises resources by using Azure AD](https://learn.microsoft.com/en-us/azure/active-directory/authentication/howto-authentication-passwordless-security-key-on-premises)
- [Enable passwordless security key sign-in
  ](https://learn.microsoft.com/en-us/azure/active-directory/authentication/howto-authentication-passwordless-security-key)
- [Prerequisites for Azure AD Connect
  ](https://learn.microsoft.com/en-us/azure/active-directory/hybrid/how-to-connect-install-prerequisites)
- [Deployment frequently asked questions (FAQs) for hybrid FIDO2 security keys in Azure AD](https://learn.microsoft.com/en-us/azure/active-directory/authentication/howto-authentication-passwordless-faqs)
- [Get started with Azure AD Connect by using express settings](https://learn.microsoft.com/en-us/azure/active-directory/hybrid/how-to-connect-install-express)
