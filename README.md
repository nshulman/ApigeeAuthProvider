# ApigeeAuthProvider Plug In (Nathan's Fork with Named Credential Secret Support)

ApigeeAuthProvider is an Auth Provider Plugin for Salesforce (a.k.a. [AuthProviderPlugInClass](https://developer.salesforce.com/docs/atlas.en-us.apexref.meta/apexref/apex_class_Auth_AuthProviderPluginClass.htm)) which will support an OAuth authorization flow to an Apigee endpoint. The primary goal of this version is to enhance security by storing the endpoint's Client Secret in the password field of a Named Credential which results in both encryption when storing the secret and hiding all visibility of the value.  Once stored, the Client Secret value can not be viewed in the Setup Auth. Provider UI, Custom Metadata, or extracted in Apex, since it is stored as a password and sent server to server using [Named Credential merge fields](https://developer.salesforce.com/docs/atlas.en-us.202.0.apexcode.meta/apexcode/apex_callouts_named_credentials.htm).

This is a modified version of the original by Bobby White and has not been fully regression tested. This has been tested with both Named Credentials and the original approach of custom Auth. Provider fields.  However, if you do not intend to use Named Credentials to store endpoint information, please consider using the original version: https://github.com/bobbywhitesfdc/ApigeeAuthProvider.  Many thanks to Bobby for the idea for the approach, spec writeup and assistance with challenges encountered while making the updates.

## Changes in this fork:

- Updated API to 55
- Removed deprecated tags that fail deployment
- Added support for using an optional Named Credential to store encrypted, hidden Client Secret value

## Using ApigeeAuthProvider

1. Deploy the code to your dev environment
>**Scratch Org**
>
>`sfdx force:org:create -f config/project-scratch-def.json -a MyScratchOrg`
>
>`sfdx force:source:push -u MyScratchOrg`
>
>**Sandbox**
>
>`sfdx force:source:convert -d temp/ --packagename ApigeeAuthProvider`
>
>`sfdx force:mdapi:deploy -d temp/ -u "sandbox_username" -l RunSpecifiedTests -r ApigeeAuthProviderTest`

2. Create an Auth Provider
- Open Setup -> Security -> Auth. Providers
- Create a new Auth Provider and select type ApigeeAuthProvider
- Fill out Name (e.g. ApigeeEval)
- Fill out URL Suffix (e.g. ApigeeEval)
- Fill out Access Token URL (where the initial login takes place and the bearer token is issued) - **leave blank if using named credential**
- Fill out Auth Provider Name (this should match URL Suffix - e.g. ApigeeEval)
- Leave Callback URL blank, unless you have a custom Callback URL
- Fill out Client Id (e.g. 123abcde-12ab-123de) - **leave blank if using named credential**
- Fill out Client Secret - **leave blank if using named credential**
- Fill out Scope here if you have an application Scope.  *note: this is the only Scope used by ApigeeAuthProvider, the Scope value in the Named Credential will not be used*
- If a Named Credential will be used for the auth endpoint, follow the steps below and use the developer name (e.g. ApigeeEval_AuthEndpoint).  Otherwise, leave this field blank.
- The additional fields can be left blank, unless they are needed for your implementation

*Optional: Using a Named Credential to store the Authorization Endpoint secrets:*
- Open Setup -> Security -> Named Credentials
- Create New
- Set the URL field to the Apigee authorization endpoint. **Be sure to also add this URL to remote site settings.**
- Set Identity Type to Named Principal
- Set Authentication Protocol to Password Authentication
- Store the Client Id in the **Username** field
- Store the Client Secret in the **Password** field
- Under Callout Options, check **Allow Merge Fields in HTTP Body**.  Do not check any other options.

3. Create a Named Credential for your Apigee Endpoint *(NOT the Authorization endpoint, this is where you will make your API calls)*
- Open Setup -> Security -> Named Credentials
- Select New Named Credential
- Fill out Label and Name (e.g. ApigeeEval)
- Enter the API Endpoint URL *(NOT the auth endpoint)* in the URL field  **Be sure to also add this URL to remote site settings.**
- Set Identity Type to Named Principal
- Set Authentication Protocal to OAuth 2.0
- Set Authentication Provider to the Auth Provider you created in step 2 (should be type ApigeeAuthProvider)
- Leave Scope blank - this is ignored by ApigeeAuthProvider
- Check **Start Authentication Flow on Save**
- Under Callout Options, only **Generate Authorization Header** should be checked
- Click Save.  If this is successful, the Authentication Status will be **Authenticated**.  If you see Pending, then Authentication failed.

## PREVENTING COMMON ISSUES

When configuring the AuthProvider, make sure that you set Name and URL Suffix to the same value! If you don't, then you must override the Callback URL. See https://github.com/bobbywhitesfdc/ApigeeAuthProvider/issues/1

Be sure to set any endpoint or authorization URL in Remote Site Settings. If Remote Site Settings are not correct, you will see the following error when saving your named credential: "Problem Logging in - We can't log you in because of the following error. For more information, contact your Salesforce administrator. Remote_Error: The remote service returned an error"

The Custom Fields (e.g. Client Id) stored by this plugin, although visible and editable in the Auth Provider screen in Setup, are stored in ApigeeAuthProvider Custom Metadata records, and can also be seen in Custom Metadata.  Although these Custom Metadata records may be modified directly and will impact the provider, it is best to let the Auth Provider screen manage this record and values.  Basically, don't be surprised when you see a record in Custom Metadata that matches your Auth Provider values - they're simply passed through to the screen.
