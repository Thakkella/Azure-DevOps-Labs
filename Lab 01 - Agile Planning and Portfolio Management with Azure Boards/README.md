# Lab 01: Agile Planning and Portfolio Management with Azure Boards

In this lab, you will learn how to plan and manage an agile project using Azure Boards. Azure Boards is a service in Azure DevOps that provides rich project management capabilities, including support for agile planning, sprint planning, Kanban boards, team velocity tracking, and more.

This lab covers the following exercises and tasks:

Exercise 1: Manage Agile Project Task 1: Manage Teams, Areas, and Iterations Task 2: Manage Work Items Task 3: Manage Sprints and Capacity Task 4: Customize Kanban Boards

Exercise 2: Define Dashboards Task 1: Create and Customize Dashboards All configurations in this lab are performed through the Azure DevOps web interface. No command-line tools are required.

## Exercise 1: Manage Agile Project - Task 1: Manage Teams, Areas, and Iterations In this task, you will create a new team in the kk-devops project and then configure its iterations (sprints) and area paths. The lab environment already contains the kk-devops project with default settings.

You will work with the new team named:

kk-devops-calc-team

Step 1: Create the kk-devops-calc-team Team In the kk-devops project, click Project settings (gear icon in the bottom-left corner) Under the General section, select Teams Click + New team In the Create a team dialog: Team name: kk-devops-calc-team Leave Create an area path with the same name as the team checked ### Click Create In the list of teams, click kk-devops-calc-team to open its configuration page Note: Each team in Azure DevOps can have its own iterations, area paths, backlogs, and boards, allowing teams to work independently while sharing the same project.

Step 2: Configure Team Iterations (Sprints)
From the kk-devops-calc-team configuration page, click Iterations and area paths
In general, check the epics option in the backlog
Select the Iterations tab
Click Select iteration(s)
Click + New Iteration and create the following three sprints at the project level:
Iteration Name	Start Date	End Date
Sprint 1	Today	3 weeks from today
Sprint 2	Day after Sprint 1 ends	3 weeks after Sprint 2 start
Sprint 3	Day after Sprint 2 ends	3 weeks after Sprint 3 start
First select the required iteration, then edit and add the start and end dates.

Ensure all three iterations are selected for kk-devops-calc-team Click Save and close Note: Iteration paths define the timeboxed sprints for your team and are used by backlogs, boards, and sprint views to group work by time period.

Step 3: Configure Area Paths In the Iterations and area paths page, switch to the Areas tab

Click Select area(s)

If not already selected, select the area path that matches the team:

kk-devops<prj>\kk-devops-calc-team

### Click Save

In the areas list, click the … (ellipsis) next to the kk-devops\\kk-devops-calc-team area path

Select Include sub areas

### Click Save

Note: Enabling Include sub areas ensures that the kk-devops-calc-team can view and manage work items in any child areas under its main area path, preventing work items from being hidden from the team backlog or boards.

### Task 2: Manage Work Items

In this task, you will create a hierarchy of Scrum work items—an Epic, a Feature, Product Backlog Items (PBIs), and Tasks—and transition them through different workflow states.

Step 1: Create an Epic In the kk-devops project, navigate to Boards → Work Items in the left sidebar Click + New Work Item and select Epic Enter the following details: Title: Product training Assigned To: Select your user account; you can extract your username from the showcreds command Area: kk-devops-<prj>\kk-devops-calc-team Iteration: kk-devops-<prj>\Sprint 1 Click Save & Close

Step 2: Create a Feature as a Child of the Epic Navigate to Boards → Backlogs → My team backlogs -> kk-devops-calc-team

In the top-right corner, use the backlog level dropdown to switch to Epics

Find the Product training Epic in the list

Click the + icon next to the Epic (or hover over it to reveal the child add button)

Type the title: Training dashboard and press Save and Close

The Feature is automatically created and linked as a child of the Epic.

Step 3: Create Product Backlog Items (PBIs) Change the backlog level dropdown to Features Find and select the Training dashboard Feature Click + Add Product Backlog Item using the + icon on the Training dashboard feature. Add the following three PBIs by typing each title and pressing Enter: View new tutorials See tutorials I recently viewed Request new tutorials

