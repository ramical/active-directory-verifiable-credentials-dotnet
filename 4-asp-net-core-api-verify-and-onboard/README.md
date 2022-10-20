---
page_type: sample
languages:
- dotnet
products:
- microsoft entra
- verified id
description: "A code sample demonstrating verification of verifiable credentials and based on that onboarding new employees to Azure Active Directory."
urlFragment: "4-asp-net-core-api-verify-and-onboard"
---
# Verified ID Code Sample

This dotnet code sample is for a developer who wants to verify identity of a employee using Microsoft Entra Verified ID from an ISV partner. 
Upon successful verification the employee will be provided a Temporary access passcode to onboard to Azure Active Directory and configure 
authentication methods for subsequent logins. 

## About this sample

Welcome to Microsoft Entra Verified ID which includes the Request Service REST API. This API allows you to issue and verify credentials. 
This sample shows you how to verify Microsoft Entra Verified ID from an ISV partner offering Identity Verification and Proofing (IDV) using the Request Service REST API.

## Contents

The project is divided in 2 parts, one for issuance and one for verifying a verifiable credential. Depending on the scenario you need you can remove 1 part. To verify if your environment is completely working you can use both parts to issue a verifiedcredentialexpert VC and verify that as well.

| Verification | |
|------|--------|
| Pages/Verifier.cshtml | The website acting as the verifier of the verifiable credential, if you have one, else it provides option to get one from the issuer.
| VerifierController.cs | This is the controller which contains the API called from the webpage. It calls the REST API after getting an access token through MSAL and helps verifying the presented verifiable credential.
| presentation_request_config - TrueIdentitySample.json | The sample payload send to the server to verify identity of a user using Microsoft Entra Verified ID from an ISV partner - True Identity.

## Setup

