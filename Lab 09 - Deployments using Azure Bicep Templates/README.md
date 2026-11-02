# Lab: Deployments using Azure Bicep Templates

In this lab, you will define infrastructure as Bicep, modularize it, and deploy it through an Azure Pipeline, with linting and a what-if preview before any change is applied.

This lab covers the following exercises and tasks:

Exercise 0: Configure Lab Prerequisites<br> Task 1: Import the Python Calculator Repository<br> Task 2: Verify the Self-Hosted Agent and Service Connection

Exercise 1: Create and Modularize Bicep Templates<br> Task 1: Review main.bicep and Modify a Parameter<br> Task 2: Create the Storage Module<br> Task 3: Integrate the Module into main.bicep

Exercise 2: Deploy with a Pipeline<br> Task 1: Deploy with the Pipeline and Verify

## Exercise 0 – Task 1: Import the Python Calculator Repository

In this task, you will import the Python Calculator source into a new Azure Repos Git repository and set main as its default branch.

In your Azure DevOps project, go to Repos > Files.<br> Open the repository selector dropdown at the top and click *Import repository* (this creates a new repository; do not import into the default project repository).<br> In Clone **URL**, paste:<br> [https://github.com/kodekloudhub/python-calculator-webapp.git](https://github.com/kodekloudhub/python-calculator-webapp.git)

In the Name field, make sure the repository is named exactly python-calculator-webapp. Azure DevOps auto-fills this from the clone **URL** and appends .git by default; delete the trailing .git so the name matches exactly. Then click *Import*.<br> Make sure that main is set as the default branch (Repos > Branches). The repository contains infra/main.bicep, infra/modules/webapp.bicep, and pipelines/bicep-deploy.yml used in this lab.

## Exercise 0 – Task 2: Verify the Self-Hosted Agent and Service Connection

In this task, you will confirm that the lab's self-hosted agent, service connection, and variable group are in place before you run the deployment pipeline.

The lab provisions a self-hosted Linux agent kk-devops-agent in the kk-devops-agent-pool-xxxxx pool, and uses a pre-created **ARM** service connection (pre-created for your project). The pipeline reads the resource group, location, and service connection from the pre-published lab-vars variable group.

Project Settings > Agent pools > kk-devops-agent-pool-xxxxx > Agents – Confirm that kk-devops-agent is Online.<br> Project Settings > Service connections – Confirm that a service connection exists.<br> Pipelines > Library – Confirm that the lab-vars variable group exists.

## Exercise 1 – Task 1: Review main.bicep and Modify a Parameter

In this task, you will review the structure of main.bicep and make a small, reviewable parameter change to the inline storage account.

Open infra/main.bicep (Repos > Files). It already provisions the Web App through a reusable module (modules/webapp.bicep), but the storage account is still defined inline:

![alt text](<Screenshot 2026-11-02 165441.png>)

Make a small, reviewable change first. In the inline storageAccount resource, change the redundancy from locally redundant to read-access geo-redundant storage:

    sku: {
    name: 'Standard_RAGRS'   // was 'Standard_LRS'
    }

Commit the change to main (edit the file in the browser > Commit). You will deploy and verify this in Exercise 2.

## Exercise 1 – Task 2: Create the Storage Module

In this task, you will extract the storage account into its own reusable Bicep module with parameters and an output.

Now, extract the storage account into its own reusable module.

In Repos > Files, under infra/modules/, create a new file named storage.bicep. Add the storage account as a parameterized module that returns its name as an output (keep the Standard_RAGRS sku from Task 1):

    // Storage account module.
    @description('Resource location.')
    param location string

    @description('Globally-unique storage account name (3-24 lowercase alphanumerics).')
    param storageAccountName string

    resource storageAccount 'Microsoft.Storage/storageAccounts@**2023**-01-01' = {
    name: storageAccountName
    location: location
    sku: {
    name: 'Standard_RAGRS'
    }
    kind: 'StorageV2'
    properties: {
    minimumTlsVersion: 'TLS1_2'
    allowBlobPublicAccess: false
    supportsHttpsTrafficOnly: true
    }
    }

    output storageAccountName string = storageAccount.name

Commit storage.bicep to main.<br> A module is simply a Bicep file you reference from another template, with its own params and outputs. The same module can be reused across deployments with different parameters.

## Exercise 1 – Task 3: Integrate the Module into main.bicep

In this task, you will replace the inline storage account with a reference to your new module and surface its output from main.bicep.

Replace the inline storage account in main.bicep with a reference to your new module, and surface its output.

Remove the inline resource storageAccount '...' = { ... } block from main.bicep. Add a module reference in its place:

    module storage 'modules/storage.bicep' = {
    name: 'storageDeploy'
    params: {
    location: location
    storageAccountName: 'st${take(suffix, 16)}'
    }
    }

At the bottom, add an output that consumes the module's output:

    output storageAccountName string = storage.outputs.storageAccountName

Commit to main. main.bicep now composes two modules (web app + storage) and returns the storage account name as a deployment output.

## Exercise 2 – Task 1: Deploy with the Pipeline and Verify

In this task, you will run the pipeline that lints and what-if-previews the template, deploys the modules, and then verify the provisioned resources in Azure.

Create and run the pipeline<br> Pipelines > Pipelines > New pipeline > Azure Repos Git > python-calculator-webapp.<br> Select Existing Azure Pipelines **YAML** file, Branch main, Path /pipelines/bicep-deploy.yml. Continue > Run.<br> Watch the stages:<br> Validate – az bicep build lints the template, and what-if previews the changes. Deploy – The modules are deployed (storage + web app), and the app is deployed to the Web App.<br> On the first run, the pipeline pauses, waiting for permission to use the service connection. Open the running job by clicking the stage/job in the run summary. The banner "This pipeline needs permission to access a resource before this run can continue" appears inside the job view. Click View > Permit > Permit to authorize it. You may need to do this for each stage on the first run.<br> Verify<br> In the Azure Portal ([https://portal.azure.com/](https://portal.azure.com/)), open the pre-created resource group and confirm:<br> A Storage account (st...) with Read-access geo-redundant (RA-**GRS**) replication An App Service (bicep-pycalc-...) and its App Service Plan<br> Browse the App Service **URL** and confirm the Simple Calculator page loads. Tip: Open the Azure portal link in a new tab of the same browser window you are using for Azure DevOps. The credentials are the same, so you will already be signed in and won't need to log in again.

Note: The first request after deployment can take up to a minute while the Web App cold-starts. Wait ~30-60 seconds after the pipeline finishes before browsing the app or checking the validation.

![alt text](<Screenshot 2026-11-02 161918.png>)

![alt text](<Screenshot 2026-11-02 162223.png>)

![alt text](<Screenshot 2026-11-02 162256.png>)

![alt text](<Screenshot 2026-11-02 162522.png>)

Review<br> Congratulations! You have completed the Deployments using Azure Bicep Templates lab.

In this lab, you:

Reviewed a Bicep template and modified a resource parameter<br> Created a reusable module (modules/storage.bicep) and integrated it into main.bicep, consuming its output<br> Deployed the infrastructure and application through an Azure Pipeline, with lint and what-if in a Validate stage<br> Verified the provisioned storage account and Web App in Azure<br> Modular Bicep gives you versioned, reviewable, repeatable deployments, with previews that reduce the risk of unintended changes.