Step 4: Transition Work Items Through States Click on View new tutorials to open the work item Change the State field from New to Approved Click Save & Close Click on See tutorials I recently viewed Change the State from New to Committed Click Save & Close

Step 5: Create Tasks with Work Estimates Open the View new tutorials **PBI**

In the Related Work section, click + Add link → New Item → Child

Create Task 1:

Work Item Type: Task Title: Add page for most recent tutorials Click on Add link and then specify the following:

Remaining Work: 5 Activity: Development Click Save & Close In Related Work, click + Add link → New Item → Child again to create Task 2:

Title: Optimize data query for most recent tutorials Remaining Work: 3 Activity: Design Click Save & Close Note: The Scrum work item hierarchy is: Epic → Feature → Product Backlog Item → Task. This hierarchy enables teams to trace individual tasks back to business objectives, making it easy to see how day-to-day work contributes to strategic goals.

### Task 3: Manage Sprints and Capacity

In this task, you will configure team member capacity for a sprint, add vacation days, update task work estimates, and create visual styling rules for the Taskboard.

Step 1: Switch to the kk-devops-calc-team Team Sprints View Navigate to Boards → Sprints in the left sidebar In the team selector dropdown, select kk-devops-calc-team Verify that Sprint 1 is the active sprint You should see the Taskboard with the tasks created in the previous task

Step 2: Configure Team Capacity Click the Capacity tab (next to Taskboard at the top of the sprint view) In the capacity table, locate your user account row Under the Activity column, click and select Development Under the Capacity per day column, set the value to 1 (hour per day) ### Click Save

Step 3: Add Days Off Still in the Capacity tab, locate your user row Click the Days off link (displayed as 0 days) next to your name In the Days off dialog: Click + Day off Choose a start date and end date that span 5 consecutive working days within the sprint period Click OK to close the dialog ### Click Save Note: After setting days off, the available sprint capacity will be automatically recalculated. Observe how the total available hours decrease in the capacity summary.

Step 4: Update Task Remaining Work Click the Taskboard tab Find the task card Add page for most recent tutorials Click on the task card to open the work item Update the Remaining Work field from 5 to 14 hours Click Save & Close Note: After updating the remaining work, observe the capacity indicator. If the remaining work exceeds the team's available capacity, the indicator bar will turn red, alerting the team to an over-commitment.

Step 5: Create a Taskboard Styling Rule In the Sprints view, click the Configure team settings gear icon (top-right corner) In the settings panel, click the Styles tab Click + Styling rule Configure the rule: Rule name: Development tasks Card color: Select Green Under Rule criteria, click + Criteria and configure: Field: Activity Operator: = Value: Development ### Click Save Note: After saving, switch back to the Taskboard. All task cards with the Development activity type will appear with a green background, providing immediate visual identification of development work items on the board.

### Task 4: Customize Kanban Boards

In this task, you will enhance the Kanban board with tag colors, a new workflow column with **WIP** limits, split columns, a definition of done, and a swimlane for priority work.

Step 1: Add Tag Colors Navigate to Boards → Boards (the Kanban board view) In the team selector, ensure kk-devops-calc-team team board is displayed Click the Configure board settings gear icon (top-right corner of the board) In the settings panel, click Tag colors Click Add + Tag color In the Tag field, type: data Click the color selector and choose Yellow Click Save and close

Step 2: Add Tags to a Work Item On the Kanban board, find the View new tutorials **PBI** card Click on the card to open the work item Find the Tags field and add the following tags: data ux Click Save & Close Observe the work item card on the board: the data tag should now appear with a yellow highlight

Step 3: Add a New Column with **WIP** Limit Click the Configure board settings gear icon again Click Columns in the settings panel Click + Column to add a new workflow column Configure the new column: Name: QA Approved **WIP** limit: 1 Enable the Split column into doing and done toggle In the Definition of done text box, enter: Passes **all** tests

