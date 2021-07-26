# Enzoic API Integrations for Azure AD B2C

## Introduction
This repository contains everything necessary to integrate the Enzoic API into an Azure Active Directory B2C configuration.   

### Requirements
* You will need an Azure account to host the Azure AD B2C Tenant.
* You will also need Enzoic API Keys.

### Azure AD B2C and custom policies
This assumes that you have some familiarity with Azure Active Directory B2C and more specifically with creating custom policies to implement user flows.  If you are totally new to Azure AD B2C, I would suggest you check out the [Azure Active Directory B2C Documentation](https://docs.microsoft.com/en-us/azure/active-directory-b2c/).
If you already know the basics, but are new to creating custom user flows, I would suggest you first become familiar with the the [Custom Policy Overview](https://docs.microsoft.com/en-us/azure/active-directory-b2c/custom-policy-overview).

### Overview
The next several sections walk you through preparing your Azure environment and deploying custom policies with Enzoic integration.
* Prepare your environment - Setup and configure your environment to host the starter policies.
* Starter policies - Configure and deploy a set of starter policies for signup/signin, password reset, and password change.
* Enzoic policies - Integrate Enzoic password validation into the starter policies.
  
_If you are already experienced with Azure AD B2C and have a set of custom policies in use, you can jump directly to the `Enzoic policies` section to see how to integrate the Enzoic API into an existing set of policies._  

_We've also provided a set of policies with Enzoic already integrated. Details on these can be found in the `policies/LocalAccountsWithEnzoic` directory and the `README.md` file found there._

## Prepare your environment
The first step is to prepare your environment to host the custom user flows implemented in the starter policies.

These instructions are based on this [tutorial](https://docs.microsoft.com/en-us/azure/active-directory-b2c/tutorial-create-user-flows?pivots=b2c-custom-policy#register-identity-experience-framework-applications).  For the sake of simplicity, we have removed the parts about integrating with Facebook.

If you haven't already, you will need to work through these initial instructions to prepare your Azure account to work with the custom policies:
-  [Create an Azure AD B2C tenant](https://docs.microsoft.com/en-us/azure/active-directory-b2c/tutorial-create-tenant)
-  [Register a web application](https://docs.microsoft.com/en-us/azure/active-directory-b2c/tutorial-register-applications), and [enable ID token implicit grant](https://docs.microsoft.com/en-us/azure/active-directory-b2c/tutorial-register-applications#enable-id-token-implicit-grant) using the Azure portal so you'll be able to test your policy.   This will walk you through creating a simple Azure Web Application that will allow you to test your custom policies without an actual web application.

---

### Add signing and encryption keys
Follow these steps to create the necessary encryption and signing keys needed for the custom policies.

1.  Sign in to the [Azure portal](https://portal.azure.com/).
2.  Select the **Directory + Subscription** icon in the portal toolbar, and then select the directory that contains your Azure AD B2C tenant.
3.  In the Azure portal, search for and select **Azure AD B2C**.
4.  On the overview page, under **Policies**, select **Identity Experience Framework**.

#### Create the signing key

1.  Select **Policy Keys** and then select **Add**.
2.  For **Options**, choose `Generate`.
3.  In **Name**, enter `TokenSigningKeyContainer`. The prefix `B2C_1A_` might be added automatically.
4.  For **Key type**, select **RSA**.
5.  For **Key usage**, select **Signature**.
6.  Select **Create**.

#### Create the encryption key

1.  Select **Policy Keys** and then select **Add**.
2.  For **Options**, choose `Generate`.
3.  In **Name**, enter `TokenEncryptionKeyContainer`. The prefix `B2C_1A`\_ might be added automatically.
4.  For **Key type**, select **RSA**.
5.  For **Key usage**, select **Encryption**.
6.  Select **Create**.

### Register Identity Experience Framework applications
Azure AD B2C requires you to register two applications that it uses to sign up and sign in users with local accounts: _IdentityExperienceFramework_, a web API, and _ProxyIdentityExperienceFramework_, a native app with delegated permission to the IdentityExperienceFramework app. Your users can sign up with an email address or username and a password to access your tenant-registered applications, which creates a "local account." Local accounts exist only in your Azure AD B2C tenant.

You need to register these two applications in your Azure AD B2C tenant only once.

#### Register the IdentityExperienceFramework application

To register an application in your Azure AD B2C tenant, you can use the **App registrations** experience.

1.  Select **App registrations**, and then select **New registration**.
2.  For **Name**, enter `IdentityExperienceFramework`.
3.  Under **Supported account types**, select **Accounts in this organizational directory only**.
4.  Under **Redirect URI**, select **Web**, and then enter `https://your-tenant-name.b2clogin.com/your-tenant-name.onmicrosoft.com`, where `your-tenant-name` is your Azure AD B2C tenant domain name.
5.  Under **Permissions**, select the _Grant admin consent to openid and offline\_access permissions_ check box.
6.  Select **Register**.
7.  Record the **Application (client) ID** for use in a later step.

Next, expose the API by adding a scope:

1.  In the left menu, under **Manage**, select **Expose an API**.
2.  Select **Add a scope**, then select **Save and continue** to accept the default application ID URI.
3.  Enter the following values to create a scope that allows custom policy execution in your Azure AD B2C tenant:
    -   **Scope name**: `user_impersonation`
    -   **Admin consent display name**: `Access IdentityExperienceFramework`
    -   **Admin consent description**: `Allow the application to access IdentityExperienceFramework on behalf of the signed-in user.`
4.  Select **Add scope**

---

#### Register the ProxyIdentityExperienceFramework application

1.  Select **App registrations**, and then select **New registration**.
2.  For **Name**, enter `ProxyIdentityExperienceFramework`.
3.  Under **Supported account types**, select **Accounts in this organizational directory only**.
4.  Under **Redirect URI**, use the drop-down to select **Public client/native (mobile & desktop)**.
5.  For **Redirect URI**, enter `myapp://auth`.
6.  Under **Permissions**, select the _Grant admin consent to openid and offline\_access permissions_ check box.
7.  Select **Register**.
8.  Record the **Application (client) ID** for use in a later step.

Next, specify that the application should be treated as a public client:

1.  In the left menu, under **Manage**, select **Authentication**.
2.  Under **Advanced settings**, in the **Allow public client flows** section, set **Enable the following mobile and desktop flows** to **Yes**. Ensure that **"allowPublicClient": true** is set in the application manifest.
3.  Select **Save**.

Now, grant permissions to the API scope you exposed earlier in the _IdentityExperienceFramework_ registration:

1.  In the left menu, under **Manage**, select **API permissions**.
2.  Under **Configured permissions**, select **Add a permission**.
3.  Select the **My APIs** tab, then select the **IdentityExperienceFramework** application.
4.  Under **Permission**, select the **user\_impersonation** scope that you defined earlier.
5.  Select **Add permissions**. As directed, wait a few minutes before proceeding to the next step.
6.  Select **Grant admin consent for (your tenant name)**.
7.  Select your currently signed-in administrator account, or sign in with an account in your Azure AD B2C tenant that's been assigned at least the _Cloud application administrator_ role.
8.  Select **Accept**.
9.  Select **Refresh**, and then verify that "Granted for ..." appears under **Status** for the scopes - offline\_access, openid and user\_impersonation. It might take a few minutes for the permissions to propagate.

---

## Starter policies
Once you have all of the prerequisites taken care of you will need to modify the starter policies to reference your environment. These policies will deploy a set of user flows for common use cases such as signup/signin, password reset, and password change. This will provide us with a baseline that we can use to walk through adding the Enzoic integration.

These starter policies are based on the [LocalAccounts policies](https://github.com/Azure-Samples/active-directory-b2c-custom-policy-starterpack/tree/master/LocalAccounts/readme.md) provided in the [Azure AD B2C Custom Policy Starter Pack](https://github.com/Azure-Samples/active-directory-b2c-custom-policy-starterpack). We have made a few small changes to support a password change polic.

### Add your tenant ID to the custom policy files
1.  In all of the policy files in the `policies/LocalAccounts` directory, replace the string `yourtenant` with the name of your Azure AD B2C tenant.
    
    For example, if the name of your B2C tenant is _acme_, all instances of `yourtenant.onmicrosoft.com` become `acme.onmicrosoft.com`.
	
### Add application IDs to the custom policy's Extensions file
Add the application IDs to the extensions file _TrustFrameworkExtensions.xml_.  These are the application IDs you created above for the `IdentityExperienceFrameworkApp` and the `ProxyIdentityExperienceFrameworkApp`.

1.  Open `policies/LocalAccounts/TrustFrameworkExtensions.xml` and find the element `<TechnicalProfile Id="login-NonInteractive">`.
2.  Replace both instances of `IdentityExperienceFrameworkAppId` with the application ID of the IdentityExperienceFramework application that you created earlier.
3.  Replace both instances of `ProxyIdentityExperienceFrameworkAppId` with the application ID of the ProxyIdentityExperienceFramework application that you created earlier.
4.  Save the file.

### Upload the policies
1.  Select the **Identity Experience Framework** menu item in your B2C tenant in the Azure portal.
2.  Select **Upload custom policy**.
3.  In this order, upload the policy files:
    1.  _TrustFrameworkBase.xml_
    2.  _TrustFrameworkExtensions.xml_
    3.  _SignUpOrSignin.xml_
    4.  _PasswordReset.xml_
    5.  _PasswordChange.xml_

As you upload the files, Azure adds the prefix `B2C_1A_` to each.

### Test a custom policy
1.  Under **Custom policies**, select **B2C_1A_SignUpOrSignIn**.
2.  For **Select application** on the overview page of the custom policy, select the web application named _webapp1_ that you previously registered.
3.  Make sure that the **Reply URL** is `https://jwt.ms`.
4.  Select **Run now**.
5.  Sign up using an email address.
6.  Select **Run now** again.
7.  Sign in with the same account to confirm that you have the correct configuration.

---

## Enzoic custom policies
This section will walk you through adding the Enzoic integration to the starter policies.  It should be a similar process to integrate with your own existing policies.

_If you would prefer to skip the integration walkthrough and just deploy the finished project, you can check out the `README.md` file in the `policies/LocalAccountsWithEnzoic` directory and the policies contained there!_

### Add Enzoic API and Secret Keys
Follow these steps to create the necessary Policy Key containers for the Enzoic API keys.

#### Create the Enzoic API key

1.  Select **Policy Keys** and then select **Add**.
2.  For **Options**, choose `Manual`.
3.  In **Name**, enter `EnzoicApiKeyContainer`. The prefix `B2C_1A_` might be added automatically.
4.  For **Secret**, enter your Enzoic API Key.
5.  For **Key usage**, select **Signature**.
6.  Select **Create**.

#### Create the Enzoic Secret key

1.  Select **Policy Keys** and then select **Add**.
2.  For **Options**, choose `Manual`.
3.  In **Name**, enter `EnzoicSecretKeyContainer`. The prefix `B2C_1A_` might be added automatically.
4.  For **Secret**, enter your Enzoic Secret Key.
5.  For **Key usage**, select **Encryption**.
6.  Select **Create**.

--- 


### Adding Enzoic to combined SignUpOrSignIn policy
To add password validation to the starter SignUpSignIn combined policy you will just need to add a new [Technical Profile](https://docs.microsoft.com/en-us/azure/active-directory-b2c/technicalprofiles) that calls the Enzoic API, and also add a reference to it as a [Validation Technical Profile](https://docs.microsoft.com/en-us/azure/active-directory-b2c/validation-technical-profile) into an already existing profile. For more information on integrating a RESTful API into custom policies, check this [link](https://docs.microsoft.com/en-us/azure/active-directory-b2c/restful-technical-profile#returning-validation-error-message) to Microsoft's documentation.

* Add the new `response_format` and `compromised_message` `ClaimTypes` to the [ClaimsSchema](https://docs.microsoft.com/en-us/azure/active-directory-b2c/claimsschema) section of the [BuildBlocks](https://docs.microsoft.com/en-us/azure/active-directory-b2c/buildingblocks) section of your _TrustFrameworkExtensions.xml_ file.

```xml
<BuildingBlocks>
	<ClaimsSchema>

		...

		<ClaimType Id="response_format">
			<DisplayName>Enzoic API Response Format</DisplayName>
			<DataType>int</DataType>
		</ClaimType>
		<ClaimType Id="compromised_message">
			<DisplayName>Message returned by Enzoic API</DisplayName>
			<DataType>string</DataType>
		</ClaimType>
	</ClaimsSchema>
</BuildingBlocks>
```

* Add a new `ClaimsProvider` to the [ClaimsProviders](https://docs.microsoft.com/en-us/azure/active-directory-b2c/claimsproviders) section of your _TrustFrameworkExtensions.xml_ file that will contain the Technical Profiles related to querying the Enzoic API.

```xml
<ClaimsProvider>
	<DisplayName>Enzoic</DisplayName>
	<TechnicalProfiles>
		<TechnicalProfile Id="Enzoic-RestApiPasswordCheck">
			<DisplayName>Validate user password</DisplayName>
			<Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.RestfulProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
			<Metadata>
				<Item Key="ServiceUrl">https://api.enzoic.com/passwords</Item>
				<Item Key="AuthenticationType">Basic</Item>
				<Item Key="SendClaimsIn">QueryString</Item>
			</Metadata>
			<CryptographicKeys>
				<Key Id="BasicAuthenticationUsername" StorageReferenceId="B2C_1A_EnzoicApiKeyContainer" />
				<Key Id="BasicAuthenticationPassword" StorageReferenceId="B2C_1A_EnzoicSecretKeyContainer" />
			</CryptographicKeys>          
			<InputClaims>
				<InputClaim ClaimTypeReferenceId="newPassword" PartnerClaimType="password" />
				<InputClaim ClaimTypeReferenceId="response_format" DefaultValue="1" />
				<InputClaim ClaimTypeReferenceId="compromised_message" DefaultValue="Bad Password" />
			</InputClaims>
			<UseTechnicalProfileForSessionManagement ReferenceId="SM-Noop" />
		</TechnicalProfile>
		
		<!-- Change LocalAccountSignUpWithLogonEmail (defined in TrustFrameworkBase.xml) to use your new technical profile for validation -->
		<TechnicalProfile Id="LocalAccountSignUpWithLogonEmail">
			<ValidationTechnicalProfiles>
			<ValidationTechnicalProfile ReferenceId="Enzoic-RestApiPasswordCheck" />
			</ValidationTechnicalProfiles>
		</TechnicalProfile>			  
	</TechnicalProfiles>
</ClaimsProvider>		  
```

_If you would prefer to validate full credential sets rather than just passwords check out the `EnzoicRestApiCredentialCheck` technical profile found in the `profiles/LocalAccountsWithEnzoic/TrustFrameworkExtensions.xml` file_

The Technical Profile with the `Id` of `Enzoic-RestApiPasswordCheck` is used to query the Enzoic API.  It will return a `200` if the password has not been exposed, or a `409` along with a [result formatted for Azure AD B2C](https://docs.microsoft.com/en-us/azure/active-directory-b2c/restful-technical-profile#returning-validation-error-message) if it has been seen in any of the exposures found in the Enzoic database.

The next Technical Profile with the `Id` of `LocalAccountSignUpWithLogonEmail` was originally defined in the _TrustFrameworkBase.xml_ file.  This override is just adding a Validation Technical Profile to use the `Enzoic-RestApiPasswordCheck` for validation. Depending on the results from Enzoic it will allow the SignUp to continue, or display an error message and force the user to try again.

You should now be able to now upload your updated _TrustFrameworkExtensions.xml_ file and test the policy and as described above.

### Adding Enzoic to PasswordReset policy
This section will continue the walk through by showing how to update the PasswordReset policy to use the Enzoic integration.

* Starting from the previously modified _TrustFrameworkExtensions.xml_ file, we will update another Technical Profile in the Enzoic `ClaimProvider`.  This just adds an additional Validation Technical Profile into the Technical Profile as it was defined in the Base file.

```xml
<ClaimsProvider>
	<DisplayName>Enzoic</DisplayName>
	<TechnicalProfiles>
		
		...
		
		<TechnicalProfile Id="LocalAccountWritePasswordUsingObjectId">
			<ValidationTechnicalProfiles>
			<ValidationTechnicalProfile ReferenceId="Enzoic-RestApiPasswordCheck" />
			<ValidationTechnicalProfile ReferenceId="AAD-UserWritePasswordUsingObjectId" />
			</ValidationTechnicalProfiles>
		</TechnicalProfile>
	</TechnicalProfiles>

```

* You should now be able to upload and test the _PasswordReset_ policy.

### Adding Enzoic to PasswordChange policy
We finish the walk through by adding Enzoic to the PasswordChange policy.

* Starting from the previously modified _TrustFrameworkExtensions.xml_ file, we will add a new `ValidationTechnicalProfile` into the `TechnicalProfile` with the `Id` of `LocalAccountWritePasswordChangeUsingObjectId` in the `Local Account` `ClaimsProvider`.  if you compare the profile below we are just adding a reference to the `Enzoic-RestApiPasswordCheck` Profile as a Validation immediately before we actually write the new password to Azure AD.
  
```xml
<ClaimsProvider>
  <DisplayName>Local Account</DisplayName>
    <TechnicalProfiles>
		<TechnicalProfile Id\="LocalAccountWritePasswordChangeUsingObjectId">
			<DisplayName>Change password (username)</DisplayName>
			<Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.SelfAssertedAttributeProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
			<Metadata>
				<Item Key="ContentDefinitionReferenceId">api.selfasserted</Item>
			</Metadata>
			<InputClaims>
				<InputClaim ClaimTypeReferenceId="objectId" />
			</InputClaims>
			<OutputClaims>
				<OutputClaim ClaimTypeReferenceId="oldPassword" Required="true" />
				<OutputClaim ClaimTypeReferenceId="newPassword" Required\="true" />
				<OutputClaim ClaimTypeReferenceId="reenterPassword" Required\="true" />
			</OutputClaims>
			<ValidationTechnicalProfiles>
				<ValidationTechnicalProfile ReferenceId="login-NonInteractive-PasswordChange" />
				<ValidationTechnicalProfile ReferenceId="Enzoic-RestApiPasswordCheck" />
				<ValidationTechnicalProfile ReferenceId="AAD-UserWritePasswordUsingObjectId" />
			</ValidationTechnicalProfiles>
	  	</TechnicalProfile>
    </TechnicalProfiles>
</ClaimsProvider>		  
```


