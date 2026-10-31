# Lab 01: Agile Planning and Portfolio Management with Azure Boards

In this lab, you will learn how to plan and manage an agile project using Azure Boards. Azure Boards is a service in Azure DevOps that provides rich project management capabilities, including support for agile planning, sprint planning, Kanban boards, team velocity tracking, and more.

This lab covers the following exercises and tasks:

Exercise 1: Manage Agile Project<br> Task 1: Manage Teams, Areas, and Iterations<br> Task 2: Manage Work Items<br> Task 3: Manage Sprints and Capacity<br> Task 4: Customize Kanban Boards

Exercise 2: Define Dashboards<br> Task 1: Create and Customize Dashboards<br> All configurations in this lab are performed through the Azure DevOps web interface. No command-line tools are required.

## Exercise 1: Manage Agile Project

### Task 1: Manage Teams, Areas, and Iterations

In this task, you will create a new team in the kk-devops project and then configure its iterations (sprints) and area paths. The lab environment already contains the kk-devops project with default settings.

You will work with the new team named:

kk-devops-calc-team

Step 1: Create the kk-devops-calc-team Team<br> In the kk-devops project, click Project settings (gear icon in the bottom-left corner)<br> Under the General section, select Teams<br> Click + New team<br> In the Create a team dialog:<br> Team name: kk-devops-calc-team<br> Leave Create an area path with the same name as the team checked<br> Click Create<br> In the list of teams, click kk-devops-calc-team to open its configuration page Note: Each team in Azure DevOps can have its own iterations, area paths, backlogs, and boards, allowing teams to work independently while sharing the same project.

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

Ensure all three iterations are selected for kk-devops-calc-team<br> Click Save and close<br> Note: Iteration paths define the timeboxed sprints for your team and are used by backlogs, boards, and sprint views to group work by time period.

Step 3: Configure Area Paths<br> In the Iterations and area paths page, switch to the Areas tab

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

Step 1: Create an Epic<br> In the kk-devops project, navigate to Boards → Work Items in the left sidebar Click + New Work Item and select Epic<br> Enter the following details:<br> Title: Product training<br> Assigned To: Select your user account; you can extract your username from the showcreds command<br> Area: kk-devops-<prj>\kk-devops-calc-team<br> Iteration: kk-devops-<prj>\Sprint 1<br> Click Save & Close

Step 2: Create a Feature as a Child of the Epic<br> Navigate to Boards → Backlogs → My team backlogs -> kk-devops-calc-team

In the top-right corner, use the backlog level dropdown to switch to Epics

Find the Product training Epic in the list

Click the + icon next to the Epic (or hover over it to reveal the child add button)

Type the title: Training dashboard and press Save and Close

The Feature is automatically created and linked as a child of the Epic.

Step 3: Create Product Backlog Items (PBIs)<br> Change the backlog level dropdown to Features<br> Find and select the Training dashboard Feature<br> Click + Add Product Backlog Item using the + icon on the Training dashboard feature.<br> Add the following three PBIs by typing each title and pressing Enter:<br> View new tutorials<br> See tutorials I recently viewed<br> Request new tutorials

Step 4: Transition Work Items Through States<br> Click on View new tutorials to open the work item<br> Change the State field from New to Approved<br> Click Save & Close<br> Click on See tutorials I recently viewed<br> Change the State from New to Committed<br> Click Save & Close

Step 5: Create Tasks with Work Estimates<br> Open the View new tutorials **PBI**

In the Related Work section, click + Add link → New Item → Child

Create Task 1:

Work Item Type: Task<br> Title: Add page for most recent tutorials<br> Click on Add link and then specify the following:

Remaining Work: 5<br> Activity: Development<br> Click Save & Close<br> In Related Work, click + Add link → New Item → Child again to create Task 2:

Title: Optimize data query for most recent tutorials<br> Remaining Work: 3<br> Activity: Design<br> Click Save & Close<br> Note: The Scrum work item hierarchy is: Epic → Feature → Product Backlog Item → Task. This hierarchy enables teams to trace individual tasks back to business objectives, making it easy to see how day-to-day work contributes to strategic goals.

