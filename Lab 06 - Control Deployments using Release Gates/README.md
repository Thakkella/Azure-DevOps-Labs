# Lab: Control Deployments using Release Gates

In this lab, you will gate a deployment from a DevTest environment to a Production environment using a classic Release pipeline and an automated Azure Monitor alerts release gate. You build a CI pipeline, create two web apps, wire up Application Insights with a Failed Requests alert, then add a post-deployment gate that blocks promotion to Production while the alert is firing.

This lab covers the following exercises and tasks:

Exercise 0: Configure Lab Prerequisites<br> Task 1: Import the Python Calculator Repository<br> Task 2: Configure the CI Build Pipeline<br> Task 3: Verify the Self-Hosted Agent and Service Connection

Exercise 1: Create Azure Resources<br> Task 1: Create the DevTest and Production Web Apps<br> Task 2: Configure Application Insights and an Alert Rule

Exercise 2: Configure the Release Pipeline<br> Task 1: Create a Classic Release Pipeline (DevTest -> Production)

Exercise 3: Configure Release Gates<br> Task 1: Configure a Pre-Deployment Approval<br> Task 2: Configure a Post-Deployment Azure Monitor Alerts Gate

Exercise 4: Test the Release Gate<br> Task 1: Generate Failures and Observe the Gate Block then Pass

## Exercise 0 – Task 1: Import the Python Calculator Repository

In this task, you will import the Python Calculator source into a new Azure Repos Git repository and set main as its default branch.

