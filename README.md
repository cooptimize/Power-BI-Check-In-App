# Overview
The Power BI Checkin Power App expedites publishing Power BI content, deploying through Pipelines, and tracking changes.

<img width="1621" alt="image" src="https://user-images.githubusercontent.com/105446443/201536370-0c82a187-9366-4aa6-bb8d-471a3eb10675.png">

## Inside Power BI Desktop
The best part of this App is it sits inside Power BI Desktop - right where you are authoring Reports or Data Models. When you are finished making changes, the Documentation and Publishing experience is sitting in front of you.

You can also view the Change History inside the .pbix file.

## Tie SharePoint File to Workspace
One of the annoying parts of the existing Power BI Publishing experience is constantly having to select the Workspace to Publish to. We can get around that annoyance by putting the Workspace Id as the SharePoint Title field. Having a link between File and Workspace means the ongoing Publish process is simplified.

## Simplify Pipelines
Pipelines are a great way to be able to Test changes before they are promoted to Production. However, this has an unwanted side effect - making a quick change to a Report or Measure can be overly cumbersome. The Power BI App solves this by optionally automating Pipeline promotion steps.

## No Report when Publishing a Dataset
With the API, we can publish only a Data Model without publishing the report.

The App checks if a Dataset is published but a Report with the same name is not published (because you manually deleted it from PowerBI.com after the 1st publish).

## Links to PowerBI.com
Within the .pbix file, we know 1) the File Name and 2) the Workspace Id. From these two data points the App can retreive other Metadata. The App has direct links to the Workspace, Pipeline, Dataset, and Report. These links mean you can quickly view content after publishing.

[Installation Steps](INSTALLATION.md)

[Managed App Download](pbiapp_PowerBICheckinApp.zip)
