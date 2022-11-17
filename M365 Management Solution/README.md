# M365 Management Solution

Are you struggling to keep up with the Service Health Dashboard (Incidents/Advisories), or better yet, can’t keep up with the Message Center?

By leveraging the Power Platform, we can create a one stop shop solution for the M365 Admin team to stay on top of SHD, Message Center, MS RoadMap, etc.

![M365 Management App](https://github.com/morismm99/PowerApps/blob/main/M365%20Management%20Solution/HomeScreen.png?raw=true)

This solution uses the following:

1. SharePoint Online lists to store SHD, Message Center and RoadMap data (premium data sources like Dataverse or SQL could be used instead)
2. Power Automate flows run on a schedule to scrape the MS Graph and store data in the SPO lists
3. Power Apps canvas app is created to easily view and interact with this data
4. Power Bi reports can be created to analyze this data
5. The app can be added as a tab in an MS Teams channel as a tab for quick access!

App Registration

The first step is creating an Azure App Registration with the necessary permissions. It will be used in two of the flows which use the HTTP Action to do a GET REST API call from the MS GRAPH API. Permissions required for the app registration are ServiceHealth.Read.All (Delegaged/Application) - this is for the Service Health Dashboard; ServiceMessage.Read.all (Delegated/Application) - this for the Message Center.

You will need the following after your App Registration has been created:
1. Application (client) ID
2. Tenant ID
3. App Secret value - Can be generated from the “Certificates & Secrets” in the App Registration options in AAD.

SharePoint Online Lists

Three SPO lists are needed:
1. M365 Incidents
2. M365 Message Center
3. M365 Roadmap

These lists can be created on a public SPO site, and permissions can be set accordingly to reduce number of users who can read/write to these lists
Every app user will need the ability to read these lists. 
For the Message Center list, any user who needs to modify entries will need Editor permissions to the list.

The canvas app could also be built using a different data source like Dataverse or Azure SQL.
Dataverse and Azure SQL are premium data sources and require additional licensing for both makers and end users of the app

SPO lists custom columns needed:

M365 Incidents

1. Title (Single line of text)
2. Service (Single line of text)
3. Status (Single line of text)
4. UserImpact (Multiple lines of text)
5. Alert (Single line of text)
6. Class (Single line of text)
7. IncID (Single line of text)
8. Updated (Date and Time)
9. StartTime (Date and Time)
10. EndTime (Date and Time)
11. Twitter (Single line of text)
12. IsResolved (Single line of text)
13. HighImpact (Single line of text)
14. Origin (Single line of text)
15. Feature (Single line of text)
16. FeatureGroup (Single line of text)

M365 Message Center

1. Title (Single line of text)
2. Link (Single line of text)
3. MessageCenterID (Single line of text)
4. Description (Multiple lines of text - –enable enhanced rich text)
5. SolutionAffected (Single line of text)
6. ActbyDate (Date and Time)
7. Category (Single line of text)
8. ActionType (Choice) - choices: Yammer Post, Video, News Post, Change
9. Status (Choice) - choices: Completed, In Progess, New, Updated
10. Severity (Single line of text)
11. CreatedDate (Date and Time)
12. ModifiedDate (Date and Time)
13. EndDate (Date and Time)
14. Assignee (Person or Group)
15. InternalNotes (Multiple Lines of text)

M365 RoadMap:

1. Title (Single line of text)
2. RoadMapID (Single line of text)
3. Link (Single line of text)
4. Description (Multiple lines of text)
5. Status (Single line of text)
6. MoreInfoLink (Single line of text)
7. PublicRoadMapStatus (Single line of text)
8. Tags (Multiple lines of text))
9. CreatedDate (Date and Time)
10. ModifiedDate (Date and Time)
11. PublicDisclosureAvailabilityDate (Single line of text)

Power Automate Flows

Three core SPO flows are needed
1. SHD Graph API – Incidents
2. SHD Graph API – Message Center
3. Query MS Roadmap

These flows use the HTTP Action to query API endpoints. The Incidents and Message Center flows query the GRAPH API using the app registration. The MS Roadmap queries a public API, thus no authentication is needed. All three use a recurrence (scheduled trigger).

Optional flows
1. Daily M365 Open Incidents Report
2. Search M365 Incidents in Twitter
3. Any other additional notification flow or flows that can be triggered via a button in the Power App

SHD Graph API – Incidents flow breakdown

1. Step 1: Recurrence trigger – can be set to run every hour for example
2. Step 2: The HTTP Action does a GET to the following URI: https://graph.microsoft.com/v1.0/admin/serviceAnnouncement/issues
Headers: Content-type application/json
Authentication: Active Directory Oauth
Using App registration Client ID and credential type Secret
Settings need to be edited and pagination turned on and threshold set to 1000
3. Step 3: A Parse JSON action needs to added
Generate schema from HTTP action output
4. Step 4: Apply to each scoped to Value of the Parse JSON
Inside of the apply to each, a Get Items SPO action is needed
Filter using an ODATA query to look for new incidents via ID
5. Step 5: Add a condition that checks whether the item exists or not
If these don’t exist a new entry will be created using the Create Item SPO action
If it exists, the entry will be updated IF the updated dates don’t match, using the Update Item SPO action. A condition will need to be added do check if dates match

