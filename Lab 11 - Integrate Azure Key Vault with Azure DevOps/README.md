# Lab: Integrate Azure Key Vault with Azure DevOps

In this lab, you will build a container image for a Python (Flask) calculator with Docker Compose, push it to Azure Container Registry (**ACR**), store the **ACR** password in Azure Key Vault as the secret acr-secret, and consume that secret in a release pipeline (through a Key Vault-linked variable group) to deploy the container to Azure Container Instances (**ACI**).

This lab covers the following exercises and tasks:

Exercise 0: Configure Lab Prerequisites<br> Task 1: Import the Python Calculator Repository<br> Task 2: Verify the Self-Hosted Agent and Service Connection

Exercise 1: Build the Image and Store the **ACR** Secret<br> Task 1: Run the Docker Compose CI Pipeline (provision **ACR** + build/push) Task 2: Enable the **ACR** Admin User and Store its Password in Key Vault

Exercise 2: Consume the Secret in a Release Pipeline<br> Task 1: Create a Key Vault-Linked Variable Group<br> Task 2: Run the CD Pipeline to Deploy to **ACI** and Verify

## Exercise 0 – Task 1: Import the Python Calculator Repository

In this task, you will import the Python Calculator source into a new Azure Repos Git repository and set main as its default branch.

In your Azure DevOps project, go to Repos > Files.<br> Open the repository selector dropdown at the top and click *Import repository* (this creates a new repository; do not import into the default project repository).<br> In Clone **URL**, paste:<br> [https://github.com/kodekloudhub/python-calculator-webapp.git](https://github.com/kodekloudhub/python-calculator-webapp.git)

In the Name field, make sure the repository is named exactly python-calculator-webapp. Azure DevOps auto-fills this from the clone **URL** and appends .git by default; delete the trailing .git so the name matches exactly. Then click *Import*.<br> Make sure that main is set as the default branch (Repos > Branches). The repository contains the app, Dockerfile, docker-compose.yml, the infra/acr.bicep template, and the pipelines used below.

## Exercise 0 – Task 2: Verify the Self-Hosted Agent and Service Connection

In this task, you will confirm that the lab's Docker-enabled agent and service connection are in place before you build any pipelines.

The lab provisions a self-hosted Linux agent kk-devops-agent (with Docker + Compose installed) in the kk-devops-agent-pool-xxxxx pool, and uses a pre-created **ARM** service connection (pre-created for your project).

Project Settings > Agent pools > kk-devops-agent-pool-xxxxx > Agents – Confirm that kk-devops-agent is Online.<br> Project Settings > Service connections – Confirm that a service connection exists.

## Exercise 1 – Task 1: Review the Docker Compose CI Pipeline

In this task, you will review how the Docker Compose CI pipeline provisions **ACR** and builds and pushes the image, before you run it.

Open pipelines/python-calc-ci-dockercompose.yml. It:

Deploys infra/acr.bicep to create an Azure Container Registry and outputs its login server. Parses the login server from the Bicep output. Uses the DockerCompose@1 task to build the services defined in docker-compose.yml and push them to **ACR**.

    - task: DockerCompose@1
      displayName: Build services
      inputs:
        containerregistrytype: 'Azure Container Registry'
        azureSubscription: $(azureServiceConnection)
        azureContainerRegistry: '$(acrLoginServer)'
        dockerComposeFile: 'docker-compose.yml'
        action: 'Build services'

docker-compose.yml defines a single web service built from the Dockerfile. The build runs on the self-hosted agent (which has Docker installed), and the image is pushed as web:<buildId> / web:latest.

Note: The pipeline reads azureServiceConnection, resourceGroup, location, and subscriptionId from the lab-vars variable group (set at startup).

## Exercise 1 – Task 1 (cont.): Run the Docker Compose CI Pipeline

In this task, you will create and run the CI pipeline to provision **ACR** and push the container image.

Navigate to Pipelines > Pipelines > New Pipeline.<br> Select Azure Repos Git (**YAML**) > python-calculator-webapp.<br> Select Existing Azure Pipelines **YAML** File, Branch main, Path /pipelines/python-calc-ci-dockercompose.yml. Click Continue > Run.<br> Confirm the steps complete: Deploy **ACR** (Bicep), Parse **ACR** login server, Build services, Push services.<br> Rename the pipeline to python-calc-ci-dockercompose.<br> In the Azure Portal, open the **ACR** (cr...) > Repositories and confirm the web image is present.<br> First run – Permit access: The first time the pipeline runs, it pauses with "This pipeline needs permission to access a resource before this run can continue.* Click *View* > *Permit" to authorize the service connection, and the run continues.

![alt text](<Screenshot 2026-11-03 122331.png>)

## Exercise 1 – Task 2: Enable the ACR Admin User and Store its Password in Key Vault

In this task, you will enable the **ACR** admin user, create a Key Vault, store the **ACR** password as the acr-secret secret, and grant the pipeline service principal access to read it.

Now, capture the **ACR** admin password and store it in Azure Key Vault as the secret acr-secret. Use the lab terminal.

Signed in? The lab terminal is already authenticated to Azure. If any command below returns *Please run 'az login' to setup account*, run az login, pick the lab subscription, and re-run the block.

    RG=$(cat /root/code/lab-resource-group.txt 2>/dev/null || az group list --query "[0].name" -o tsv)
    LOC=$(az group show --name *$RG* --query location -o tsv)
    ACR=$(az acr list --resource-group *$RG* --query "[0].name" -o tsv)

1. Enable the **ACR** admin user and read its password

    az acr update --name "$ACR" --admin-enabled true
    ACR_PW=$(az acr credential show --name "$ACR" --query "passwords[0].value" -o tsv)

2. Create a Key Vault using the access-policy model. Key Vault names are unique across all of Azure, so derive one from this lab's resource group suffix.

    KV_NAME="kv-acr-${RG##*-}" 
    echo "Key Vault: $KV_NAME"   # note this for the variable group
    az keyvault create --name "$KV_NAME" --resource-group "$RG" --location "$LOC" --enable-rbac-authorization false --retention-days 7

3. Store the **ACR** password as the 'acr-secret' secret

    az keyvault secret set --vault-name "$KV_NAME" --name acr-secret --value "$ACR_PW"

4. Grant the pipeline service principal get/list on secrets

    CLIENT_ID=$(showcreds | awk -F'│' '/Azure Application Client ID/ {gsub(/^[ \t]+|[ \t]+$/,"",$3); print $3}')
    SP_OBJECT_ID=$(az ad sp show --id "$CLIENT_ID" --query id -o tsv)
    az keyvault set-policy --name "$KV_NAME" --object-id "$SP_OBJECT_ID" --secret-permissions get list

Why enable the admin user? Azure Container Instances authenticates to **ACR** with a registry username/password. You store that password in Key Vault as acr-secret so the release pipeline can read it without hardcoding it.

![alt text](<Screenshot 2026-11-03 122331-1.png>)

![alt text](<Screenshot 2026-11-03 123100.png>)

![alt text](<Screenshot 2026-11-03 124610.png>)
![alt text](<Screenshot 2026-11-03 124638.png>)
![alt text](<Screenshot 2026-11-03 125750.png>)

![alt text](<Screenshot 2026-11-03 125800.png>)

![alt text](<Screenshot 2026-11-03 131639.png>)
![alt text](<Screenshot 2026-11-03 131710.png>)

![alt text](<Screenshot 2026-11-03 131800.png>)

## Exercise 2 – Task 1: Create a Key Vault-Linked Variable Group

In this task, you will create a variable group that links to your Key Vault so the release pipeline can read acr-secret as a pipeline variable.

Create a variable group named acr-keyvault-vg that links to your Key Vault and exposes acr-secret.

Steps:<br> Navigate to Pipelines > Library.<br> Click *+ Variable group*.<br> Variable group name: acr-keyvault-vg<br> Toggle *Link secrets from an Azure key vault as variables* to On.<br> Azure subscription: Select the pre-created service connection (authorize if prompted).<br> Key vault name: Select the vault you created in the previous task (the name printed by echo *Key Vault: $KV_NAME*).<br> Click *+ Add*, select acr-secret, and click OK.<br> Click *Save*.

![alt text](<Screenshot 2026-11-03 132007.png>)

## Exercise 2 – Task 2: Run the CD Pipeline to Deploy to ACI and Verify

In this task, you will run the CD pipeline that reads acr-secret from the linked variable group to authenticate to **ACR**, deploy the image to Azure Container Instances, and then verify the running app.

The pipeline pipelines/python-calc-cd-aci.yml references the acr-keyvault-vg variable group and uses $(acr-secret) to authenticate to **ACR**, then deploys the image to Azure Container Instances.

    variables:
    - group: lab-vars
    - group: acr-keyvault-vg     # provides acr-secret from Key Vault
    ...
    az container create ... --registry-password *$(acr-secret)* ...

Steps:<br> Navigate to Pipelines > Pipelines > New Pipeline.<br> Select Azure Repos Git (**YAML**) > python-calculator-webapp.<br> Select Existing Azure Pipelines **YAML** File, Branch main, Path /pipelines/python-calc-cd-aci.yml. Click Continue.<br> Click *Run*. If prompted, permit the pipeline to use acr-keyvault-vg and the service connection.<br> Watch the Deploy stage authenticate with the Key Vault secret and create the container group pycalc-aci.<br> Rename the pipeline to python-calc-cd-aci.<br> Verify<br> In the Azure Portal, open the Container instances resource pycalc-aci, copy its **FQDN**, and browse to [http://<fqdn>](http://<fqdn>) – confirm that the Simple Calculator page loads.

![alt text](<Screenshot 2026-11-03 145327.png>)

![alt text](<Screenshot 2026-11-03 151440.png>)

![alt text](<Screenshot 2026-11-03 151515.png>)

![alt text](<Screenshot 2026-11-03 151950.png>)

Review<br> Congratulations! You have successfully completed the Integrate Azure Key Vault with Azure DevOps lab.

In this lab, you:

Built and pushed a container image to Azure Container Registry using a Docker Compose pipeline<br> Enabled the **ACR** admin user and stored its password in Azure Key Vault as acr-secret<br> Granted the pipeline service principal get/list access to the vault<br> Created a Key Vault-linked variable group (acr-keyvault-vg) exposing acr-secret<br> Ran a CD pipeline that consumed $(acr-secret) to authenticate to **ACR** and deploy the container to Azure Container Instances<br> Storing registry (and other) secrets in Key Vault and surfacing them to pipelines through a linked variable group keeps credentials out of source control and pipeline definitions, with central management and rotation.