# Lab: Package Management with Azure Artifacts

In this lab, you will create an Azure Artifacts feed, publish a Python package (kk-calculator) to it from a pipeline, and learn how to consume the package with pip.

This lab covers the following exercises and tasks:

Exercise 0: Configure Lab Prerequisites<br> Task 1: Import the Python Calculator Repository<br> Task 2: Verify the Self-Hosted Agent

Exercise 1: Create a Feed and Publish a Package<br> Task 1: Create the Artifacts Feed<br> Task 2: Review the Package and Publish Pipeline<br> Task 3: Point the Pipeline at Your Feed and Run It

Exercise 2: Consume the Package<br> Task 1: Consume the Package with pip

## Exercise 0 – Task 1: Import the Python Calculator Repository

In this task, you will import the Python Calculator source (including the package source) into a new Azure Repos Git repository and set main as its default branch.

In your Azure DevOps project, go to Repos > Files.<br> Open the repository selector dropdown at the top and click *Import repository* (this creates a new repository; do not import into the default project repository).<br> In Clone **URL**, paste:<br> [https://github.com/kodekloudhub/python-calculator-webapp.git](https://github.com/kodekloudhub/python-calculator-webapp.git)

In the Name field, make sure the repository is named exactly python-calculator-webapp. Azure DevOps auto-fills this from the clone **URL** and appends .git by default; delete the trailing .git so the name matches exactly. Then click *Import*.<br> Make sure that main is set as the default branch (Repos > Branches). The repository contains the package/ source (the kk-calculator package) and pipelines/publish-package.yml used in this lab.

## Exercise 0 – Task 2: Verify the Self-Hosted Agent

In this task, you will confirm the lab's self-hosted agent is online before you run the publish pipeline on it.

The lab provisions a self-hosted Linux agent kk-devops-agent in the kk-devops-agent-pool-xxxxx pool; the publish pipeline runs on it. (This lab uses Azure Artifacts only; no Azure resources or service connection are needed.)

Project Settings > Agent pools > kk-devops-agent-pool-xxxxx > Agents – Confirm kk-devops-agent is Online.

## Exercise 1 – Task 1: Create the Artifacts Feed

In this task, you will create the Azure Artifacts feed that your package will be published to and consumed from.

An Azure Artifacts feed is where your packages are stored and versioned.

Name your feed with the fixed prefix kk-python-feed- followed by your lab username suffix. To find the suffix, run showcreds in the lab terminal and look at the Azure User Name, e.g., kk_lab_user_dev-1db8b60db0c94d88@...; the suffix is the part between kk_lab_user_dev- and @ (here, 1db8b60db0c94d88), so your feed would be kk-python-feed-1db8b60db0c94d88. Note the exact name; you will point the pipeline at it in Task 3.

Steps:<br> Navigate to Artifacts in the left sidebar.<br> Click *+ Create Feed*.<br> Name: kk-python-feed-<lab-username-suffix><br> Scope: Select Project (This project).<br> Leave Include packages from common public sources unchecked (not needed for this lab).<br> Click *Create*.<br> Project-scoped feeds normally grant the project's build service Contributor access automatically, so the pipeline can publish. If the publish step in Task 3 returns a **403** Forbidden, grant it manually using the note in that task.

![alt text](<Screenshot 2026-11-02 172009.png>)

![alt text](<Screenshot 2026-11-02 172052.png>)

![alt text](<Screenshot 2026-11-02 180651.png>)

## Exercise 1 – Task 2: Review the Package and Publish Pipeline

In this task, you will review the package source and the pipeline that builds and publishes it, so you understand each step before running it.

Note: Both the package source and the pipeline file already exist in the python-calculator-webapp repository you imported in Exercise 0 – Task 1. You do not need to create or upload anything in this task; browse to them in Repos > Files and read through them.

![alt text](<Screenshot 2026-11-02 184702.png>)

The publish pipeline (pipelines/publish-package.yml)

    variables:
        - group: lab-vars                    # lab-provided; supplies $(agentPool) -- do not remove
        - name: feedName                     # <-- you set this to **YOUR** feed in Task 3
        value: 'kk-python-feed'

    pool: name: $(agentPool)

    steps:
    - script: |
        sudo apt-get update
        sudo apt-get install -y python3 python3-pip python3-venv
        displayName: Install Python3, pip, venv

    - script: |
        python3 -m venv venv
        source venv/bin/activate
        pip install --upgrade pip build twine
        cd package
        python -m build                    # produces dist/*.whl and *.tar.gz
        displayName: Build the kk-calculator package (sdist + wheel)

    - task: TwineAuthenticate@1
        displayName: Authenticate twine with the Azure Artifacts feed
        inputs:
        artifactFeed: '$(System.TeamProject)/$(feedName)'   # project-scoped feed

    - script: |
        source venv/bin/activate
        twine upload -r $(feedName) --config-file $(PYPIRC_PATH) package/dist/*
        displayName: Publish the package to the feed

## Exercise 1 – Task 3: Point the Pipeline at Your Feed and Run It

In this task, you will set the pipeline's feed name, grant the build identity access, run the pipeline, and confirm the package appears in your feed.

In Repos > Files, open pipelines/publish-package.yml and edit it. The file declares its variables as a list, so change only the value: under - name: feedName to the exact feed name you created in Task 1; leave the - group: lab-vars entry in place:

    variables:
    - group: lab-vars
    - name: feedName
    value: 'kk-python-feed-<lab-username-suffix>'

Commit the change to main.

Grant the pipeline's build identity access to the feed so the publish step can upload: open Artifacts > your feed > gear icon (Feed settings) > Permissions > + Add users/groups, add <your-project> Build Service with the Contributor role, and save. (Project-scoped feeds sometimes grant this automatically, but setting it explicitly avoids a **403** Forbidden on publish.)<br> Navigate to Pipelines > Pipelines > New Pipeline.<br> Select Azure Repos Git (**YAML**) > python-calculator-webapp.<br> Select Existing Azure Pipelines **YAML** File, Branch main, Path /pipelines/publish-package.yml. Click Continue > Run.<br> If prompted, permit the pipeline to use the feed.<br> Confirm the steps complete (Build the package, Authenticate twine, Publish the package).<br> Rename the pipeline to publish-package.<br> Navigate to Artifacts > your feed and confirm the kk-calculator 1.0.0 package is listed.<br> Still seeing **403** Forbidden on the Publish step (mentioning a Build Service identity)? The build-service permission from step 2 is missing or not yet applied; re-check Feed settings > Permissions that <your-project> Build Service has the Contributor role, then re-run the pipeline.

![alt text](<Screenshot 2026-11-02 180708.png>)

![alt text](<Screenshot 2026-11-02 180815.png>)

## Exercise 2 – Task 1: Consume the Package with pip

In this task, you will install the package you published from the feed with pip and verify that it works.

Now, consume the package you published from another environment (the lab terminal).

Steps: Get the feed's pip connection details: in Artifacts > your feed (kk-python-feed-<lab-username-suffix>), click *Connect to feed* > pip. Copy the index **URL**, which looks like:<br> [https://pkgs.dev.azure.com/<your-org>/<project>/_packaging/<your-feed>/pypi/simple/](https://pkgs.dev.azure.com/<your-org>/<project>/_packaging/<your-feed>/pypi/simple/)

In the lab terminal, activate the pre-created virtual environment (Python, pip and venv are already installed), then install the keyring helper and the package inside the venv:

    source /root/venv/bin/activate
    pip install keyring artifacts-keyring
    pip install kk-calculator \
    --index-url "[https://pkgs.dev.azure.com/<your-org>/<project>/_packaging/<your-feed>/pypi/simple/"](https://pkgs.dev.azure.com/<your-org>/<project>/_packaging/<your-feed>/pypi/simple/")

Verify that it works (still inside the venv):

    python -c "from kkcalc import add, safe_divide; print(add(2,3), safe_divide(10,2))"
    # -> 5 5.0

Note: The first install triggers an interactive sign-in via artifacts-keyring (a device-code / browser prompt). Complete it with your lab credentials; once authenticated, pip installs kk-calculator from the feed.

![alt text](<Screenshot 2026-11-02 181252.png>)

![alt text](<Screenshot 2026-11-02 182748.png>)
![alt text](<Screenshot 2026-11-02 182826.png>)
![alt text](<Screenshot 2026-11-02 182903.png>)

Review<br> Congratulations! You have successfully completed the Package Management with Azure Artifacts lab.

In this lab, you:

Created a uniquely named Azure Artifacts feed (kk-python-feed-...)<br> Built a Python package (kk-calculator) with python -m build<br> Published it to the feed from a pipeline using TwineAuthenticate and twine upload<br> Learned how to consume the package with pip using the feed's index **URL** and artifacts-keyring<br> Azure Artifacts gives your teams a private, versioned package feed, enabling controlled sharing and reuse of internal libraries across projects.