In your Azure DevOps project, go to Repos > Files. Open the repository selector dropdown at the top and click *Import repository* (this creates a new repository; do not import into the default project repository).<br> In Clone **URL**, paste: [https://github.com/kodekloudhub/python-calculator-webapp.git](https://github.com/kodekloudhub/python-calculator-webapp.git)

In the Name field, make sure the repository is named exactly python-calculator-webapp. Azure DevOps auto-fills this from the clone **URL** and appends .git by default; delete the trailing .git so the name matches exactly. Then click *Import*. Make sure that main is set as the default branch (Repos > Branches).

## Exercise 0 – Task 2: Configure the CI Build Pipeline

In this task, you will create the CI build pipeline whose artifact the release pipeline will later deploy.

The release pipeline deploys the artifact produced by a CI build. Create that build pipeline from pipelines/python-calc-ci.yml (it tests the app and publishes Web.zip as the Website artifact).

Navigate to Pipelines > Pipelines > New Pipeline. Select Azure Repos Git > python-calculator-webapp > Existing Azure Pipelines **YAML** File.<br> Branch main, Path /pipelines/python-calc-ci.yml. Click Continue > Run. Confirm the build succeeds and publishes the Website artifact (containing Web.zip). Rename the pipeline to python-calc-ci.

## Exercise 0 – Task 3: Verify the Self-Hosted Agent and Service Connection

In this task, you will confirm that the lab's self-hosted agent and service connection are in place before you build any pipelines.

The lab provisions a self-hosted Linux agent kk-devops-agent (in the kk-devops-agent-pool-xxxxx pool) and a pre-created **ARM** service connection (pre-created for your project).

Project Settings > Agent pools > kk-devops-agent-pool-xxxxx > Agents – Confirm that kk-devops-agent is Online.<br> Project Settings > Service connections – Confirm that a service connection exists.

## Exercise 1 – Task 1: Create the DevTest and Production Web Apps

In this task, you will create the two Linux Python web apps—DevTest and Production—that the release pipeline will deploy to.

Using the lab terminal, create an App Service plan and two Linux Python web apps in the pre-created resource group: one for DevTest and one for Production.

RG=$(cat /root/code/lab-resource-group.txt 2>/dev/null || az group list --query *[0].name* -o tsv)<br> **LOC**=$(az group show -n *$RG* --query location -o tsv)

az appservice plan create -g *$RG* -n rgates-plan --sku B1 --is-linux --location *$**LOC***

az webapp create -g *$RG* -p rgates-plan -n ***RGATES**-az400-DevTest* --runtime ***PYTHON**:3.12*<br> az webapp create -g *$RG* -p rgates-plan -n ***RGATES**-az400-Prod*    --runtime ***PYTHON**:3.12*<br> echo "DevTest: **RGATES**-az400-DevTest   |   Prod: **RGATES**-az400-Prod"   # note these names

Note the two web app names; you select them in the release pipeline's deploy tasks.

![alt text](<Screenshot 2026-10-31 161840.png>)

## Exercise 1 – Task 2: Configure Application Insights and an Alert Rule

In this task, you will create a Log Analytics workspace, enable Application Insights on the DevTest web app, and create the Failed Requests alert that the release gate will query.

Create the Log Analytics workspace<br> Application Insights is workspace-based, so it requires a Log Analytics workspace.

In the Azure Portal, use the top search bar to search for *Log Analytics workspaces* and select it.<br> Click *+ Create*.<br> Subscription: Keep the lab subscription selected.<br> Resource group: Select the pre-created lab resource group (its name begins with kml_rg_dev-).<br> Name: rgates-la-ws<br> Region: Select East US.<br> Click *Review + Create*, then *Create*, and wait for the deployment to finish. Enable Application Insights<br> Navigate to the DevTest web app (**RGATES**-az400-DevTest).<br> Go to Monitoring > Application Insights and click *Turn on Application Insights*.<br> Choose *Create new resource* and keep the suggested name.<br> For Log Analytics Workspace, select the rgates-la-ws workspace you just created; confirm that the resource group shown is the pre-created lab resource group.<br> Click *Apply*, then *Yes* to confirm.<br> Create the Failed Requests alert<br> In the Application Insights resource you just created, under Monitoring, select Alerts.<br> Click + Create, then Alert rule. The Scope is already set to this Application Insights resource.<br> Select the Condition tab. In the Select a signal pane, search for and select Failed Requests; choose the entry whose Signal type is Metric (not a log search).<br> Under Alert logic, set:<br> Threshold: Static<br> Aggregation type: Count<br> Operator: Greater than<br> Threshold value: 0<br> Leave the default Check every and Lookback period values.<br> Select the Details tab and set:<br> Severity: 2 - Warning<br> Alert rule name: RGATESDevTest_FailedRequests<br> Leave Automatically resolve alerts unchecked.<br> Click Review + create, then Create.

![alt text](<Screenshot 2026-10-31 162938.png>)

![alt text](<Screenshot 2026-10-31 163234.png>)

![alt text](<Screenshot 2026-10-31 164403.png>)

![alt text](<Screenshot 2026-10-31 164507.png>)

## Exercise 2 – Task 1: Create a Classic Release Pipeline (DevTest -> Production)

In this task, you will build a classic Release pipeline with two stages that deploy the CI artifact to the DevTest and Production web apps.

Build a classic Release pipeline with two stages that deploy the CI artifact to the two web apps.

Navigate to Pipelines > Releases and click *New pipeline*.<br> Select the *Azure App Service deployment* template and click *Apply*.<br> Rename Stage 1 to DevTest; rename the pipeline to python-calc-cd.<br> Hover the DevTest stage and click *Clone* to create a Production stage.<br> On the Pipeline tab, click *Add an artifact* > source python-calc-ci > Add.<br> Click the lightning bolt on the artifact and enable the Continuous deployment trigger.<br> Configure the DevTest stage<br> Click *1 job, 1 task* on DevTest.<br> Azure subscription: Select the pre-created service connection (Authorize if prompted).<br> App type: Web App on Linux. App Service name: Select your **RGATES**-az400-DevTest app.<br> Select the *Deploy Azure App Service* task:<br> Package or folder: $(System.DefaultWorkingDirectory)/**/Web.zip<br> Runtime Stack: **PYTHON**|3.12<br> Startup command: gunicorn --bind=0.0.0.0:**8000** python_calc:app<br> Application and Configuration Settings > App settings: -SCM_DO_BUILD_DURING_DEPLOYMENT true<br> Configure the Production stage<br> Repeat steps 7-10 on Production, selecting the **RGATES**-az400-Prod app. Click *Save* > *OK*.<br> Click *Create release* > *Create*, then watch DevTest deploy, followed by Production.<br> Browse both web app URLs and confirm the Simple Calculator page loads. First run – Permit access: If prompted, Permit the release to use the service connection.

![alt text](<Screenshot 2026-10-31 172929.png>)

![alt text](<Screenshot 2026-10-31 173207.png>)

![alt text](<Screenshot 2026-10-31 173343.png>)

![alt text](<Screenshot 2026-10-31 173618.png>)

![alt text](<Screenshot 2026-10-31 173747.png>)

![alt text](<Screenshot 2026-10-31 174635.png>)

![alt text](<Screenshot 2026-10-31 175005.png>)

![alt text](<Screenshot 2026-10-31 175640.png>)

![alt text](<Screenshot 2026-10-31 175847.png>)

![alt text](<Screenshot 2026-10-31 175904.png>)

![alt text](<Screenshot 2026-10-31 175946.png>)

![alt text](<Screenshot 2026-10-31 180006.png>)

![alt text](<Screenshot 2026-10-31 180328.png>)

![alt text](<Screenshot 2026-10-31 180842.png>)

## Exercise 3 – Task 1: Configure a Pre-Deployment Approval

In this task, you will add a manual pre-deployment approval to the DevTest stage so the deployment pauses until you approve it.

Open the python-calc-cd release pipeline and click *Edit*.<br> On the left edge of the DevTest stage, click the *Pre-deployment conditions* oval.<br> Toggle *Pre-deployment approvals* to Enabled.<br> In Approvers, add yourself. Save and close.<br> Click *Create release*; when prompted, Approve the DevTest stage.

![alt text](<Screenshot 2026-10-31 181127.png>)

![alt text](<Screenshot 2026-10-31 181333.png>)

## Exercise 3 – Task 2: Configure a Post-Deployment Azure Monitor Alerts Gate

In this task, you will add the automated release gate that queries Azure Monitor alerts after DevTest deploys and blocks promotion to Production while an alert is firing.

This is the release gate: after DevTest deploys, the pipeline queries Azure Monitor and only promotes to Production if no alert is firing.

In the python-calc-cd pipeline (Edit), click the *Post-deployment conditions* oval on the right edge of the DevTest stage.<br> Toggle *Gates* to Enabled.<br> Click *+ Add* > *Query Azure Monitor alerts*.<br> Azure subscription: It's your service connection; Resource group: It's the pre-created resource group.<br> Expand Advanced and set the filters: Severity Sev0-Sev4, Time Range Past Hour, Alert State Acknowledged/New, Monitor Condition Fired.<br> Expand Evaluation options:<br> Time between re-evaluation of gates: 5 Minutes<br> Timeout after which gates fail: 8 Minutes<br> Click *Save*.

![alt text](<Screenshot 2026-10-31 181940.png>)

![alt text](<Screenshot 2026-10-31 182002.png>)

![alt text](<Screenshot 2026-10-31 182220.png>)

![alt text](<Screenshot 2026-10-31 182929.png>)

## Exercise 4 – Task 1: Generate Failures and Observe the Gate Block then Pass

In this task, you will generate failed requests to fire the alert, then watch the release gate block the Production deployment and later pass once the alert clears.

Now, prove that the gate works: create failed requests so the alert fires, and then watch the gate hold the Production deployment.

Generate failed requests<br> Browse the DevTest web app **URL**, then append /discount (a non-existent path) and refresh several times; each returns an error, recorded by Application Insights as a failed request.<br> In the Application Insights resource > Alerts, confirm the RGATESDevTest_FailedRequests alert fires (Severity 2).<br> Observe the gate<br> In Pipelines > Releases > python-calc-cd, click *Create release*.<br> Approve the DevTest pre-deployment approval; DevTest deploys.<br> The Post-deployment gates begin evaluating; click *Evaluation gates* to view the Query Azure Monitor alerts status. While the alert is firing, the gate is failed and Production is blocked.<br> The gate re-evaluates every 5 minutes. Once the alert is no longer active (or the 8-minute timeout passes and you re-run), the gate passes and the Production stage deploys.<br> Browse the Production web app **URL** and confirm the Simple Calculator page loads.<br> Congratulations! You used a release gate to automatically block promotion to Production based on live monitoring signals.

![alt text](<Screenshot 2026-10-31 183224.png>)

![alt text](<Screenshot 2026-10-31 184304.png>)

![alt text](<Screenshot 2026-10-31 184643.png>)

![alt text](<Screenshot 2026-10-31 185345.png>)

![alt text](<Screenshot 2026-10-31 185444.png>)

![alt text](<Screenshot 2026-10-31 185537.png>)

![alt text](<Screenshot 2026-10-31 185659.png>)

Review<br> Congratulations! You have completed the Control Deployments using Release Gates lab.

In this lab, you:

Built a CI pipeline that publishes a deployable artifact<br> Created DevTest and Production web apps and wired up Application Insights with a Failed Requests alert<br> Built a classic Release pipeline that deploys to DevTest then Production<br> Added a pre-deployment approval and a post-deployment Query Azure Monitor alerts gate<br> Demonstrated the gate blocking promotion to Production while an alert was firing, then passing once it cleared<br> Release gates let you automate quality/health checks, combining human approvals with monitoring signals, before promoting changes to production.