SHD Graph API – Message Center flow breakdown

1. Step 1: Recurrence trigger – can be set to run once a day
2. Step 2: The HTTP Action does a GET to the following URI: https://graph.microsoft.com/v1.0/admin/serviceAnnouncement/messages
Headers: Content-type application/json
Authentication: Active Directory Oauth
Using App registration Client ID and credential type Secret
Settings need to be edited and pagination turned on and threshold set to 1000
3. Step 3: A Parse JSON action needs to be added
Generate schema from HTTP action output
4. Step 4: Apply to each scoped to Value of the Parse JSON
Inside of the apply to each, a Get Items SPO action is needed
Filter using an ODATA query to look for new messages via ID
5. Step 5: Add two Compose actions to convert services dynamic value to a string, and then another compose to clean up the array.
6. Step 6: Add a condition that checks whether the item exists or not
If these don’t exist a new entry will be created using the Create Item SPO action
If it exists, the entry will be updated IF the updated dates don’t match, using the Update Item SPO action. A condition will need to be added do check if dates match

Query MS Roadmap flow breakdown

1. Step 1: Recurrence trigger – can be set to run once a week
2. Step 2: The HTTP Action does a GET to the following URI: https://roadmap-api.azurewebsites.net/api/features
Headers: Content-type application/json
Authentication: None
3. Step 3: A Parse JSON action needs to added
Generate schema from HTTP action output
4. Step 4: Apply to each scoped to Value of the Parse JSON
Inside of the apply to each, a Get Items SPO action is needed
Filter using an ODATA query to look for new messages via RoadMapID
5. Step 5: Add two Compose actions to convert tags dynamic value to a string, and then another compose to clean up the array.
6. Step 6: Add a condition that checks whether the item exists or not
If these don’t exist a new entry will be created using the Create Item SPO action
If it exists, the entry will be updated IF the updated dates don’t match, using the Update Item SPO action. A condition will need to be added do check if dates match

Additional/Optional flows

More flows can be created to enhance/improve the solution:

1. Daily M365 Open Incidents Report
Similar setup as the Incidents flow, but this one needs some filtering added to only look for open incidents (no end date). Then, create an HTML table of those and send it via an email to the M365 Admin Team or post to MS Teams channel.

2. Search M365 Incidents in Twitter
It is known that when there is a major incident, Microsoft will post about it in Twitter using the @MSFT365Status account. This flow runs anytime an item is modified in the Incidents lists and searches for that incident ID on any posts made by the account above and updates the SPO list item is MS has tweeted about it.

3. Any other additional notification flow or flows that can be triggered via a button in the Power App
Good to Know flow (would allow Admin team to mark Message Center items as good to know). Message Center Item Assignee notification (would update the assignee anytime a Message Center item is updated/marked as complete)

Power BI Reports - Optional

With Power BI and the data being stored in SharePoint Online lists, you can easily create custom reports for Incidents, Message Center and Roadmap which can be set to refresh on a schedule

M365 Management Canvas App

This is just an example of what the canvas app can look like. You can create an app using colors for your company, different layout, etc.

Tips/tricks and things to consider

If using SPO as the data source for this app, depending on how you decide to filter the data in galleries, keep delegation in mind.

For example, in my sample solution where I use SPO lists to store the data, I am currently using search in my formulas to filter my galleries. Search with SPO list does not support delegation. This means I have to keep my list under 2000 items, or I would need to find a different method to filter if I wanted to continue using Search, or only use functions that support delegation.

If using SPO lists, create “Archive” flows for the three main lists which can run on a schedule and delete closed/older/released items onto different archive lists which are not tied to the Power App

You can add the canvas app to MS Teams for easy access
The app can continue to be expanded, adding additional functionality to have a one stop shop M365 management solution.

Importing the M365Managemeent_1_0_0_3.Zip unmanaged solution

I’d recommend importing this “unmanaged” solution into a dev/test environment where you have the Maker Role, so you can successfully do the import.

Make sure you have the three SPO lists created accordingly as the instructions above

When the solution imports, you will need to set variables. Since this is an unmanaged solution, you will need to go to Solutions > Default Solution > Environment Variables > update each from there, setting values in the “Current value” field for each.

Once you do that, you will be able to enable the flows in the solution, but I would recommend going into edit mode for each one of these so you can see how these are built/setup and make any desired changes. 

Once the flows have been enabled, last step would be to go into edit mode for the canvas app (Microsoft 365 Administration Demo). While in edit mode, you will need to remove the existing SPO lists and add your site/lists. If you used the same names, every screen/gallery/formula will map correctly.

1.	Click View tab
2.	Data sources
3.	Remove the three SPO lists > click three dots > remove
4.	Add data source > choose your SharePoint connection
5.	Search/find the SPO site where the lists reside in
6.	Check all three lists
7.	Make sure data load and do a save

The app has several screens that are currently not being used, not built out yet. This is where you can make this app your own, remove screens, add new screens/functionality, change colors, sky is the limit. The Incidents/Advisories, Message Center and RoadMap screens are functional and already have various filters set.