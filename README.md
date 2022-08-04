# ApigeeAuthProvider (Fork with Named Credential Secret Support)<br/>

ApigeeAuthProvider is an Auth Provider Plugin for Salesforce which will support an OAuth connection to an Apigee endpoint.  The primary goal of this version is to enhance security by storing the endpoint's Client Secret in the password field of a Named Credential which results in encryption and hiding the value.

## Changes in this fork:
- Updated API to 55
- Removed deprecated tags that fail deployment
- Added support for using an optional Named Credential to store encrypted, hidden Client Secret value 

## Dev -- Deploy to Scratch Org<br/> 
sfdx force:org:create -f config/project-scratch-def.json -a MyScratchOrg<br/>
sfdx force:source:push -u MyScratchOrg<br/>

## Dev -- Deploy to Sandbox<br/> 
sfdx force:source:convert -d temp/ --packagename ApigeeAuthProvider<br/>
sfdx force:mdapi:deploy -d temp/ -u "sandbox_username" -l RunSpecifiedTests -r ApigeeAuthProviderTest<br/>

## FAQ
Please note that this is a modified version of the original version by Bobby White and has not been fully regression tested.  If you do not intend to use Named Credentials to store endpoint information, please consider using the original version: https://github.com/bobbywhitesfdc/ApigeeAuthProvider

When configuring the AuthProvider, make sure that you set Name and URL Suffix to the same value!  If you don't, then you must override the Callback URL.  See https://github.com/bobbywhitesfdc/ApigeeAuthProvider/issues/1

Be sure to set any endpoint or authorization URL in Remote Site Settings.  If Remote Site Settings are not correct, you will see the following error when saving your named credential: "Problem Logging in - We can't log you in because of the following error.  For more information, contact your Salesforce administrator.  Remote_Error: The remote service returned an error"