Drag the QA Approved column to position it between Committed and Done Click Save and close Note: Work in Progress (**WIP**) limits enforce workflow discipline. When the number of items in a column reaches its **WIP** limit, the column header turns red, signaling that the team must finish existing work before pulling in new items.

Step 4: Create an Expedite Swimlane Click the Configure board settings gear icon Click Swimlanes in the settings panel Click + Swimlane Configure the swimlane: Name: Expedite Color: Select Green Click Save and close Note: Swimlanes allow teams to categorize work by service-level class. The Expedite swimlane is reserved for high-priority, time-critical items that must be addressed immediately and may bypass normal **WIP** constraints.

Step 5: Verify the Board Customizations Return to the Kanban board and confirm:

The data tag on the View new tutorials card appears with a yellow highlight The QA Approved column appears between Committed and Done, split into Doing and Done sub-columns The Expedite swimlane appears as a dedicated horizontal lane at the top of the board Dragging two items into QA Approved causes the column header to turn red (**WIP** limit exceeded)

## Exercise 2: Define Dashboards - Task 1: Create and Customize Dashboards

In this task, you will create a team dashboard, add sprint progress widgets, create a shared work item query, generate a chart from the query results, and pin the chart to the dashboard.

Step 1: Create a New Dashboard In the kk-devops project, navigate to Overview → Dashboards in the left sidebar Click + New dashboard In the Create a dashboard dialog, configure: Name: Product training Team: Select kk-devops-calc-team Description: (Optional) ### Click Create

Step 2: Add the Sprint Overview Widget On the new Product training dashboard, click Edit (pencil icon, top-right) Click + Add a widget In the widget catalog, search for Sprint Overview Click Add on the Sprint Overview widget If prompted to configure the widget: Team: kk-devops-calc-team Iteration: Sprint 1 Click Done Editing on the widget

Step 3: Add the Sprint Capacity Widget Click + Add a widget again Search for Sprint Capacity Click Add on the Sprint Capacity widget Configure the widget: Team: kk-devops-calc-team Iteration: Sprint 1 Click Save on the widget Note: The Sprint Capacity widget requires that team capacity has been configured (which you completed in Task 3). If capacity is not set, the widget may not display meaningful data.

Step 4: Create a Shared Work Item Query Navigate to Boards → Queries in the left sidebar Click + New query Configure the query with the following filters: Row 1: Work Item Type = Task Click + And to add a second condition Row 2: Area Path Under kk-devops-\kk-devops-calc-team Click Run query to test and verify that your tasks appear in the results Click Save query (the disk icon in the toolbar) In the save dialog: Name: Web tasks Folder: Expand and select Shared Queries (not My Queries) Click OK Important: Save the query under Shared Queries. Only queries saved in Shared Queries can be used as the data source for team dashboard widgets.

Step 5: Generate a Chart from the Query In the Queries view, navigate to Shared Queries → open Web tasks Click the Charts tab (next to the Results tab) Click + New chart Configure the chart: Name: Web tasks - By assignment Chart type: Pie Group by: Assigned To Aggregation: Count Click Save Chart to save the chart

Step 6: Add the Chart Widget to the Dashboard Navigate to Overview → Dashboards → Product training ### Click Edit Click + Add a widget Search for Chart for Work Items Click Add on the Chart for Work Items widget Click the Configure icon (pencil/gear) on the newly added widget In the widget configuration: Query: Select Shared Queries → Web tasks Group by: Select Assigned to - Count Click Save on the widget configuration Click Done Editing to save and publish the dashboard Verification Your completed Product training dashboard should now display:

Sprint Overview – Showing sprint burndown and completion status Sprint Capacity – Showing team capacity versus remaining work Web tasks - By assignment – A pie chart showing task distribution by team member

Congratulations! You have successfully completed the Azure Boards lab.

In this lab, you:

Created and configured a team with dedicated area paths and sprint iterations Built a Scrum work item hierarchy (Epic → Feature → **PBI** → Task) Managed sprint capacity and configured Taskboard styling rules Customized a Kanban board with tag colors, **WIP** limits, split columns, and swimlanes Created a team dashboard with sprint widgets and work item charts