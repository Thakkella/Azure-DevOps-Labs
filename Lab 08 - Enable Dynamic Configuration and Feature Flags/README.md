# Lab: Enable Dynamic Configuration and Feature Flags

In this lab, you will use Azure App Configuration to drive a Python (Flask) calculator app's behavior at runtime, with the app reading App Configuration via its connection string:

Dynamic configuration – Show a banner message (Calculator:Settings:Banner) and change it without redeploying.<br> Feature flags – Toggle the SalesWeekend feature on and off to show/hide a promotional banner.<br> This lab covers the following exercises and tasks:

Exercise 0: Configure Lab Prerequisites<br> Task 1: Import the Python Calculator Repository<br> Task 2: Verify the Self-Hosted Agent and Service Connection

Exercise 1: Build and Deploy the Web App<br> Task 1: Create and Run the CI Pipeline<br> Task 2: Create and Run the CD Pipeline

Exercise 2: Manage Azure App Configuration<br> Task 1: Create the App Configuration Store<br> Task 2: Connect the Web App to App Configuration<br> Task 3: Test Dynamic Configuration<br> Task 4: Test the Feature Flag

## Exercise 0 – Task 1: Import the Python Calculator Repository

In this task, you will import the Python Calculator source into a new Azure Repos Git repository and set main as its default branch.

