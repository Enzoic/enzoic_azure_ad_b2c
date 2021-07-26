# Enzoic Custom Policies
These custom policies will validate passwords with the Enzoic API when a user signs up, resets, or changes their password.   

## Requirements
* Azure AD B2C Tenant ID
* Application IDs referencing both the `IdentityExperienceFramework` app, as well as the `ProxyIdentityExperienceFramework` app created above.
* Enzoic API Keys

## Add Enzoic API and Secret Keys
Follow these steps to create the necessary Policy Key containers for the Enzoic API keys.

### Create the Enzoic API key

1.  Select **Policy Keys** and then select **Add**.
2.  For **Options**, choose `Manual`.
3.  In **Name**, enter `EnzoicApiKeyContainer`. The prefix `B2C_1A_` might be added automatically.
4.  For **Secret**, enter your Enzoic API Key.
5.  For **Key usage**, select **Signature**.
6.  Select **Create**.

### Create the Enzoic Secret key

1.  Select **Policy Keys** and then select **Add**.
2.  For **Options**, choose `Manual`.
3.  In **Name**, enter `EnzoicSecretKeyContainer`. The prefix `B2C_1A_` might be added automatically.
4.  For **Secret**, enter your Enzoic Secret Key.
5.  For **Key usage**, select **Encryption**.
6.  Select **Create**.
7.  

--- 

### Configuration
To quickly see these policies in action you can just update the XML files in the `policies/LocalAccountsWithEnzoic` directory as explained below, and then upload and test the files as described in `Starter Policies` section of the main README. These modified policies are designed to live side by side with the Starter Policies. 

You will need to replace the following text in the XML files before uploading them:
1.  In all of the files in the `policies/LocalAccounts` directory of this repository, replace the string `YourTenant` with the name of your Azure AD B2C tenant.
2.  in the _TrustFrameworkExtensions.xml_ file, replace both instances of `IdentityExperienceFrameworkAppId` with the application ID of the IdentityExperienceFramework application that you created earlier.
3.  In the _TrustFrameworkExtensions.xml_ file, replace both instances of `ProxyIdentityExperienceFrameworkAppId` with the application ID of the ProxyIdentityExperienceFramework application that you created earlier.

### Customization
If you would rather validate full credential sets rather than just passwords, you will need to ... TODO


