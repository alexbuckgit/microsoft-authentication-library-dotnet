## Acquiring a token: this depends on the kind of application

As explained in [Scenarios](Scenarios), there are many ways of acquiring a token. They are detailed in the next topics. Some require user interactions through a web browser. Some don't require any user interactions. 
In general the way to acquire a token is different depending on if the application is a public client application (Desktop / Mobile) or a confidential client application (Web App, Web API, daemon application like a windows service)

### MSAL.NET caches tokens
For both Public client and confidential client applications, MSAL.NET maintains a token cache (or two caches in the case of confidential client applications), and applications should [try to get a token from the cache](AcquireTokenSilentAsync-using-a-cached-token) first before any other means, except in the case of [Client Credentials](https://aka.ms/msal-net-client-credentials), which does look at the application cache by itself. 

In the case of UWP, Xamarin iOS and Xamarin Android, the token cache serialization to an isolated storage is provided by MSAL.NET. In the case of .NET Desktop (.NET Framework and .NET Core) platforms, though, the application needs to customize the [token cache serialization](token-cache-serialization).

### Token acquisition methods

#### Public client application:

##### Public flows

- will often [acquire token interactively](Acquiring-tokens-interactively), having the user sign-in.
  > Remember that this is not possible yet in .NET Core as .NET core does not provide UI capabilities and this is required for interactive authentication 
- But it's also possible for a desktop application running on a Windows machine which is joined to a domain or to Azure AD, to [get a token silently for the user signed-in on the machine](https://aka.ms/msal-net-iwa), leveraging Integrated Windows Authentication (IWA/Kerberos). 
- On .NET framework desktop client applications , it's also possible (but not recommended) to [get a token with a username and password](https://aka.ms/msal-net-up) (U/P).
  > Note that you should not use username/password in confidential client applications
- Finally, for applications running on devices which don't have a Web browser, it's possible to acquire a token through the [device code flow](https://aka.ms/msal-net-device-code-flow) mechanism, which provides the user with a URL and a code. The user goes to a web browser on another device, enters the code and signs-in, which has Azure AD get them a token back on the browser-less device.

##### Summary for public client applications

The following table summarizes the ways available to acquire tokens in public client applications depending on the Operating system, and therefore the chosen platform, and the kind of application.

Operating System | Library Platform | Kind of App | [Interactive Auth](Acquiring-tokens-interactively) | [IWA](https://aka.ms/msal-net-iwa) | [U/P](https://aka.ms/msal-net-up) | [Device Code](https://aka.ms/msal-net-device-code-flow)
-- | -------- | --- | ----------  | --- | --- | ----------------- 
<img src="https://docs.microsoft.com/en-us/azure/active-directory/develop/media/index/logo_windows.svg" width="40" /> <br/> Windows desktop	| <img src="https://docs.microsoft.com/en-us/azure/active-directory/develop/media/index/logo_net.svg" width="40" /> <br/> .NET| Desktop (WPF,<br/>  Windows.Forms,<br/>  Console) | Y | Y | Y | (Y)
<img src="https://docs.microsoft.com/en-us/azure/active-directory/develop/media/index/logo_windows.svg" width="40" /> <br/>Windows 10	| <img src="https://docs.microsoft.com/en-us/azure/active-directory/develop/media/index/logo_windows.svg" width="40" /> <br/> UWP	| Store app	| Y	| Y	| 	
<img src="https://docs.microsoft.com/en-us/azure/active-directory/develop/media/index/logo_android.svg" width="40" /> <br/> Android	| <img src="https://docs.microsoft.com/en-us/azure/active-directory/develop/media/index/logo_xamarin.svg" width="40" /> <img src="https://docs.microsoft.com/en-us/azure/active-directory/develop/media/index/logo_android.svg" width="40" /> <br/> Xamarin Android	| Mobile	| Y	| 	| Y	
<img src="https://docs.microsoft.com/en-us/azure/active-directory/develop/media/index/logo_ios.svg" width="40" />| <img src="https://docs.microsoft.com/en-us/azure/active-directory/develop/media/index/logo_xamarin.svg" width="40" /> <img src="https://docs.microsoft.com/en-us/azure/active-directory/develop/media/index/logo_ios.svg" width="40" /><br/> Xamarin iOS	| Mobile	| Y	| 	| 	| Y
Mac OS, Linux, Windows	| <img src="https://docs.microsoft.com/en-us/azure/active-directory/develop/media/index/logo_netcore.svg" width="40" /> <br/> .NET Core 	| Console	| N/A see [1](https://aka.ms/msal-net-uses-web-browser)	| Y	| Y	| Y

#### Confidential client applications 

##### the flows will rather:
- Acquire token **for the application itself**, not for a user, using [client credentials](Client-credential-flows). This can be used for syncing tools, or tools which process users in general, not a particular user. 
- In the case of Web APIs calling and API on behalf of the user, using the [On Behalf Of flow](on-behalf-of) and still identifying the application itself with client credentials to acquire a token based on some User assertion (SAML for instance, or a JWT token). This can be used for applications which need to access resources of a particular user in service to service calls.
- **For Web apps**, acquire tokens [by authorization code](Acquiring-tokens-with-authorization-codes-on-web-apps) after letting the user sign-in through the authorization request URL. This is typically the mechanism used by an open id connect application, which lets the user sign-in using Open ID connect, but then wants to access Web APIs on behalf of this particular user.

##### summary for confidential client applications
The following table summarizes the ways available to acquire tokens in confidential client applications depending on the Operating system, and therefore the chosen platform, and the kind of application.

Operating System | Library Platform | Kind of App | [Client Credential](Client-credential-flows) | [On behalf of](on-behalf-of) | [Auth Code](Acquiring-tokens-with-authorization-codes-on-web-apps)
 -- | -------- | --- | ----------  | --- | --- 
Windows	| <img src="https://docs.microsoft.com/en-us/azure/active-directory/develop/media/index/logo_net.svg" width="40"/> <br/> .NET Framework | ![image](https://user-images.githubusercontent.com/13203188/53402109-60f25680-39b1-11e9-868a-0dcad865cb31.png) | Y | | Y
Windows, MacOS, Linux | <img src="https://docs.microsoft.com/en-us/azure/active-directory/develop/media/index/logo_netcore.svg" width="40" /> <br/> ASP.NET Core  | ![image](https://user-images.githubusercontent.com/13203188/53425607-a6c91200-39e5-11e9-9a1d-c533bf546156.png) | Y | | Y | 	
Windows | <img src="https://docs.microsoft.com/en-us/azure/active-directory/develop/media/index/logo_net.svg" width="40"/> <br/> .NET Framework | ![image](https://user-images.githubusercontent.com/13203188/53402132-6ea7dc00-39b1-11e9-8c59-79d39e38f6bd.png) | Y | Y | 	
Windows, MacOS, Linux | <img src="https://docs.microsoft.com/en-us/azure/active-directory/develop/media/index/logo_netcore.svg" width="40" /><br/> ASP.NET Core | ![image](https://user-images.githubusercontent.com/13203188/53425997-6f0e9a00-39e6-11e9-9517-18dfdd359e37.png) | Y | Y | 	
Windows	| <img src="https://docs.microsoft.com/en-us/azure/active-directory/develop/media/index/logo_net.svg" width="40"/> <br/> .NET Framework | ![image](https://user-images.githubusercontent.com/13203188/53402143-74052680-39b1-11e9-9c8e-57cdca0a9c1d.png) <br/> (windows service) | Y | |
Windows, MacOS, Linux | <img src="https://docs.microsoft.com/en-us/azure/active-directory/develop/media/index/logo_netcore.svg" width="40" /> <br/> .NET Core | ![image](https://user-images.githubusercontent.com/13203188/53426315-0bd13780-39e7-11e9-8ac2-0ae9f6746370.png)| Y | | 	

### Pattern to acquire tokens in MSAL.NET

All the Acquire Token methods in MSAL.NET have the following pattern:
- from the application, you call the AcquireToken*XXX* method corresponding to the flow you want to use, passing the mandatory parameters for this flow (in general flow)
- this returns a command builder, on which you can add optional parameters using .With*YYY* methods
- then you call ExecuteAsync() to get your authentication result.

Here is the pattern:

```CSharp
AuthenticationResult result = app.AcquireTokenXXX(mandatory-parameters)
 .WithYYYParameter(optional-parameter)
 .ExecuteAsync();
```

## AuthenticationResult properties in MSAL.NET 

In all cases above, methods to acquire tokens return an ``AuthenticationResult`` (or in the case of the async methods a ``Task<AuthenticationResult>``.

In MSAL.NET, AuthenticationResult exposes:
- ``AccessToken`` for the Web API to access resources. This is a string, usually a base64 encoded JWT but the client should never look inside the access token. The format isn't guaranteed to remain stable and it can be encrypted for the resource. People writing code depending on access token content on the client is one of the biggest sources of errors and client logic breaks 
- ``IdToken`` for the user (this is a JWT)
- ``ExpiresOn`` tells the date/time when the token expires
- ``TenantId`` contains the tenant in which the user was found. Note that in the case of guest users (Azure AD B2B scenarios), the TenantId is the guest tenant, not the unique tenant.
When the token is delivered in the name of a user, ``AuthenticationResult`` also contains information about this user. For confidential client flows where tokens are requested with no user (for the application), this User information is null.
- The ``Scopes`` for which the token was issued (See [Scopes not resources](Adal-to-Msal#scopes-not-resources))
- The unique Id for the user.

### IAccount

MSAL.NET defines the notion of Account (through the `IAccount` interface). This breaking change provides the right semantics: the fact that the same user can have several accounts, in different Azure AD directories. Also MSAL.NET provides better information in the case of guest scenarios, as home account information is provided.
The following diagram shows the structure of the `IAccount` interface:

![image](https://user-images.githubusercontent.com/13203188/44657759-4f2df780-a9fe-11e8-97d1-1abbffade340.png)

The `AccountId` class identifies an account in a specific tenant. It has the following properties:

Property | Description
-------- | -----------
`TenantId` | A string representation for a GUID, which is the ID of the tenant where the account resides
`ObjectId` | A string representation for a GUID which is the ID of the user who owns the account in the tenant
`Identifier` | Unique identifier for the account (this is the concatenation of `ObjectId` and `TenantId` separated by a comma and are not base64 encoded)

The `IAccount` interface represents information about a single account. The same user can be present in different tenants, that is, a user can have multiple accounts. Its members are:

Property | Description
--- | ----
`Username` | A string containing the displayable value in UserPrincipalName (UPN) format, for example, john.doe@contoso.com. This can be null, whereas the HomeAccountId and HomeAccountId.Identifier won’t be null. This property replaces the `DisplayableId` property of `IUser` in previous versions of MSAL.NET.
`Environment` | A string containing the identity provider for this account, for example, `login.microsoftonline.com`. This property replaces the `IdentityProvider` property of `IUser`, except that `IdentityProvider` also had information about the tenant (in addition to the cloud environment), whereas here this is only the host.
`HomeAccountId` | AccountId of the home account for the user. This uniquely identifies the user across AAD tenants.