---
title: ADFS support in MSAL.NET 
description: Learn about Active Directory Federation Services (ADFS) support in the Microsoft Authentication Library for .NET (MSAL.NET).
services: active-directory
author: Dickson-Mwendia
manager: CelesteDG

ms.service: msal
ms.subservice: msal-dotnet
ms.topic: conceptual
ms.workload: identity
ms.date: 08/24/2023
ms.author: dmwendia
ms.reviewer: ddelimarsky
ms.custom: devx-track-csharp, aaddev, devx-track-dotnet
#Customer intent: As an application developer, I want to learn about AD FS support in MSAL.NET so I can decide if this platform meets my application development needs and requirements.
---

# Active Directory Federation Services support in MSAL.NET

Active Directory Federation Services (AD FS) in Windows Server enables you to add OpenID Connect and OAuth 2.0 based authentication and authorization to applications you are developing. Those applications can, then, authenticate users directly against AD FS. For more information, read [AD FS Scenarios for Developers](/windows-server/identity/ad-fs/overview/ad-fs-openid-connect-oauth-flows-scenarios).

Microsoft Authentication Library for .NET (MSAL.NET) supports two scenarios for authenticating against AD FS:

- MSAL.NET talks to Microsoft Entra ID, which itself is *federated* with AD FS.
- MSAL.NET talks **directly** to an ADFS authority. This is only supported from AD FS 2019 and above. One of the scenarios this highlights is [Azure Stack](https://azure.microsoft.com/overview/azure-stack/) support

<a name='cases-where-identity-providers-are-federated-with-azure-ad'></a>

## Cases where identity providers are federated with Microsoft Entra ID

MSAL.NET supports talking to Microsoft Entra ID, which itself signs-in managed users (users managed in Microsoft Entra ID), or federated users (users managed by another identity provider, which, in the case we are interested is federated through ADFS). MSAL.NET does not know about the fact that users are federated. As far as it’s concerned, it talks to Microsoft Entra ID.

The [authority](/azure/active-directory/develop/msal-client-applications) you'll use in the case is the usual authority (common, organizations, or tenanted).

### Acquiring a token interactively

When you call the the `AcquireTokenInteractive`,the user experience is typically: 

- The user enter their upn (or the account or `loginHint` is provided part of the call to <xref:Microsoft.Identity.Client.IPublicClientApplication.AcquireTokenAsync(System.Collections.Generic.IEnumerable{System.String})>)
- Microsoft Entra ID displays briefly "Taking you to your organization's page"
- Microsoft Entra ID then redirects the user to the sign-in page of the identity provider (usually customized with the logo of the organization)

Supported ADFS versions in this federated scenario are ADFS v2 , ADFS v3 (Windows Server 2012 R2) and ADFS v4 (ADFS 2016)

### Acquiring a token using `AcquireTokenByIntegratedAuthentication` or `AcquireTokenByUsernamePassword`

When acquiring a token using the `AcquireTokenByIntegratedAuthentication` or `AcquireTokenByUsernamePassword` methods, MSAL.NET gets the identity provider to contact based on the username.  MSAL.NET receives a [SAML 1.1 token](/azure/active-directory/develop/reference-saml-tokens) after contacting the identity provider.  MSAL.NET then provides the SAML token to Microsoft Entra ID as a user assertion (similar to the [on-behalf-of flow](../web-apps-apis/on-behalf-of-flow.md) to get back a JWT.


Microsoft recommends you **do not use** the username and password flow. In most scenarios, more secure alternatives are available and recommended. This flow requires a very high degree of trust in the application, and carries risks that are not present in other flows. You should only use this flow when other more secure flows aren't viable. For more information about why you want to avoid using this grant, see [why Microsoft is working to make passwords a thing of the past](https://news.microsoft.com/features/whats-solution-growing-problem-passwords-says-microsoft/).


## Case where MSAL connects directly to ADFS

MSAL.NET supports connecting to AD FS 2019, which is OpenID Connect compliant and understands PKCE and scopes. This support requires that a service pack [KB 4490481](https://support.microsoft.com/help/4490481/windows-10-update-kb4490481) is applied to Windows Server. When connecting directly to AD FS, the authority you'll want to use to build your application is similar to `https://mysite.contoso.com/adfs/`.

Currently, there are no plans to support a direct connection to:

- AD FS 16, as it doesn't support PKCE and still uses resources, not scope
- AD FS v2, which is not OIDC-compliant.

However for MSAL.NET we have no plans to support a direct connection to ADFS 2016 (it does not support PKCE and still uses resources, not scope). When you have upgraded your on-premise system to ADFS 2019, you'll be able to use MSAL.NET.

MSAL does not support Integrated Windows Authentication (by calling `AcquireTokenByIntegratedWindowsAuth`) directly to ADFS.

## See also

- [Troubleshooting IWA/ADFS Setup](/windows-server/identity/ad-fs/troubleshooting/ad-fs-tshoot-iwa)
- For the federated case, see [Configure Microsoft Entra sign-in behavior for an application by using a Home Realm Discovery policy](/azure/active-directory/manage-apps/configure-authentication-for-federated-users-portal)
