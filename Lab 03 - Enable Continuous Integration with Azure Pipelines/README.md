# Lab: Enable Continuous Integration with Azure Pipelines - In this lab, you will learn how to define build pipelines in Azure DevOps using **YAML**. The pipelines will be used in two scenarios:

As part of a Pull Request validation workflow (build validation policy) As a Continuous Integration (CI) pipeline that triggers automatically on code changes You will work with a Python Calculator application hosted in Azure Repos.

This lab covers the following exercises and tasks:

Exercise 0: Configure Lab Prerequisites Task 1: Import the Python Calculator Repository Task 2: Set Main as Default Branch

Exercise 1: Include Build Validation as Part of a Pull Request Task 1: Import the PR Validation **YAML** Pipeline Task 2: Configure Branch Policies Task 3: Test the PR Validation Workflow

Exercise 2: Configure CI Pipeline as Code with **YAML** Task 1: Import the CI **YAML** Pipeline Task 2: Enable Continuous Integration Trigger Task 3: Test the CI Pipeline

## Exercise 0 — Task 1: Import the Python Calculator Repository In this task, you will import the Python Calculator source into a new Azure Repos Git repository in your Azure DevOps project.

The repository contains:

pipelines folder – Azure DevOps **YAML** pipeline definitions infra folder – Bicep infrastructure-as-code templates src folder – Python application source code tests folder – Unit tests for the application python_calc.py – Flask web calculator application requirements.txt – Python dependencies Steps: In your Azure DevOps project, go to Repos > Files.

Open the repository selector dropdown at the top and click *Import repository* (this creates a new repository; do not import into the default project repository).

