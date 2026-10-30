# Lab: Set up and Run Functional Tests

In this lab, you will add automated tests to a CI pipeline for a Python (Flask) calculator and review the results in the pipeline's Tests tab, following the official lab. The pipeline runs unit tests and functional tests entirely in the pipeline (the functional tests drive the app in-process with Flask's test client, no deployment or browser required).

This lab covers the following exercises and tasks:

Exercise 0: Configure Lab Prerequisites Task 1: Import the Python Calculator Repository Task 2: Verify the Self-Hosted Agent

Exercise 1: Set up Tests in the CI Pipeline Task 1: Create the CI Pipeline (build + unit tests) Task 2: Add Functional Tests to the Pipeline Task 3: Check the Test Summary Results

## Exercise 0 – Task 1: Import the Python Calculator Repository In this task, you will import the Python Calculator source into a new Azure Repos Git repository named python-calculator-webapp and set main as its default branch.

In your Azure DevOps project, go to Repos > Files. Open the repository selector dropdown at the top of the page (it shows the project's default repository name) and click *Import repository*. ⚠ Use the dropdown, not the button on the page. A new project's default repository is empty, so the Repos > Files page also shows an *Import a repository* button. That button imports into the default repository and its dialog has no Name field, so the code lands in a repository named after the project and the validation for this task fails. Only the dropdown's Import repository option creates a new, nameable repository.

In the Import a Git repository dialog, paste this into Clone **URL**: [https://github.com/kodekloudhub/python-calculator-webapp.git](https://github.com/kodekloudhub/python-calculator-webapp.git)

In the Name field, make sure the repository is named exactly python-calculator-webapp. Azure DevOps auto-fills this from the clone **URL** and appends .git by default; delete the trailing .git so the name matches exactly. Then click *Import*. Make sure that main is set as the default branch (Repos > Branches). Already imported into the default repository? You do not need to redo the import; rename it instead: go to Project Settings > Repositories, select the repository, open the ⋮ (More options) menu, choose Rename, and enter python-calculator-webapp.

![alt text](<Screenshot 2026-10-30 153834.png>)

## Exercise 0 – Task 2: Verify the Self-Hosted Agent In this task, you will confirm the lab's self-hosted agent is online before you create the pipeline that runs the tests on it.

The lab provisions a self-hosted Linux agent kk-devops-agent in the kk-devops-agent-pool-xxxxx pool. The pipeline runs the tests on this agent.

Project Settings > Agent pools > kk-devops-agent-pool-xxxxx > Agents – Confirm kk-devops-agent is Online. This lab runs the tests in the pipeline only; no Azure resources or service connection are required.

![alt text](<Screenshot 2026-10-30 154150.png>)

![alt text](<Screenshot 2026-10-30 154200.png>)

![alt text](<Screenshot 2026-10-30 154252.png>)

![alt text](<Screenshot 2026-10-30 154715.png>)

## Exercise 1 – Task 1: Create the CI Pipeline (build + unit tests) In this task, you will create and run the CI pipeline that runs the unit tests and publishes the results to the Tests tab.

The repository ships pipelines/python-functional-tests.yml, which installs dependencies, runs the unit tests (tests/test_calc.py), and publishes the results to the Tests tab. Create and run it.

Navigate to Pipelines > Pipelines > New Pipeline. Select Azure Repos Git > python-calculator-webapp > Existing Azure Pipelines **YAML** File. Branch main, Path /pipelines/python-functional-tests.yml. Click Continue > Run. When the run completes, open the run and check the Tests tab; the unit tests appear there. Rename the pipeline to python-functional-tests.

![alt text](<Screenshot 2026-10-30 155013.png>)

## Exercise 1 – Task 2: Add Functional Tests to the Pipeline In this task, you will edit the pipeline **YAML** to add a functional-test step alongside the unit tests.

Now, edit the pipeline **YAML** to add the functional tests. These use Flask's test client to drive the app's routes in-process (functional-tests/test_functional.py), no deployment or browser needed.

Open python-functional-tests and click *Edit*.
Insert the following step immediately after the *Run unit tests* step (and before the PublishTestResults@2 task):
- script: |
    source venv/bin/activate
    pytest functional-tests/test_functional.py --junitxml=functional-results.xml
    displayName: Run functional tests

Watch the indentation. In this pipeline, the steps are not indented under steps:; each - script: / - task: starts at the left margin. After pasting, check that the new - script: lines up with the - script: above it and that the editor did not add extra leading spaces. Use the editor's Validate to confirm before saving.

Click *Validate and save* > *Save* (commit directly to main). This queues a new run. The pipeline now runs both the unit and functional tests; the existing Publish test results task picks up functional-results.xml automatically (it globs *-results.xml).

![alt text](<Screenshot 2026-10-30 155200.png>)

## Exercise 1 – Task 3: Check the Test Summary Results In this task, you will review the combined unit and functional test results in the pipeline run's Tests tab.

Navigate to Pipelines > Pipelines > python-functional-tests and open the latest run. Open the Tests tab. Confirm the summary shows both the unit tests and the functional tests passing (total tests increased after Task 2). Expand the results to see the individual test cases (e.g., test_homepage_loads, test_addition, test_division_by_zero_shows_error). Congratulations! You added functional tests to a CI pipeline and reviewed the results in the Tests tab, without deploying the app anywhere.

![alt text](<Screenshot 2026-10-30 155301.png>)

## Review  - Congratulations! You have completed the Set up and Run Functional Tests lab.

In this lab, you:

Created a CI pipeline that runs the unit tests and publishes results to the Tests tab Added functional tests (using Flask's test client to exercise the app's routes in-process) Reviewed the combined test summary in the pipeline's Tests tab Running unit and functional tests in the pipeline gives fast, automated feedback on every change, catching regressions before code is deployed.