# Lab: Configure Pipelines as Code with YAML

In this lab, you will define a multi-stage CI/CD pipeline as code (**YAML**) that builds a Python (Flask) calculator and deploys it to a pre-created Azure Web App, then add environment approvals to gate the deployment.

This lab covers the following exercises and tasks:

Exercise 0: Configure Lab Prerequisites<br> Task 1: Import the Python Calculator Repository<br> Task 2: Create the Azure Web App<br> Task 3: Verify the Self-Hosted Agent and Service Connection

Exercise 1: Configure CI/CD Pipelines as Code with **YAML**<br> Task 1: Understand the Multi-Stage **YAML** Pipeline<br> Task 2: Create and Run the Pipeline<br> Task 3: Review the Deployed Site

Exercise 2: Configure Environment Settings for the Pipeline<br> Task 1: Add an Environment Approval and Convert the Deploy Job<br> Task 2: Re-run the Pipeline and Approve the Deployment

## Exercise 0 – Task 1: Import the Python Calculator Repository

In this task, you will import the Python Calculator source into a new Azure Repos Git repository and set main as its default branch.

In your Azure DevOps project, go to Repos > Files.<br> Open the repository selector dropdown at the top and click *Import repository* (this creates a new repository; do not import into the default project repository).<br> In Clone **URL**, paste:<br> [https://github.com/kodekloudhub/python-calculator-webapp.git](https://github.com/kodekloudhub/python-calculator-webapp.git)

In the Name field, make sure the repository is named exactly python-calculator-webapp. Azure DevOps auto-fills this from the clone **URL** and appends .git by default; delete the trailing .git so the name matches exactly. Then click *Import*.<br> Make sure that main is set as the default branch (Repos > Branches). The repository contains python_calc.py, infra/webapp.bicep, and pipelines/python-multi-stage.yml used in this lab.

## Exercise 0 – Task 2: Create the Azure Web App

In this task, you will pre-create the Linux Python Web App that the pipeline's Deploy stage will later deploy the application to.

The Azure Web App is created up front as a prerequisite. The pipeline's Deploy stage then deploys the application to it (the Bicep template is published by the pipeline as an artifact, but the Web App itself is created here).

Use the lab terminal (already logged in to Azure **CLI**) to provision the Web App with the Bicep template from the repo:

RG=$(cat /root/code/lab-resource-group.txt 2>/dev/null || az group list --query *[0].name* -o tsv)

    az deployment group create \
        --resource-group *$RG* \
        --template-file /root/code/python-calculator-webapp/infra/webapp.bicep

This creates a Linux Python Web App named gha-pycalc-... (the name is derived from the resource group, so the pipeline can find it automatically).

![alt text](<Screenshot 2026-11-01 171901.png>)

## Exercise 0 – Task 3: Verify the Self-Hosted Agent and Service Connection

In this task, you will confirm that the lab's self-hosted agent and service connection are in place before you create any pipelines.

The lab provisions a self-hosted Linux agent kk-devops-agent (in the kk-devops-agent-pool-xxxxx pool) and uses a pre-created **ARM** service connection (pre-created for your project).

Project Settings > Agent pools > kk-devops-agent-pool-xxxxx > Agents – Confirm kk-devops-agent is Online.<br> Project Settings > Service connections – Confirm a service connection exists.

## Exercise 1 – Task 1: Understand the Multi-Stage **YAML** Pipeline

In this task, you will review the structure of the multi-stage **YAML** pipeline and the key concepts behind stages, artifacts, and deployment jobs.

Open pipelines/python-multi-stage.yml and review its structure:

    stages:
    - stage: Build
      jobs:
      - job: Build
        steps: [ install, test, archive Web.zip, publish Website + Bicep artifacts ]

    - stage: Deploy
      dependsOn: Build
      jobs:
        - deployment: Deploy
          environment: approvals      # <-- targets an environment (approvals attach here)
          strategy:
            runOnce:
              deploy:
                steps: [ download Website, find Web App, AzureRmWebAppDeployment ]

Note: The Web App was created in Exercise 0; the Deploy stage finds it with az webapp list and deploys to it. The Bicep artifact is published for reference but not deployed by this pipeline.

Why deployment and not a regular job? A normal job: cannot target an environment. To bind a stage to the approvals environment—and gain deployment history plus approvals and checks—you use a deployment job with a strategy (here runOnce). That is the key difference: swapping job: for deployment: … environment: is what turns a plain build job into a gated, environment-aware deployment.

## Exercise 1 – Task 2: Create and Run the Pipeline

In this task, you will create a pipeline from the existing Build-only **YAML**, run it, then edit the **YAML** in the pipeline editor to append a Deploy stage that delivers the app to the Web App.

The repository ships pipelines/python-multi-stage.yml containing only the Build stage. You will create a pipeline from it, then edit the **YAML** in the Azure Pipelines editor to add continuous delivery.

Create and run the build pipeline
Navigate to Pipelines > Pipelines and click *New Pipeline*.
Select *Azure Repos Git* and choose python-calculator-webapp.
Select *Existing Azure Pipelines **YAML** File*, Branch main, Path /pipelines/python-multi-stage.yml. Click Continue.
Click *Run*. Confirm the Build stage passes and publishes the Website and Bicep artifacts.
Rename the pipeline to python-multi-stage (ellipsis > Rename/Move).
Add continuous delivery (a Deploy stage)
Open the pipeline and click *Edit*.
At the end of the file (at the same indentation as - stage: Build), append a Deploy stage. Note that it is an ordinary job: for now. You will convert it to a deployment: job in Exercise 2:

    - stage: Deploy
      displayName: Deploy to an Azure Web App
      dependsOn: Build
      condition: succeeded()
      jobs:
        - job: Deploy
          pool:
            name: $(agentPool)
          steps:
        - download: current
          artifact: Website
        - task: AzureCLI@2
          displayName: Find the target Web App
          inputs:
            azureSubscription: $(azureServiceConnection)
            scriptType: bash
            scriptLocation: inlineScript
            inlineScript: |
              WEBAPP_NAME=$(az webapp list --resource-group *$(resourceGroup)* --query *[0].name* -o tsv)
              echo "##vso[task.setvariable variable=WEBAPP_NAME]$WEBAPP_NAME"
        - task: AzureRmWebAppDeployment@4
          displayName: Deploy application to Web App
          inputs:
            ConnectionType: 'AzureRM'
            azureSubscription: $(azureServiceConnection)
            appType: 'webAppLinux'
            WebAppName: '$(WEBAPP_NAME)'
            packageForLinux: '$(Pipeline.Workspace)/Website/Web.zip'
            RuntimeStack: '**PYTHON**|3.12'
            StartupCommand: 'gunicorn --bind=0.0.0.0:**8000** python_calc:app'
            DeploymentTypeLinux: 'zipDeploy'

Click *Validate and save* > *Save* (commit directly to main) to queue a new run. The run builds, then the Deploy stage finds the pre-created Web App and deploys the app to it.<br> First run – Permit access: The first time the Deploy stage runs it pauses with "This pipeline needs permission to access a resource before this run can continue* for the service connection. Click *View* > *Permit" to allow access.

![alt text](<Screenshot 2026-11-01 171953.png>)

![alt text](<Screenshot 2026-11-01 173510.png>)

![alt text](<Screenshot 2026-11-01 174927.png>)

## Exercise 1 – Task 3: Review the Deployed Site 

In this task, you will browse to the Web App and confirm the pipeline deployed the application successfully.

In the Azure Portal, open the pre-created resource group > the App Service (gha-pycalc-...).<br> Browse to [https://<app-name>.azurewebsites.net.<br>](https://<app-name>.azurewebsites.net.<br>) Confirm the Simple Calculator page loads and that you can perform a calculation.<br> This confirms the multi-stage pipeline built the app and deployed the Website artifact to the Web App.

![alt text](<Screenshot 2026-11-01 175114.png>)

## Exercise 2 – Task 1: Add an Environment Approval and Convert the Deploy Job

In this task, you will create an approvals environment with a manual approval check, then convert the Deploy stage from a plain job: into a deployment: job bound to that environment.

You protect the deployment by binding it to an environment and adding an approval. A regular job: cannot target an environment, so you also convert the Deploy job: into a deployment: job.

Create the approvals environment
Navigate to Pipelines > Environments and click *New environment* (or *Create environment*).
Name: approvals, Resource: None. Click *Create*.
On the environment page, click the ellipsis (…) > *Approvals and checks*.
Click *+* > *Approvals*, add yourself as an approver, and click *Create*.
Convert the Deploy stage to a deployment job
Open python-multi-stage > *Edit*.
Replace the entire Deploy stage with the version below. The changes: - job: Deploy becomes - deployment: Deploy, an environment: approvals line is added, and the steps move under a strategy: runOnce: deploy: block:

    - stage: Deploy
      displayName: Deploy to an Azure Web App
      dependsOn: Build
      condition: succeeded()
      jobs:
        - deployment: Deploy
          environment: approvals
          pool:
            name: $(agentPool)
          strategy:
            runOnce:
              deploy:
                steps:
                - download: current
                  artifact: Website
                - task: AzureCLI@2
                  displayName: Find the target Web App
                  inputs:
                    azureSubscription: $(azureServiceConnection)
                    scriptType: bash
                    scriptLocation: inlineScript
                    inlineScript: |
                      WEBAPP_NAME=$(az webapp list --resource-group *$(resourceGroup)* --query *[0].name* -o tsv)
                      echo "##vso[task.setvariable variable=WEBAPP_NAME]$WEBAPP_NAME"
                - task: AzureRmWebAppDeployment@4
                  displayName: Deploy application to Web App
                  inputs:
                    ConnectionType: 'AzureRM'
                    azureSubscription: $(azureServiceConnection)
                    appType: 'webAppLinux'
                    WebAppName: '$(WEBAPP_NAME)'
                    packageForLinux: '$(Pipeline.Workspace)/Website/Web.zip'
                    RuntimeStack: '**PYTHON**|3.12'
                    StartupCommand: 'gunicorn --bind=0.0.0.0:**8000** python_calc:app'
                    DeploymentTypeLinux: 'zipDeploy'

Click *Validate and save* > *Save* (commit to main).

![alt text](<Screenshot 2026-11-01 175246.png>)

![alt text](<Screenshot 2026-11-01 175425.png>)

![alt text](<Screenshot 2026-11-01 175517.png>)

## Exercise 2 – Task 2: Re-run the Pipeline and Approve the Deployment

In this task, you will re-run the pipeline and manually approve the Deploy stage at the environment gate you just configured.

Navigate to Pipelines > Pipelines and open python-multi-stage. Click *Run pipeline* > *Run*.<br> Let the Build stage complete. When the Deploy stage is reached, the run pauses for approval.<br> First run – Permit access: If prompted, Permit the pipeline to use the approvals environment.

Click *Review* > *Approve*.<br> Confirm the Deploy stage resumes and completes, redeploying the app to the Web App.<br> Because the Deploy stage is now a deployment job bound to the approvals environment, the deployment waits for your approval—the gate you add by converting job: to deployment:.

![alt text](<Screenshot 2026-11-01 180102.png>)

Review<br> Congratulations! You have successfully completed the Configure Pipelines as Code with **YAML** lab.

In this lab, you:

Pre-created an Azure Web App as a lab prerequisite<br> Authored a multi-stage **YAML** pipeline (Build → Deploy) committed alongside the application code<br> Published and consumed pipeline artifacts (Website, Bicep) between stages Deployed the application to the Web App using a deployment job bound to an environment<br> Added a manual approval to the approvals environment and gated a subsequent deployment<br> Defining pipelines as code gives you versioned, reviewable, reusable CI/CD logic with built-in release controls.