# Installation Requirements
You will need help from a Tenant Admin to get the App installed. The following components are required to setup the app:

| Component | Purpose | Access Required |
|---|---|---|
| SharePoint Site or Team | Stores .pbix files so they can be published to PowerBI.com | Ability to Create Team or SharePoint Site |
| Power App Environment | Install App and related components | Power Platform Admin |
| Power App Per User Licenses | The App requires a license, as it uses some premium components | Microsoft Billing Administrator and Power Platform Admin|
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

<img width="1511" alt="image" src="https://user-images.githubusercontent.com/105446443/197890385-b48bd2fe-8c99-4300-af42-82450210ac85.png">


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

# Power App Installation
https://make.powerapps.com/environments

## Managed vs Unmanaged
We recommend testing the functionality of the app using the managed solution. Managed solutions make it possible to install future updates and fixes. Installing the unmanaged solution means future updates won't be possible - effectively branching the code to your own version.

If you like some components of the app, but don't want to 

## Install the App

1. Download the Managed solution from GitHub.
2. Navigate to the Power Apps Authoring site (https://make.powerapps.com/environments).
3. At the very top of the page is an **Environment** selector. Select the Power BI App Environment.
4. Navigate to **Solutions** and **Import**.
5. **Browse** for the Solution available on GitHub.
6. You'll be prompted to create new Connections.

# PowerBI.com
The App Registration "user" needs to have Member or Admin access to all Workspaces and Pipelines you will publish to.

# Power BI Data Model
Your Published Dataset has to contain a list of SharePoint files. The simplest method to accomplish this is to add a hidden table to the model.

Add the following Power Query (plus any custom filters) to your model in Power BI Desktop:

```
let
    Source = SharePoint.Tables("https://cooptimizes.sharepoint.com/sites/PowerBIFilesandLists", [ApiVersion = 15]),
    #"fa0b5bfd-c2e1-4bec-9ad6-8e684eb2ecc5" = Source{[Title="Documents"]}[Items],
    #"Removed Other Columns" = Table.SelectColumns(#"fa0b5bfd-c2e1-4bec-9ad6-8e684eb2ecc5",{"Id", "Title", "File"}),
    #"Expanded File" = Table.ExpandRecordColumn(#"Removed Other Columns", "File", {"Name", "ServerRelativeUrl"}, {"Name", "Web URL"}),
    #"Filtered .pbix Files" = Table.SelectRows(#"Expanded File", each Text.EndsWith([Name], ".pbix")),
    #"Renamed Columns" = Table.RenameColumns(#"Filtered .pbix Files",{{"Id", "SharePoint Id"}}),
    #"Changed Type" = Table.TransformColumnTypes(#"Renamed Columns",{{"SharePoint Id", Int64.Type}})
in
    #"Changed Type"
```

Remember when publishing this file to Edit Connections in the dataset to autorize the SharePoint connection.