In your Azure DevOps project, go to Repos > Files.<br> Open the repository selector dropdown at the top and click *Import repository* (this creates a new repository; do not import into the default project repository).<br> In Clone **URL**, paste:<br> [https://github.com/kodekloudhub/python-calculator-webapp.git](https://github.com/kodekloudhub/python-calculator-webapp.git)

In the Name field, make sure the repository is named exactly python-calculator-webapp. Azure DevOps auto-fills this from the clone **URL** and appends .git by default; delete the trailing .git so the name matches exactly. Then click *Import*.<br> Make sure that main is set as the default branch (Repos > Branches). The repository contains the app (python_calc.py), infra/webapp-appconfig.bicep, and pipelines/python-calc-cd-appconfig.yml used below.

## Exercise 0 – Task 2: Verify the Self-Hosted Agent and Service Connection

In this task, you will confirm that the lab's self-hosted agent, service connection, and variable group are in place before you run any pipelines.

The lab provisions a self-hosted Linux agent kk-devops-agent in the kk-devops-agent-pool-xxxxx pool, and uses a pre-created **ARM** service connection (pre-created for your project). The pipeline reads the resource group, location, and service connection from the pre-published lab-vars variable group.

Project Settings > Agent pools > kk-devops-agent-pool-xxxxx > Agents – Confirm that kk-devops-agent is Online.<br> Project Settings > Service connections – Confirm that a service connection exists.<br> Pipelines > Library – Confirm that the lab-vars variable group exists.

## Exercise 1 – Task 1: Create and Run the CI Pipeline

In this task, you will create and run the CI pipeline that tests the app and publishes the deployable Website artifact.

Build and test the calculator with the CI pipeline pipelines/python-calc-ci.yml. It runs the unit tests and publishes the application as the Website build artifact, which the CD pipeline deploys.

Go to Pipelines > Pipelines > New pipeline.<br> Select Azure Repos Git > the python-calculator-webapp repository.<br> Select Existing Azure Pipelines **YAML** file, choose /pipelines/python-calc-ci.yml, and Continue.<br> Click Run. Authorize the lab-vars variable group and the service connection if prompted (View > Permit). The pipeline is created at this point and takes a default name from the repository (python-calculator-webapp).<br> Once the run has started, rename the pipeline to python-calc-ci: go to Pipelines, select the pipeline, click … (More actions) > Rename/move, enter python-calc-ci, and Save.<br> When it finishes, the run's Website artifact contains Web.zip.

## Exercise 1 – Task 2: Create and Run the CD Pipeline

In this task, you will create and run the CD pipeline that provisions the Web App and deploys the CI artifact to it.

Deploy the app with the CD pipeline pipelines/python-calc-cd-appconfig.yml. It triggers on the python-calc-ci pipeline, provisions the Web App via infra/webapp-appconfig.bicep, and deploys the CI Website artifact.

Pipelines > New pipeline > Azure Repos Git > python-calculator-webapp > Existing Azure Pipelines **YAML** file, choose /pipelines/python-calc-cd-appconfig.yml, and Continue.<br> Click Run (the first run is manual; afterward, it runs automatically whenever CI completes). The run pauses with "This pipeline needs permission to access a resource before this run can continue". Click View, then Permit for the lab-vars variable group and the service connection. The run stays queued until you do this.<br> When it finishes, a Web App named ff-pycalc-... is running in the pre-created resource group. You need this exact name in Exercise 2, so note it now:

RG=$(cat /root/code/lab-resource-group.txt 2>/dev/null || az group list --query *[0].name* -o tsv)<br> az webapp list --resource-group $RG --query *[?starts_with(name,'ff-pycalc')].name* -o tsv

Note: The first request after a deployment can take up to a minute while the Web App cold-starts. Wait ~30-60 seconds after the pipeline finishes before browsing the app or checking the validation.

![alt text](<Screenshot 2026-11-01 190044.png>)

![alt text](<Screenshot 2026-11-01 190938.png>)

![alt text](<Screenshot 2026-11-01 202446.png>)

## Exercise 2 – Task 1: Create the App Configuration Store

In this task, you will create the Azure App Configuration store that will hold the app's settings and feature flags.

Create the store (Azure Portal)<br> In the Azure Portal, click Create a resource, search for App Configuration, and click Create.<br> On the Basics tab, enter:<br> Subscription: Use the default subscription associated with your Azure account.<br> Resource group: The pre-created resource group<br> Resource name: appcs-az400-config<br> Location: West US<br> Pricing tier: Free<br> Click Review + create, then Create.<br> When deployment finishes, click Go to resource.<br> Optional – Create the store with the **CLI** instead<br> RG=$(cat /root/code/lab-resource-group.txt 2>/dev/null || az group list --query *[0].name* -o tsv)<br> APPCS_NAME=*appcs-az400-config*<br> az appconfig create --name *$APPCS_NAME* --resource-group $RG --location westus --sku Free<br> echo *App Configuration store: $APPCS_NAME*

![alt text](<Screenshot 2026-11-01 202804.png>)

## Exercise 2 – Task 2: Connect the Web App to App Configuration

In this task, you will give the Web App the App Configuration connection string so it can read settings and feature flags at runtime.

App Configuration access keys need no role assignment.

Copy the connection string (Azure Portal)<br> Open your App Configuration store.<br> Under Settings, select Access settings (labelled Access keys in older portal versions).<br> On the Read-only keys tab, copy the Primary Connection string.<br> Set it on the Web App<br> Open the Web App (ff-pycalc-...).<br> Under Settings, select Environment variables.<br> Click + Add and enter:<br> Name: AZURE_APPCONFIG_CONNECTION_STRING<br> Value: the connection string you copied<br> Click Apply, then Confirm. The app restarts to pick up the new setting.<br> Optional – Set it with the **CLI** instead<br> RG=$(cat /root/code/lab-resource-group.txt 2>/dev/null || az group list --query *[0].name* -o tsv)<br> **WEBAPP**=$(az webapp list --resource-group $RG --query *[?starts_with(name,'ff-pycalc')].name | [0]* -o tsv)<br> APPCS_NAME=$(az appconfig list --resource-group $RG --query *[0].name* -o tsv)<br> **CONN**=$(az appconfig credential list --name *$APPCS_NAME* --resource-group $RG --query *[?name=='Primary'].connectionString | [0]* -o tsv)<br> az webapp config appsettings set --resource-group $RG --name "$

![alt text](<Screenshot 2026-11-01 203613.png>)

![alt text](<Screenshot 2026-11-01 203820.png>)

## Exercise 2 – Task 3: Test Dynamic Configuration

In this task, you will add a banner key-value in App Configuration and watch it appear on the running app without redeploying.

The app watches a sentinel key, so you bump sentinel to push refreshes.

Add the key-values (Azure Portal)<br> Open your App Configuration store.<br> Under Configuration Management, select Configuration explorer (this section is labelled Operations in older portal versions).<br> Click Create > Key-value, enter the following, and click Apply:<br> Key: Calculator:Settings:Banner<br> Value: Welcome to the KodeKloud Calculator!<br> Click Create > Key-value again, enter the following, and click Apply:<br> Key: sentinel<br> Value: 1<br> Optional – Add them with the **CLI** instead<br> RG=$(cat /root/code/lab-resource-group.txt 2>/dev/null || az group list --query *[0].name* -o tsv)<br> APPCS_NAME=$(az appconfig list --resource-group $RG --query *[0].name* -o tsv)<br> az appconfig kv set --name *$APPCS_NAME* --key *Calculator:Settings:Banner* --value *Welcome to the KodeKloud Calculator!* --yes<br> az appconfig kv set --name *$APPCS_NAME* --key *sentinel* --value *1* --yes

Refresh the web page after ~10-15 seconds: the banner appears. Change the banner value and bump sentinel again to see it update live.

![alt text](<Screenshot 2026-11-01 204141.png>)

![alt text](<Screenshot 2026-11-01 204311.png>)

![alt text](<Screenshot 2026-11-01 204529.png>)

## Exercise 2 – Task 4: Test the Feature Flag

In this task, you will create and toggle the SalesWeekend feature flag and watch the promotional banner appear and disappear on the running app.

Create and enable the flag (Azure Portal)<br> Open your App Configuration store.<br> Under Feature Management, select Feature manager (this section is labelled Operations in older portal versions).<br> Click Create, set Feature flag name to SalesWeekend, and check Enable feature flag.<br> Click Apply.<br> Go to Configuration Management > Configuration explorer, edit the sentinel key, and change its value (e.g., to 2) to push the refresh.<br> Optional – Do it with the **CLI** instead<br> RG=$(cat /root/code/lab-resource-group.txt 2>/dev/null || az group list --query *[0].name* -o tsv)<br> APPCS_NAME=$(az appconfig list --resource-group $RG --query *[0].name* -o tsv)<br> az appconfig feature set    --name *$APPCS_NAME* --feature SalesWeekend --yes<br> az appconfig feature enable --name *$APPCS_NAME* --feature SalesWeekend --yes<br> az appconfig kv set --name *$APPCS_NAME* --key *sentinel* --value *2* --yes

Refresh the page after ~10-15 seconds: the **ALL** **CALCULATIONS** **ARE** **FREE** **THIS** **WEEKEND**! banner appears. Disable the flag (and bump sentinel), and it disappears, all without a redeploy.

![alt text](<Screenshot 2026-11-01 205133.png>)

![alt text](<Screenshot 2026-11-01 205307.png>)

![alt text](<Screenshot 2026-11-01 205412.png>)

![alt text](<Screenshot 2026-11-01 205458.png>)

![alt text](<Screenshot 2026-11-01 205637.png>)

![alt text](<Screenshot 2026-11-01 205742.png>)

Review<br> Congratulations! You have completed the Enable Dynamic Configuration and Feature Flags lab.

In this lab, you:

Built and deployed the calculator with a CI/CD pipeline (separate CI and CD pipelines)<br> Created an Azure App Configuration store and connected the Web App to it with its connection string<br> Added a dynamic configuration banner and a SalesWeekend feature flag, and saw the running app update without a redeploy<br> Centralized configuration and feature flags let you change application behavior and roll features in or out at runtime, decoupling deployment from release.