Before you can run this sample make sure your environment is setup correctly, follow the instructions in the documentation [here](https://aka.ms/vcsetup).
The sample excepts you have a specific user say Alex (FirstName: Alex, Last Name: Wilber) already provisioned in Azure AD with a single auth method Temporary access code

### Create application registration
Run the [Configure.PS1](../1-asp-net-core-api-idtokenhint/AppCreationScripts/AppCreationScripts.md) powershell script in the AppCreationScripts directory or follow these manual steps to create an application registrations, give the application the correct permissions so it can access the Verified ID Request REST API:

Register an application in Azure Active Directory: 
1. Sign in to the Azure portal using either a work or school account or a personal Microsoft account.
2. Navigate to the Microsoft identity platform for developers App registrations page.
3.	Select New registration
    -  In the Name section, enter a meaningful application name for your issuance and/or verification application
    - In the supported account types section, select Accounts in this organizational directory only ({tenant name})
    - Select Register to create the application
4.	On the app overview page, find the Application (client) ID value and Directory (tenant) ID and record it for later.
5.	From the Certificates & secrets page, in the Client secrets section, choose New client secret:
    - Type a key description (for instance app secret)
    - Select a key duration.
    - When you press the Add button, the key value will be displayed, copy and save the value in a safe location.
    - You’ll need this key later to configure the sample application. This key value will not be displayed again, nor retrievable by any other means, so record it as soon as it is visible from the Azure portal.
6.	In the list of pages for the app, select API permissions
    - Click the Add a permission button
    - Search for APIs in my organization for  Verifiable Credentials Service Request and Verifiable Credentials Service service principals, and select them.
    - Click the “Application Permission” and expand “VerifiableCredential.Create.PresentRequest”
    - Select Add permissions.
7.  In the list of pages for the app, select API permissions
    - Click the Add a permission button
    - Select Microsoft APIs, then select Microsoft Graph, then select Application Permissions
    - Search for the UserAuthenticationMethod.ReadWrite.All and User.Read.All and select them.
    - Select Add permissions.
8.  Click Grant admin consent for {tenant name} on top of the API/Permission list and click YES. This allows the application to get the correct permissions
![Admin concent](ReadmeFiles/AdminConcent.PNG)
9.  Enabling the Temporary Access Pass (TAP) 
    - Sign in to the Azure portal as a Global admin and select Azure Active Directory > Security > Authentication methods > Temporary Access Pass
    - Select Yes to enable the policy and add Alex Wilber and select which users have the policy applied, and any General settings.
    Note: If you do not have an employee Alex Wilber that needs to be onboarded, replace Alex Wilber with another employee. In real world, new employees may be provisioned
    into Azure AD via HR ISVs. More details [here](https://learn.microsoft.com/en-us/azure/active-directory/app-provisioning/what-is-hr-driven-provisioning)
    Note:TBD: To ensure existing employees going through this onboarding flow cannot access https://aka.ms/mfasetup without providing TAP, TAP may need to be
    generated so existing users are prompted for it and they will need to go through this flow to get the TAP. The code needs an update to be able to handle the 
    scenario where user may have an active TAP already. 

## Setting up and running the sample
To run the sample, clone the repository, compile & run it. It's callback endpoint must be publically reachable, and for that reason, use `ngrok` as a reverse proxy to reach your app.

```Powershell
git clone https://github.com/Azure-Samples/active-directory-verifiable-credentials-dotnet.git
cd active-directory-verifiable-credentials-dotnet/4-asp-net-core-api-verify-and-onboard
```

### API Payloads
The API is called with special payload for verifying verifiable credentials. The sample payload files are modified by the sample code by copying the correct values from the `appsettings.json` file.
If you want to replace the payload `presentation_request_config - TrueIdentitySample.json` files yourself, make sure you update to the new json file in the VerifierController.cs file. 
The code overwrites the trustedIssuers values with IssuerAuthority value from appsettings.json. So make sure to copy the trustedIssuers value from the payload to IssuerAuthority in ''appsettings.json' file 
The callback URI is modified in code to match your hostname.

Make sure you copy the `ClientId`, `ClientSecret` and `TenantTd` you copied when creating the app registration to the `appsettings.json` as well.

## Running the sample

1. Open a command prompt and run the following command:
```Powershell
dotnet build "AspNetCoreVerifiableCredentials.csproj" -c Debug -o .\bin\Debug\net6
dotnet run
```

**Note** This sample is using .Net6 and if you are using Visual Studio instead of command line above, you need to install and use Visual Studio 2022 

2. Using a different command prompt, run ngrok to set up a URL on 5000. You can install ngrok globally by using the [ngrok npm package](https://www.npmjs.com/package/ngrok/).
```Powershell
ngrok http 5000
```
**Note**: The port number should be the same as the one configured in Properties\launchSettings.json and the Redirect URI configured in **Create application registration ** section above.

3. Open the HTTPS URL generated by ngrok.
![API Overview](ReadmeFiles/ngrok-url-screen.png)
The sample dynamically copies the hostname to be part of the callback URL, this way the VC Request service can reach your sample web application to execute the callback method.

1. Select ** Step 1: Get your card to proof your identity **
2. Follow the prompts to verify identity and Click ** OK ** on the ** Verification Complete ** screen.
1. In Authenticator, scan the QR code. 
> If this is the first time you are using Verifiable Credentials the Credentials page with the Scan QR button is hidden. You can use the `add account` button. Select `other` and scan the QR code, this will enable Verified Id in Authenticator.
6. If you see the 'This app or website may be risky screen', select **Advanced**.
1. On the next **This app or website may be risky** screen, select **Proceed anyways (unsafe)**.
1. On the Add a credential screen, notice that:

  - At the top of the screen, you can see a red **Not verified** message.
  - The credential is based on the information you uploaded as the display file.

9. Select **Add**.

## Verify the verifiable credential by using the sample app
1. Open the HTTPS URL generated by ngrok and click on the ** I already have my card ** link
2. Scan the QR code
3. select the VerifiedCredentialExpert credential and click allow
4. You should see the result presented on the screen.

## About the code
Since the API is now a multi-tenant API it needs to receive an access token when it's called. 
The endpoint of the API is https://verifiedid.did.msidentity.com/v1.0/verifiableCredentials/createPresentationRequest 

To get an access token we are using MSAL as library. MSAL supports the creation and caching of access token which are used when calling Azure Active Directory protected resources like the verifiable credential request API.
Typicall calling the libary looks something like this:
```C#
app = ConfidentialClientApplicationBuilder.Create(AppSettings.ClientId)
    .WithClientSecret(AppSettings.ClientSecret)
    .WithAuthority(new Uri(AppSettings.Authority))
    .Build();
```
And creating an access token:
```C#
result = await app.AcquireTokenForClient(scopes)
                  .ExecuteAsync();
```
> **Important**: At this moment the scope needs to be: **3db474b9-6a0c-4840-96ac-1fceb342124f/.default** 

Calling the API looks like this:
```C#
HttpClient client = new HttpClient();
var defaultRequestHeaders = client.DefaultRequestHeaders;
defaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", result.AccessToken);

HttpResponseMessage res = await client.PostAsync(AppSettings.ApiEndpoint, new StringContent(jsonString, Encoding.UTF8, "application/json"));
response = await res.Content.ReadAsStringAsync();
```

## Troubleshooting

### Did you forget to provide admin consent? This is needed for confidential apps.
If you get an error when calling the API `Insufficient privileges to complete the operation.`, this is because the tenant administrator has not granted permissions
to the application. See step 6 of 'Register the client app' above.

You will typically see, on the output window, something like the following:

```Json
Failed to call the Web Api: Forbidden
Content: {
  "error": {
    "code": "Authorization_RequestDenied",
    "message": "Insufficient privileges to complete the operation.",
    "innerError": {
      "request-id": "<a guid>",
      "date": "<date>"
    }
  }
}
```


## Best practices
When deploying applications which need client credentials and use secrets or certificates the more secure practice is to use certificates. If you are hosting your application on azure make sure you check how to deploy managed identities. This takes away the management and risks of secrets in your application.
You can find more information here:
- [Integrate a daemon app with Key Vault and MSI](https://github.com/Azure-Samples/active-directory-dotnetcore-daemon-v2/tree/master/3-Using-KeyVault)


## More information

For more information, see MSAL.NET's conceptual documentation:

- [Quickstart: Register an application with the Microsoft identity platform](https://docs.microsoft.com/azure/active-directory/develop/quickstart-register-app)
- [Quickstart: Configure a client application to access web APIs](https://docs.microsoft.com/azure/active-directory/develop/quickstart-configure-app-access-web-apis)
- [Acquiring a token for an application with client credential flows](https://aka.ms/msal-net-client-credentials)