### Task 3: Manage Sprints and Capacity

In this task, you will configure team member capacity for a sprint, add vacation days, update task work estimates, and create visual styling rules for the Taskboard.

Step 1: Switch to the kk-devops-calc-team Team Sprints View<br> Navigate to Boards → Sprints in the left sidebar<br> In the team selector dropdown, select kk-devops-calc-team<br> Verify that Sprint 1 is the active sprint<br> You should see the Taskboard with the tasks created in the previous task

Step 2: Configure Team Capacity<br> Click the Capacity tab (next to Taskboard at the top of the sprint view)<br> In the capacity table, locate your user account row<br> Under the Activity column, click and select Development<br> Under the Capacity per day column, set the value to 1 (hour per day)<br> Click Save

Step 3: Add Days Off<br> Still in the Capacity tab, locate your user row<br> Click the Days off link (displayed as 0 days) next to your name<br> In the Days off dialog:<br> Click + Day off<br> Choose a start date and end date that span 5 consecutive working days within the sprint period<br> Click OK to close the dialog<br> Click Save<br> Note: After setting days off, the available sprint capacity will be automatically recalculated. Observe how the total available hours decrease in the capacity summary.

Step 4: Update Task Remaining Work<br> Click the Taskboard tab<br> Find the task card Add page for most recent tutorials<br> Click on the task card to open the work item<br> Update the Remaining Work field from 5 to 14 hours<br> Click Save & Close<br> Note: After updating the remaining work, observe the capacity indicator. If the remaining work exceeds the team's available capacity, the indicator bar will turn red, alerting the team to an over-commitment.

Step 5: Create a Taskboard Styling Rule<br> In the Sprints view, click the Configure team settings gear icon (top-right corner)<br> In the settings panel, click the Styles tab<br> Click + Styling rule<br> Configure the rule:<br> Rule name: Development tasks<br> Card color: Select Green<br> Under Rule criteria, click + Criteria and configure:<br> Field: Activity<br> Operator: =<br> Value: Development<br> Click Save<br> Note: After saving, switch back to the Taskboard. All task cards with the Development activity type will appear with a green background, providing immediate visual identification of development work items on the board.

### Task 4: Customize Kanban Boards

In this task, you will enhance the Kanban board with tag colors, a new workflow column with **WIP** limits, split columns, a definition of done, and a swimlane for priority work.

Step 1: Add Tag Colors<br> Navigate to Boards → Boards (the Kanban board view)<br> In the team selector, ensure kk-devops-calc-team team board is displayed Click the Configure board settings gear icon (top-right corner of the board) In the settings panel, click Tag colors<br> Click Add + Tag color<br> In the Tag field, type: data<br> Click the color selector and choose Yellow<br> Click Save and close

Step 2: Add Tags to a Work Item<br> On the Kanban board, find the View new tutorials **PBI** card<br> Click on the card to open the work item<br> Find the Tags field and add the following tags:<br> data<br> ux<br> Click Save & Close<br> Observe the work item card on the board: the data tag should now appear with a yellow highlight

Step 3: Add a New Column with **WIP** Limit<br> Click the Configure board settings gear icon again<br> Click Columns in the settings panel<br> Click + Column to add a new workflow column<br> Configure the new column:<br> Name: QA Approved<br> **WIP** limit: 1<br> Enable the Split column into doing and done toggle<br> In the Definition of done text box, enter:<br> Passes **all** tests

Drag the QA Approved column to position it between Committed and Done Click Save and close<br> Note: Work in Progress (**WIP**) limits enforce workflow discipline. When the number of items in a column reaches its **WIP** limit, the column header turns red, signaling that the team must finish existing work before pulling in new items.

Step 4: Create an Expedite Swimlane<br> Click the Configure board settings gear icon<br> Click Swimlanes in the settings panel<br> Click + Swimlane<br> Configure the swimlane:<br> Name: Expedite<br> Color: Select Green<br> Click Save and close<br> Note: Swimlanes allow teams to categorize work by service-level class. The Expedite swimlane is reserved for high-priority, time-critical items that must be addressed immediately and may bypass normal **WIP** constraints.

