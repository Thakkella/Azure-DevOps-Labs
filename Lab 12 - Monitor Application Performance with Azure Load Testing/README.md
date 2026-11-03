# Lab: Monitor Application Performance with Azure Load Testing

In this lab, you will deploy the Python (Flask) calculator app and use Azure Load Testing to generate load against it: first interactively from the portal, then automated in a CI/CD pipeline with failure criteria that fail the build when performance regresses.

This lab covers the following exercises and tasks:

Exercise 0: Configure Lab Prerequisites<br> Task 1: Import the Python Calculator Repository<br> Task 2: Verify the Self-Hosted Agent

Exercise 1: Build and Deploy with a Pipeline<br> Task 1: Build and Deploy the Calculator via a Pipeline

Exercise 2: Load Test from the Portal<br> Task 1: Create an Azure Load Testing Resource<br> Task 2: Create and Run a **URL**-based Load Test

Exercise 3: Automate Load Testing in CI/CD<br> Task 1: Add a Load Test Stage to the Pipeline

## Exercise 0 – Task 1: Import the Python Calculator Repository

In your Azure DevOps project, go to Repos > Files. Open the repository selector dropdown at the top and click *Import repository* (this creates a new repository; do not import into the default project repository).<br> In Clone **URL**, paste:<br> [https://github.com/kodekloudhub/python-calculator-webapp.git](https://github.com/kodekloudhub/python-calculator-webapp.git)

In the Name field, make sure the repository is named exactly python-calculator-webapp. Azure DevOps auto-fills this from the clone **URL** and appends .git by default; delete the trailing .git so the name matches exactly. Then click *Import*.<br> Make sure that main is set as the default branch (Repos > Branches). The repository contains infra/webapp.bicep, the loadtest/ test plan + config, and pipelines/loadtest-pipeline.yml used in this lab.

## Exercise 0 – Task 2: Verify the Self-Hosted Agent

The lab provisions a self-hosted Linux agent kk-devops-agent in the kk-devops-agent-pool-xxxxx pool, and uses a pre-created **ARM** service connection. Exercise 3's pipeline runs on this agent and reads the resource group, location, and service connection from the pre-published lab-vars variable group.

Project Settings > Agent pools > kk-devops-agent-pool-xxxxx > Agents – Confirm that kk-devops-agent is Online.<br> Project Settings > Service connections – Confirm that a service connection exists.<br> Pipelines > Library – Confirm that the lab-vars variable group exists.

## Exercise 1 – Task 1: Build and Deploy the Calculator via a Pipeline

Deploy the app to a Linux Web App using the CI/CD pipeline pipelines/loadtest-pipeline.yml. Its Build stage tests and packages the app; its Deploy stage provisions infra/webapp.bicep (a B1 Linux Web App) and deploys the app to it.

Pipelines > Pipelines > Create pipeline > Azure Repos Git > python-calculator-webapp.<br> Select Existing Azure Pipelines **YAML** file, Branch main, Path /pipelines/loadtest-pipeline.yml. Continue > Run.<br> On the first run, in the Deploy Web App stage, click View > Permit to authorize the service connection / variable group if prompted.<br> Rename the pipeline to loadtest-pipeline.<br> When the Deploy stage finishes, a Web App (gha-pycalc-...) is running in the pre-created resource group.<br> Note: The first request after deployment can take up to a minute while the Web App cold-starts. Wait ~30-60 seconds before browsing or checking the validation.

If the Deploy stage fails with a capacity error (No available instances..., error **03029**), that's a transient Azure shortage for the B1 tier in the region; re-run the pipeline, or change the location parameter to another allowed region (West US / Central US / South Central US).

## Exercise 2 – Task 1: Create an Azure Load Testing Resource

In this task, you will deploy an Azure Load Testing resource; the managed service that generates load against your app from the Azure Portal.

From the Azure Portal, select Create a resource.

In the Search services and marketplace field, enter Azure Load Testing and press Enter.

Select Azure Load Testing (published by Microsoft) from the search results, then click Create.

On the Basics tab of the Create a load testing resource page, provide the following details:

Subscription: Use the default subscription associated with your Azure account. Resource group: Use the pre-created resource group.<br> Name: alt-az400-loadtest.<br> Region: East US.<br> Click Review + create to validate your settings, then click Create to start the deployment.

Wait a few minutes until the Deployment is in progress page reports the deployment completed successfully, then click Go to resource to open your alt-az400-loadtest resource.

Note: If you closed the blade during deployment, you can reopen the resource from the Portal Search field or the Recent resources list.

Optional – Create it with the **CLI** instead. Run the following in the lab terminal:

    RG=$(cat /root/code/lab-resource-group.txt 2>/dev/null || az group list --query "[0].name" -o tsv)
    az extension add --name load --only-show-errors
    az load create --name alt-az400-loadtest --resource-group "$RG" --location eastus

## Exercise 2 – Task 2: Create and Run a URL-based Load Test

In this task, you will explore Azure Load Testing interactively by creating and running a quick **URL**-based test from the portal and then validate the results.

Create and run the test<br> From within your Azure Load Testing resource (alt-az400-loadtest), navigate to Tests under Tests in the left menu. Click + Create and select Create a **URL**-based test.

Uncheck the Enable advanced settings checkbox (it is enabled by default).

Complete the following parameters and settings to create the load test:

Test **URL**: The **URL** of the Web App you deployed in Exercise 1 (gha-pycalc-...azurewebsites.net), including [https://.<br>](https://.<br>) Specify load: Virtual users<br> Number of virtual users: 20<br> Test duration (minutes): 2<br> Ramp-up time (minutes): 1<br> Click Review + create (do not change anything on the other tabs), then click Create once more.

This kicks off the load test, which runs for about 2 minutes. Stay on the test run page while it executes.

Validate the results<br> When the run finishes, review the test-run dashboard and identify the key metrics:

Total requests – The total number of requests sent during the run<br> Response time – Shown as the 90th percentile (90% of requests completed within this time), alongside the average<br> Throughput – Requests per second the app sustained<br> Errors (%) – The percentage of failed requests<br> Below the summary tiles, the same values are plotted as client-side metric graphs (response time, virtual users, requests/sec, and errors) over the duration of the run.

Take a few minutes to interpret the results and consider how more virtual users affect the B1 App Service's response time and error rate, the same signals you will gate the pipeline on in Exercise 3.

## Exercise 3 – Task 1: Add a Load Test Stage to the Pipeline

Now, gate the pipeline on performance. Add a LoadTest stage that runs the JMeter test (loadtest/calculator.jmx) against the deployed app via az load, and fails the build if the run breaches the criteria in loadtest/loadtest.yaml (avg response time > **2000** ms or errors > 5%).

Edit pipelines/loadtest-pipeline.yml (Repos > Files). After the Deploy stage, where the comment marks Exercise 3, append this stage:

    - stage: LoadTest
      displayName: Load Test
      dependsOn: Deploy
      jobs:
      - job: LoadTest
        pool:
          name: $(agentPool)
        steps:
        - checkout: self
        - task: AzureCLI@2
          displayName: Run the load test with Azure Load Testing
          inputs:
            azureSubscription: $(azureServiceConnection)
            scriptType: bash
            scriptLocation: inlineScript
            inlineScript: |
              set -e
              az extension add --name load --only-show-errors
              RG="$(resourceGroup)"
              ALT=$(az load list --resource-group "$RG" --query "[0].name" -o tsv)
              WEBAPP=$(az webapp list --resource-group "$RG" --query "[0].name" -o tsv)
              HOST=$(az webapp show --resource-group "$RG" --name "$WEBAPP" --query defaultHostName -o tsv)
              cd loadtest
              sed -i "s/REPLACE_WITH_WEBAPP_HOST/$HOST/" loadtest.yaml
              RUN_ID="ci-run-$(Build.BuildId)"
              az load test create --load-test-resource "$ALT" --resource-group "$RG" --test-id calculator-ci-test --load-test-config-file loadtest.yaml
              az load test-run create --load-test-resource "$ALT" --resource-group "$RG" --test-id calculator-ci-test --test-run-id "$RUN_ID" || true
              STATUS="NOT_APPLICABLE"
              for i in $(seq 1 20); do
                STATUS=$(az load test-run show --load-test-resource "$ALT" --resource-group "$RG" --test-run-id "$RUN_ID" --query "testResult" -o tsv 2>/dev/null)
                echo "Attempt $i: testResult=$STATUS"
                if [ "$STATUS" = "PASSED" ] || [ "$STATUS" = "FAILED" ]; then break; fi
                sleep 15
              done
              echo "Final load test result: $STATUS"
              if [ "$STATUS" != "PASSED" ]; then
                echo "##vso[task.logissue type=error]Load test result: $STATUS"
                exit 1
              fi

Commit to main. Run the pipeline again. The new Load Test stage provisions the test, runs it, polls until Azure finalizes the result, and reports **PASSED** / **FAILED**.<br> No role grant needed: The pipeline authenticates with the service connection, whose identity has Contributor at the subscription level, which automatically grants Load Test data-plane access. (To see the gate fail like a real regression check, tighten failureCriteria in loadtest.yaml, e.g., avg(response_time_ms) > **300**, and re-run.)

Review<br> Congratulations! You have completed the Monitor Application Performance with Azure Load Testing lab.

In this lab, you:

Deployed the calculator app to Azure App Service<br> Created an Azure Load Testing resource and ran a **URL**-based test from the portal<br> Automated load testing in a CI/CD pipeline with az load, gating the build on failure criteria (response time + error %)<br> Running load tests in your pipeline lets you automatically catch performance regressions before they reach users and fail the build when the app no longer meets its performance targets.