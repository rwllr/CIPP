---
description: API Authentication
---

# Setup & Authentication

### Enable the API

{% hint style="info" %}
Hosted clients can request enablement of the API via the helpdesk.
{% endhint %}

To enable the CIPP-API you'll need to activate the CIPP API Extension via the Settings -> Extensions menu.

Enabling the CIPP API requires the following:

* Your CIPP-SAM user must be a global administrator in your tenant when activating the API
* Your CIPP-SAM Application requires an extra permission
  * Go to your CIPP-SAM application via Settings -> Execute a permissions check -> Click Details -> Click on the CIPP SAM Link
  * Click on Add permission and add Azure Service Management - User Impersonation as a Delegate permission.
* Your CIPP-SAM user must have access to the Azure Subscription with the minimum level of "contributor" during activation of the API:
  1. Sign in to the Azure portal: [https://portal.azure.com/](https://portal.azure.com/)
  2. In the left-hand menu, navigate to "Subscriptions".
  3. Click on the subscription where you want to add a user.
  4. In the left-hand menu of the subscription, select "Access control (IAM)".
  5. At the top of the Access control (IAM) pane, click "+ Add".
  6. In the drop-down menu, select "Add role assignment".
  7. In the "Role" drop-down list, type "Contributor" and select it. The Contributor role should allow the user to create and manage all types of Azure resources but does not allow them to grant access to others.
  8. In the "Assign access to" drop-down menu, select "User, group, or service principal".
  9. In the "Select" field, type "CIPP-SAM". As you begin typing, the list of options will narrow. If the user CIPP-SAM exists in your Azure AD, you should be able to select it.
  10. After you've selected the user, click "Save" to assign the role.
* After enablement of the API a new application will be created in your tenant.

### Manually enabling the API
If your application tenant is different to your CIPP-SAM tenant, you will need to setup the API manually.

* Create the App Registration:
  1. Sign in to the Azure portal: [https://portal.azure.com/](https://portal.azure.com/)
  2. Navigate to Azure Active Directory -> App Registrations.
  3. At the top, click + New Registration. Enter;
     * Name: CIPP-API
     * Single Tenant
     * Redirect Uri:
       * Platform: Web
       * Uri: https://<CIPPInstance>>.azurewebsites.net/.auth/login/aad/callback
  4. Click Register
  5. In the left-hand menu choose Authentication
  6. Check ID tokens (used for implicit and hybrid flows)
  7. In the left-hand menu choose Overview and copy the Application ID
  8. In the left-hand menu choose Expose an API.
  9. Click Add a scope. Enter;
     * Application ID URI: api://<<Application ID>>
     * Scope name: user_impersonation
     * Who can consent? Admins and users
     * Admin consent display name: Access CIPP-API
     * Admin consent description: Access CIPP-API
  10. Click Add scope to save
  11. Go to Certificates & secrets
  12. Click New Client secret
  13. Note down the Value and keep it safe together with the Application ID and Tenant ID from the Overview tab

* Add the Identity Provider to Function App
  1. Sign in to the Azure portal: [https://portal.azure.com/](https://portal.azure.com/)
  2. Locate the CIPP Function App, which should be named cipp#####
  3. Choose the Authentication tab on the left hand side
  4. Click Add provider
  5. From the drop-down menu choose Microsoft. Enter;
     * Tenant type: Workforce
     * App registration type: Provide the details of an existing app registration
     * Application ID: Appplication ID noted from step above
     * Client secret: Leave blank
     * Issuer URL: https://sts.windows.net/<<TenantId>>/v2.0
     * Allowed token audiences: api://<<ApplicationID>>
   6. Click Create

### Authentication

CIPP uses OAuth authentication to be able to connect to the API using your Application ID and secret. You can use the PowerShell example below to connect to the API

```powershell
$CIPPAPIUrl = "https://yourcippurl.com"
$ApplicationId = "your application ID"
$ApplicationSecret = "your application secret"
$TenantId = "your tenant id"

$AuthBody = @{
    client_id     = $ApplicationId
    client_secret = $ApplicationSecret
    scope         = "api://$($ApplicationId)/.default"
    grant_type    = 'client_credentials'
}
$token = Invoke-RestMethod -Uri "https://login.microsoftonline.com/$TenantId/oauth2/v2.0/token" -Method POST -Body $AuthBody



$AuthHeader = @{ Authorization = "Bearer $($token.access_token)" }
Invoke-RestMethod -Uri "$CIPPAPIUrl/api/ListLogs" -Method GET -Headers $AuthHeader -ContentType "application/json"

```

### Time and rate limits

The API actions have a maximum timeout of 10 minutes. There are no active ratelimits, but heavy usage of the API can cause frontend operations to slow down.

## Endpoint documentation

Each page in the user documentation has a list of the endpoints used to load or create data on that specific page