In Clone **URL**, paste:

   [https://github.com/kodekloudhub/python-calculator-webapp.git](https://github.com/kodekloudhub/python-calculator-webapp.git)

In the Name field, make sure the repository is named exactly python-calculator-webapp. Azure DevOps auto-fills this from the clone **URL** and appends .git by default; delete the trailing .git so the name matches exactly. Then click *Import*.

Wait for the import to complete; the repository files should now appear in the Repos > Files view.

If the import fails or prompts for credentials: The source repository is public, so no authentication is required; leave any credential fields blank. A one-off failure is usually a transient GitHub rate-limit, so simply retry the import. Alternatively, clone it locally with git clone [https://github.com/kodekloudhub/python-calculator-webapp.git,](https://github.com/kodekloudhub/python-calculator-webapp.git,) create an empty repository named exactly python-calculator-webapp in Azure Repos, and push the contents.

## Exercise 0 — Task 2: Set Main as Default Branch
After importing, the default branch may not be set to main. In this task, you will set main as the default branch.

Steps:
Navigate to Repos > Branches

Hover over the main branch

Click the ellipsis (…) on the right side

Click "Set as default branch"

Verify that main now shows the default badge

## Exercise 1: Include Build Validation as Part of a Pull Request - Task 1: Import the PR Validation YAML Pipeline
In this task, you will import the YAML build pipeline that will be used as a branch policy to validate Pull Requests before they can be merged into main.

Steps:
Navigate to Pipelines > Pipelines

Click "Create Pipeline" (or "New Pipeline")

Select "Azure Repos Git (YAML)" as the source

Select the python-calculator-webapp repository

Select "Existing Azure Pipelines YAML File"

In the panel that appears:

Branch: main
Path: /pipelines/python-calc-ci-pr.yml
Click "Continue"

On the "Review your pipeline YAML" page, click the down-facing caret next to the Run button

Click "Save" (do not run the pipeline yet)

Navigate back to Pipelines > Pipelines, click on the newly created pipeline

Click the ellipsis (…) and select "Rename/Move"

Name it: python-calc-ci-pr

Click "Save"

Important: When you create the pipeline, Azure DevOps auto-names it after the repository (python-calculator-webapp), not after the YAML file. You must rename it to python-calc-ci-pr (steps 11–13) or the validation check will not pass.

## Exercise 1: Include Build Validation as Part of a Pull Request - Task 2: Configure Branch Policies In this task, you will add branch policies to the main branch. These policies will:

Require at least one reviewer for every Pull Request Require the python-calc-ci-pr pipeline to pass as build validation before merging Steps: Navigate to Repos > Branches

On the Mine tab, hover over the main branch

Click the ellipsis (…) and select *Branch Policies*

Under *Require minimum number of reviewers*:

Toggle it On Set the minimum number of reviewers to 1 Check *Allow requestors to approve their own changes* (this lets you approve your own PR in the lab) Under *Build Validation* section:

Click the *+* button to add a build policy In the Build pipeline dropdown, select python-calc-ci-pr Leave other settings at defaults Click *Save* Note: In a real-world project, you would typically not allow requestors to approve their own changes. We enable this here so you can complete the lab without a second user account.

## Exercise 1: Include Build Validation as Part of a Pull Request - Task 3: Test the PR Validation Workflow In this task, you will test the Pull Request validation by creating a feature branch, making a code change, and submitting a Pull Request that triggers the build validation pipeline.

Create a Feature Branch and Make a Change: Navigate to Repos > Branches

Click *New branch*

Name the branch: Feature01 (based on main)

Click *Create*

Click on the Feature01 branch to browse its files

Navigate to src/calc.py

Click the *Edit* button (top-right)

Add the following comment on the first line:

   ### Testing my PR

Click *Commit > Commit* (accept the default commit message) Create and Complete the Pull Request: When prompted, click *Create a Pull Request*

In the New pull request tab, leave defaults and click *Create*

The PR will show pending requirements:

Minimum reviewer count not met Build validation running (python-calc-ci-pr) Wait for the python-calc-ci-pr build to complete successfully

Click *Approve* (top-right) to satisfy the reviewer requirement

From the *Set auto-complete* dropdown, click *Complete*

On the Complete Pull Request tab, click *Complete Merge*

## Exercise 2: Configure CI Pipeline as Code with **YAML** - Task 1: Import the CI **YAML** Pipeline In this task, you will import the full CI pipeline **YAML** definition. This pipeline performs additional steps compared to the PR validation pipeline, including archiving and publishing build artifacts.

Steps: Navigate to Pipelines > Pipelines

Click *New Pipeline*

Select *Azure Repos Git (**YAML**)* as the source

Select the python-calculator-webapp repository

Select *Existing Azure Pipelines **YAML** File*

In the panel:

Branch: main Path: /pipelines/python-calc-ci-build.yml Click *Continue*

On the review page, click *Run* to save and execute the pipeline

Wait for the pipeline run to complete successfully

Navigate back to Pipelines > Pipelines, click on the pipeline

Click the ellipsis (…) and select *Rename/Move*

Name it: python-calc-ci

Click *Save*

## Exercise 2: Configure CI Pipeline as Code with **YAML** - Task 2: Enable Continuous Integration Trigger In this task, you will modify the CI pipeline **YAML** to enable automatic triggering when code changes are pushed to the main branch under the src/ path.

By default, the trigger section in the **YAML** file is commented out. You will uncomment and configure it.

Steps: Navigate to Pipelines > Pipelines and click on the python-calc-ci pipeline

Click *Edit* (or click the ellipsis near *Run new* and select *Edit pipeline*)

In the **YAML** editor, find the commented-out trigger lines:

    # trigger:
    # - main

Replace them with the following trigger configuration:
trigger:
    branches:
    include:
    - main
    paths:
    include:
    - src/*

This configuration means the pipeline will automatically run whenever:

A commit is pushed to the main branch **AND** The commit includes changes to files inside the src/ folder Click *Validate and save*

Choose *Save directly to the main branch* (since we have branch policies, you may need to select *Create a new branch for this commit* and complete a PR)

If you created a branch:

Accept the default branch name Keep *Start a pull request* checked Click *Save* Navigate to Repos > Pull Requests Open the PR for the **YAML** update After validations pass, click *Approve* then *Complete* > *Complete Merge*

## Exercise 2: Configure CI Pipeline as Code with **YAML** - Task 3: Test the CI Pipeline In this task, you will verify that the CI pipeline triggers automatically when a code change is merged into main within the src/ path.

Create a Feature Branch and Make a Change: Navigate to Repos > Branches

Click *New branch*

Name the branch: Feature02 (based on main)

Click *Create*

Click on the Feature02 branch to browse its files

Navigate to src/calc.py

Click the *Edit* button (top-right)

Remove the comment # Testing my PR from the first line (added during Exercise 1)

Click *Commit > Commit* (accept the default commit message)

Create and Complete the Pull Request: When prompted, click *Create a Pull Request*

In the New pull request tab, leave defaults and click *Create*

Wait for the build validation to pass, then click *Approve*

Click *Complete* > *Complete Merge*

Verify the CI Pipeline Triggered: Navigate to Pipelines > Pipelines

Observe that the python-calc-ci pipeline has been triggered automatically by the merge

Click on the python-calc-ci pipeline run and select the latest run

Wait for the run to complete successfully

Click *Related* > *Published* to view the published artifacts:

Bicep – The infrastructure-as-code artifact Website – The application artifact

## Review - Congratulations! You have successfully completed the Enable Continuous Integration with Azure Pipelines lab.

In this lab, you:

Imported the Python Calculator repository into Azure Repos and set main as the default branch Created a PR validation pipeline (python-calc-ci-pr) from an existing **YAML** definition Configured branch policies on main requiring a minimum reviewer count and build validation Tested the Pull Request workflow by creating a feature branch, making a change, and merging through a validated PR Created a CI pipeline (python-calc-ci) from an existing **YAML** definition Configured CI triggers to automatically run the pipeline on code changes to src/* in the main branch Verified the CI pipeline triggers automatically on merge and publishes both Website and Bicep artifacts These practices form the foundation of a robust CI/CD workflow: automated build validation on Pull Requests ensures code quality, and Continuous Integration pipelines enable rapid feedback.