Step 5: Verify the Board Customizations<br> Return to the Kanban board and confirm:

The data tag on the View new tutorials card appears with a yellow highlight The QA Approved column appears between Committed and Done, split into Doing and Done sub-columns<br> The Expedite swimlane appears as a dedicated horizontal lane at the top of the board<br> Dragging two items into QA Approved causes the column header to turn red (**WIP** limit exceeded)

## Exercise 2: Define Dashboards

### Task 1: Create and Customize Dashboards

In this task, you will create a team dashboard, add sprint progress widgets, create a shared work item query, generate a chart from the query results, and pin the chart to the dashboard.

Step 1: Create a New Dashboard<br> In the kk-devops project, navigate to Overview → Dashboards in the left sidebar<br> Click + New dashboard<br> In the Create a dashboard dialog, configure:<br> Name: Product training<br> Team: Select kk-devops-calc-team<br> Description: (Optional)<br> ### Click Create

Step 2: Add the Sprint Overview Widget<br> On the new Product training dashboard, click Edit (pencil icon, top-right) Click + Add a widget<br> In the widget catalog, search for Sprint Overview<br> Click Add on the Sprint Overview widget<br> If prompted to configure the widget:<br> Team: kk-devops-calc-team<br> Iteration: Sprint 1<br> Click Done Editing on the widget

Step 3: Add the Sprint Capacity Widget<br> Click + Add a widget again<br> Search for Sprint Capacity<br> Click Add on the Sprint Capacity widget<br> Configure the widget:<br> Team: kk-devops-calc-team<br> Iteration: Sprint 1<br> Click Save on the widget<br> Note: The Sprint Capacity widget requires that team capacity has been configured (which you completed in Task 3). If capacity is not set, the widget may not display meaningful data.

Step 4: Create a Shared Work Item Query<br> Navigate to Boards → Queries in the left sidebar<br> Click + New query<br> Configure the query with the following filters:<br> Row 1: Work Item Type = Task<br> Click + And to add a second condition<br> Row 2: Area Path Under kk-devops-\kk-devops-calc-team<br> Click Run query to test and verify that your tasks appear in the results<br> Click Save query (the disk icon in the toolbar)<br> In the save dialog:<br> Name: Web tasks<br> Folder: Expand and select Shared Queries (not My Queries)<br> Click OK<br> Important: Save the query under Shared Queries. Only queries saved in Shared Queries can be used as the data source for team dashboard widgets.

Step 5: Generate a Chart from the Query<br> In the Queries view, navigate to Shared Queries → open Web tasks<br> Click the Charts tab (next to the Results tab)<br> Click + New chart<br> Configure the chart:<br> Name: Web tasks - By assignment<br> Chart type: Pie<br> Group by: Assigned To<br> Aggregation: Count<br> Click Save Chart to save the chart<br>

Step 6: Add the Chart Widget to the Dashboard<br> Navigate to Overview → Dashboards → Product training<br> Click Edit<br> Click + Add a widget<br> Search for Chart for Work Items<br> Click Add on the Chart for Work Items widget<br> Click the Configure icon (pencil/gear) on the newly added widget<br> In the widget configuration:<br> Query: Select Shared Queries → Web tasks<br> Group by: Select Assigned to - Count<br> Click Save on the widget configuration<br> Click Done Editing to save and publish the dashboard<br> Verification<br> Your completed Product training dashboard should now display:

Sprint Overview – Showing sprint burndown and completion status Sprint Capacity – Showing team capacity versus remaining work<br> Web tasks - By assignment – A pie chart showing task distribution by team member

Congratulations! You have successfully completed the Azure Boards lab.

In this lab, you:

Created and configured a team with dedicated area paths and sprint iterations<br> Built a Scrum work item hierarchy (Epic → Feature → **PBI** → Task)<br> Managed sprint capacity and configured Taskboard styling rules<br> Customized a Kanban board with tag colors, **WIP** limits, split columns, and swimlanes<br> Created a team dashboard with sprint widgets and work item charts