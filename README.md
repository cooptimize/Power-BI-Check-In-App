# Overview
The Power BI Checkin App expedites publishing Power BI content, deploying through Pipelines, and tracking changes. The Power App sits inside Power BI Desktop.

<img width="1514" alt="image" src="https://user-images.githubusercontent.com/105446443/196832639-6f8bd359-9376-44ae-acaa-877018865426.png">

# Installation Requirements
You will need help from a Tenant Admin to get the App installed. The following components are required to setup the app:

| Component | Purpose | Access Required |
|---|---|---|
| SharePoint Site or Team | Stores .pbix files so they can be published to PowerBI.com | Ability to Create Team or SharePoint Site |
| Power App Environment | Install App and related components | Power Platform Admin |
| App Registration | Access to the Power BI API | Azure Active Directory |
| Register Provider for Azure Subscription | Needed for Power App to retreive the Key Vault | Subscription Owner |
| Create Azure Key Vault | Stores the Secret from the App Registration | Azure Resource Group Owner or Azure Administrator |
| Install Power App | App and Power Automate components | Power App Environment System Administrator |

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
The Power Apps provider is off by default on an Azure Subscription. It needs to be on for the Subscription the Key Vault will use.

1. 
