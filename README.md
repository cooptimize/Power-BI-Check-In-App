# Overview
The Power BI Checkin Power App expedites publishing Power BI content, deploying through Pipelines, and tracking changes.

<img width="1514" alt="image" src="https://user-images.githubusercontent.com/105446443/197231484-1cc0070d-6dd1-4fd2-bcd2-d3902282862a.png">

## Inside Power BI Desktop
The best part of this App is it sits inside Power BI Desktop - right where you are authoring Reports or Data Models. When you are finished making changes, you don't need to login somewhere else to document what changed.

You can also view the Change History inside the .pbix file.

## Tie SharePoint File to Workspace
One of the annoying parts of the existing Power BI Publishing experience is constantly having to select the Workspace to Publish to. We can get around that annoyance by putting the Workspace Id as the SharePoint Title field. We use this link to automate the Publish process.

## Simplify Pipelines
Pipelines are a great way to be able to Test changes before they are promoted to Production. However, this has an unwanted side effect - making a quick change to a Report or Measure can be overly cumbersome. The Power BI App solves this by optionally automating Pipeline promotion steps.

## No Report when Publishing a Dataset
With the API, we can publish only a Data Model without publishing the report.

## Links to PowerBI.com
By knowing which Workspace the .pbix file is linked to and the Name of the .pbix file, other Metadata can be gathered. The App has direct links to the Workspace, Pipeline, Dataset, and Report making it much faster to view content after publishing.

# Installation Requirements
You will need help from a Tenant Admin to get the App installed. The following components are required to setup the app:

| Component | Purpose | Access Required |
|---|---|---|
| SharePoint Site or Team | Stores .pbix files so they can be published to PowerBI.com | Ability to Create Team or SharePoint Site |
| Power App Environment | Install App and related components | Power Platform Admin |
| App Registration | Access to the Power BI API | Azure Active Directory |
| Register Provider for Azure Subscription | Needed for Power App to retreive the Key Vault | Subscription Owner |
| Create Azure Key Vault | Stores the Secret from the App Registration | Azure Resource Group Owner or Azure Administrator |
| (Optional) Azure AD Group | To manage API and App Access | Azure Active Directory |
| Install Power App | App and Power Automate components | Power App Environment System Administrator |
| Integrate with Power BI Content | Embed the app directly inside of Power BI Desktop | Power BI Publish |

# Installation Steps

## SharePoint Site or Team
A SharePoint site stores the .pbix files. Access the files via the OneDrive sync client so they automatically upload to SharePoint.

## Power App Environment
Creating a new environment isn't mandatory. But for an app like this, it's the simplest method.
https://admin.powerplatform.microsoft.com/environments
1. Create a **+New** environment.
2. Suggested **Name** "Power BI App".
3. **Type** "Production".
4. **Create a database for this environment** - Yes.
5. Other settings can be the default settings.

Once the environment is created, the Users who will manage App updates are added as System Adminstrator of the Environment (**Environment > Users See All > Add user**).

## App Registration
https://portal.azure.com/#view/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/~/RegisteredApps

The App Registration allows Power Automate to use the Power BI API. Giving access here is only one part of the security puzzle. The App Registration "user" must be given appropriate permissions to any Power BI Workspace, Pipeline, or App to perform API related functions.

1. Create a **+New** App Registration. Suggested name "Power BI API".
2. Click **API Permissions** then **+Add a Permission**.
3. Search for the Power BI Service and add: 
>* App.Read.All
>* Dashboard.Read.All
>* Dataflow.Read.All
>* Dataset.ReadWrite.All
>* Pipeline.Deploy
>* Report.ReadWrite.All
>* Workspace.Read.All
4. Click **Certificates & Secrets** then **+New Client Secret**. Suggested **Description** "Power BI API" **Expires** 24 months.
5. You only get one chance to copy the secret. Paste it in your password manager - it will be required in the next step.

## Register Provider for Azure Subscription
https://portal.azure.com/#view/Microsoft_Azure_Billing/SubscriptionsBlade

The Power Apps provider is off by default on an Azure Subscription. It needs to be on for the Subscription the Key Vault will use. This allows a Power App to communicate with a Key Vault.

1. Click on **Resource providers** then **Filter by name...** "Microsoft.PowerPlatform". **Register**

## Create Azure Key Vault
https://portal.azure.com/#view/HubsExtension/BrowseResource/resourceType/Microsoft.KeyVault%2Fvaults

A Key Vault allows a User to to retreive a secret without knowing the secret. Key Vaults may incur Azure Fees - for this App it will be pennies if anything.
1. **+Create** a Key Vault. Assign the appropriate Subscription and Resource Group.
2. The **Name** must be unique to Azure. Suggested Name "{yourcompanyname}powerbiapi".
3. Open the Key Vault once Azure has provisioned it.
4. Navigate to **Secrets** and **+Generate/Import**. **Name** is "PowerBIAPI" and the Secret is pasted from the App Registration step.
5. Navigate to **Access policies** and **+Create**.
6. Users need **Secret > Get** permissions to retreive the secret.

## Send Information to the Power BI App Installer
The person installing the app needs to know some values from Azure.

**App Registration**
* Application (client) ID
* Directory (tenant) ID

**Azure Key Vault**
* Key Vault Name (eg {yourcompanyname}powerbiapi)
* Resource Group Name
* Subscription Id
* Secret Name (eg "PowerBIAPI")

## (Optional) Azure AD Group
The AD Group could be more specific to this App, or tied into existing Power BI Groups.

Suggested AD Group Name: "Power BI App"

Members:
* Users who will Publish Power BI content
* The Power BI API App Registration

Group Assignments:
* Key Vault Access policies
* Power App - App level permissions
* (Optional) Power BI Workspaces and Pipelines (Member level)

## Power App Installation
https://make.powerapps.com/environments

1. Navigate to the Power Apps Authoring site (https://make.powerapps.com/environments).
2. At the very top of the page is an **Environment** selector. Select the Power BI App Environment.
3. Navigate to **Solutions** and **Import**.
