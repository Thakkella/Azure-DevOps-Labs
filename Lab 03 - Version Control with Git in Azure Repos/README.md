# Lab: Version Control with Git in Azure Repos

In this lab, you will use Git in Azure Repos to:

Clone an existing repository

Save work using commits

Review change history

Work with branches (create, delete, restore)

Configure branch policies

Create and complete Pull Requests

Apply release tags to commits

Open the Azure DevOps project **URL** provided in the lab and keep it open for the rest of the tasks.

## Exercise 0 – Task 1: Configure Git and Visual Studio Code In this task, you will configure Git with your identity and set up the credential helper in Visual Studio Code.

Open Visual Studio Code on the lab computer. In the terminal, run the following command to configure the credential helper: 

    git config --global credential.helper store

Run these commands to set your Git identity (replace placeholders with your details, without < > symbols):

    git config --global user.name *<John Doe>*
    git config --global user.email <[johndoe@example.com](mailto:johndoe@example.com)>

![alt text](<Screenshot 2026-10-30 123641.png>)

## Exercise 0 – Task 2: Import the eShopOnWeb Repository In this task, you will import the eShopOnWeb repository from GitHub into the repository that already exists in your pre-provisioned project, and then set main as the default branch.

Important: which repository to use. Your project already contains one empty repository named exactly the same as your project: kk-devops-dev-***. Import into that repository. Do not create a new repository and do not type a different name in the import dialog. Later exercises refer to the path /kk-devops-dev-***/src/Web/Program.cs.

Import the Repository In your browser, open the Azure DevOps project **URL** provided in the lab credentials. Inside the project, go to Repos > Files. Confirm the repository selector at the top shows kk-devops-dev-*** and that the repository is empty. In the *Import a repository* section, click *Import*. In the Import a Git Repository window, leave Source type as Git and paste this into Clone **URL**: [https://github.com/MicrosoftLearning/eShopOnWeb.git](https://github.com/MicrosoftLearning/eShopOnWeb.git)

Click *Import*. The import runs in the background and can take up to a minute. Wait until the file list appears (refresh the page if needed) before continuing. Set Main as Default Branch Go to Repos > Branches. Notice that the import brought in several branches (copilot/*, dependabot/*, main, and others), and that main is not the default branch. The default is whichever branch sorts first. This step is required, not optional. Hover over the main branch. Click the ellipsis (…) on the right. Click *Set as default branch*.

![alt text](<Screenshot 2026-10-30 123939.png>)

## Exercise 1: Clone an Existing Repository In this task, you will clone the repository from Azure Repos to your local machine using Visual Studio Code.

Before cloning, set up your **SSH** key:

Open a terminal and generate an **SSH** key pair with: 

    ssh-keygen -t rsa -b **4096** -C *ci-agent* -N "" -f ~/.ssh/id_rsa

Copy the public key output from: 

    cat ~/.ssh/id_rsa.pub

In Azure DevOps, go to User Settings located on the top right, beside your profile logo → **SSH** Public Keys (under Security) → Add your public key. Clone the repository:

In the Azure DevOps portal, open your project. In the vertical navigation pane, select the Repos icon. In the upper-right corner of the repository pane, click *Clone*. On the Clone Repository panel, select **SSH** and click *Copy to clipboard* to copy the clone **URL**. Close the Clone Repository panel. Switch to Visual Studio Code. Click the View menu and select *Command Palette*. At the Command Palette prompt, run: Git: Clone Paste the copied **SSH** repository **URL** into the text box and press Enter. In the Select Folder dialog, navigate to your home directory (/root/). After cloning completes, when prompted, click *Open* to open the cloned repository in VS Code. Note: If the Command Palette doesn't show Git: Clone, make sure the Git extension is active in VS Code.

![alt text](<Screenshot 2026-10-30 124359.png>)

![alt text](<Screenshot 2026-10-30 124447.png>)

## Exercise 2 – Task 1: Commit Changes In this task, you will make a change to a file, commit it locally, and push it to the remote repository.

Note: VS Code runs in your browser in this lab, so keyboard shortcuts such as Ctrl+S and Ctrl+Enter are captured by the browser and may not reach the editor. Use the menu and on-screen buttons described below instead.

In VS Code, select the **EXPLORER** tab. Navigate to and open the file: /kk-devops-dev-***/src/Web/Program.cs On the first line, add the following comment: // My first change

Save the file using the menu: File > Save. Switch to the **SOURCE** **CONTROL** tab in VS Code. Verify that Git has detected the change to Program.cs. In the message textbox at the top of the **SOURCE** **CONTROL** pane, type: My commit from VS Code

Click the ✓ Commit button at the top of the **SOURCE** **CONTROL** pane to commit locally. If prompted to auto-stage and commit, click *Always*. In the lower-left corner of VS Code, click the Synchronize Changes icon (circle with two arrows, showing 1). If prompted to proceed, click *OK* to push and pull commits to/from origin/main. The validation only passes once the commit has been pushed to Azure Repos, so do not skip step 9.

![alt text](<Screenshot 2026-10-30 125545.png>)

![alt text](<Screenshot 2026-10-30 125611.png>)

## Exercise 2 – Task 2 & 3: Review Commits and Stage Changes In this task, you will verify your commit in Azure DevOps, then practice staging only specific files before committing.

Note: Use File > Save and the on-screen **SOURCE** **CONTROL** buttons because browser-intercepted shortcuts such as Ctrl+S may not reach the editor in this lab environment.

### Review Commits

Switch to the Azure DevOps portal in your browser. In the Repos section, select *Commits*. Verify your *My commit from VS Code* appears at the top of the list. Stage Changes. Switch back to Visual Studio Code. Open /kk-devops-dev-***/src/Web/Program.cs and change the first comment to: //My second change

Save the file (File > Save).

Navigate to /kk-devops-dev-***/src/Web/Constants.cs and add on the first line: // My third change

Save the file (File > Save).

Switch to the **SOURCE** **CONTROL** tab. Hover over the Program.cs entry and click the *+* (stage) icon on the right to stage only that file. In the message textbox, type: Added comments to Program.cs

Click the ellipsis (…) at the top of the **SOURCE** **CONTROL** tab. In the dropdown, select *Commit*, then in the cascading menu select *Commit Staged*. Click the Synchronize Changes button in the lower-left corner. If prompted, click *OK* to push and pull commits. The validation checks the remote repository, so it only passes after step 12 has pushed your commit. Constants.cs should remain unstaged and uncommitted.

![alt text](<Screenshot 2026-10-30 125742.png>)

![alt text](<Screenshot 2026-10-30 130258.png>)

## Exercise 3: Review History
In this task, you will compare file versions in VS Code and browse the repository state at a historical commit in Azure DevOps.

Compare Files in VS Code
In the SOURCE CONTROL tab of VS Code, select Constants.cs (the unstaged version).
Review the comparison view showing what changed vs. the committed version.
Browse Commits in Azure DevOps
Switch to the Azure DevOps portal and go to the Commits pane under Repos.
Scroll down to find the "My commit from VS Code" entry.
Hover over it to reveal the ellipsis (…) on the right side.
Click the ellipsis and select "Browse Files".
Review the repository state at that specific commit. This shows all files as they existed at the time of that commit.

![alt text](<Screenshot 2026-10-30 130355.png>)

## Exercise 4 – Task 1: Create a New Branch in Local Repository
In this task, you will create a new local branch named dev from main using Visual Studio Code.

Switch to Visual Studio Code.
In the SOURCE CONTROL tab, click "main" in the lower-left corner of the VS Code window.
In the pop-up window, select "+ Create new branch from…"
In the "Select a ref to create the branch from" textbox, select main as the reference branch.
In the "Branch name" textbox, type:
   dev

Press Enter.
You are automatically switched to the dev branch.
Note: You can also create a branch from the Command Palette using Git: Create Branch From....

![alt text](<Screenshot 2026-10-30 130721.png>)

## Exercise 4 – Task 2: Delete and Restore a Branch In this task, you will publish the dev branch to the remote, delete it from Azure DevOps, and then restore it from the deleted branches list.

Publish and Delete the Branch In VS Code, in the lower-left corner, click the Publish changes icon (to the right of the dev label) to push the dev branch to the remote. Switch to the Azure DevOps portal and go to Repos > Branches. On the Mine tab, verify the dev branch is listed. Hover over the dev branch entry to reveal the ellipsis (…). Click the ellipsis and select *Delete branch*. When prompted, click *Delete* to confirm. Restore the Deleted Branch Switch back to VS Code, click *dev* in the lower-left corner. Note that two dev branches are now listed (one local, one remote). Go back to Azure DevOps, switch to the All tab of Branches. In the Search branch name textbox, type dev. Find the Deleted branches section and locate the dev entry. Hover over it to reveal the ellipsis (…) and click it. Select *Restore branch*.

![alt text](<Screenshot 2026-10-30 130933.png>)

![alt text](<Screenshot 2026-10-30 131205.png>)

![alt text](<Screenshot 2026-10-30 131344.png>)

## Exercise 4 – Task 3 & 4: Configure and Test Branch Policies In this task, you will protect the main branch with policies that require a reviewer and a linked work item, then verify that a direct commit is blocked.

Configure Branch Policies on Main In Azure DevOps, go to Repos > Branches. On the Mine tab, hover over the main branch and click the ellipsis (…). Select *Branch Policies*. Enable *Require minimum number of reviewers* and set the value to 1. Check *Allow requestors to approve their own changes*. Enable *Check for linked work items* and leave it set to *Required*. Test the Branch Policy In Azure DevOps, go to Repos > Files and make sure main is selected. Navigate to /kk-devops-dev-***/src/Web/Program.cs and select it. On the first line, add this comment: // Testing main branch policy

Click *Commit > Commit*. Observe the warning: changes to main branch can only be done using Pull Request. Click *Cancel* to skip the commit.

![alt text](<Screenshot 2026-10-30 131903.png>)

![alt text](<Screenshot 2026-10-30 132153.png>)

![alt text](<Screenshot 2026-10-30 132238.png>)

## Exercise 4 – Task 5: Create and Complete a Pull Request In this task, you will create a work item, commit a change on the dev branch, raise a Pull Request, satisfy all policy requirements, and complete the merge into main.

Create a Work Item In Azure DevOps, go to Boards > Work Items. Click *+ New Work Item > Product Backlog Item*. Set the title to: Testing my first PR

Click *Save*. Make a Change on the Dev Branch Go to Repos > Files and switch to the dev branch. Navigate to /kk-devops-dev-***/src/Web/Program.cs. On the first line, change the comment to: // Testing my first PR

Click *Commit > Commit* (leave the default commit message). In the pop-up, click *Create a Pull Request*. Complete the Pull Request In the New pull request tab, leave defaults and click *Create*. Note the failed/pending requirements from branch policies. On the right side, click the *+* next to *Work Items* and link the Testing my first PR work item. Open the Files tab to review proposed changes. Go back to the Overview tab and click *Approve* (top-right). Once all requirements turn green, click *Complete*. On the Complete Pull Request tab: Merge Type: *Merge (no fast forward)* Check *Complete associated work item…* Click *Complete Merge*.

![alt text](<Screenshot 2026-10-30 132932.png>)

![alt text](<Screenshot 2026-10-30 133030.png>)

![alt text](<Screenshot 2026-10-30 133230.png>)

![alt text](<Screenshot 2026-10-30 135106.png>)

![alt text](<Screenshot 2026-10-30 135327.png>)

![alt text](<Screenshot 2026-10-30 135409.png>)

## Exercise 4 – Task 6: Apply a Release Tag In this task, you will create a Git tag on the main branch to mark it as a beta release.

In the Azure DevOps portal, go to the Repos section. Select *Tags*. In the Tags pane, click *New tag*. In the Create a tag panel: Name: v1.1.0-beta Based on: leave main selected Description: Beta release v1.1.0 Click *Create*.

![alt text](<Screenshot 2026-10-30 135538.png>)

## Exercise 5: Remove Branch Policies (Cleanup) In this task, you will remove the branch policies applied to main to avoid blocking future lab exercises.

In Azure DevOps, go to Repos > Branches. On the Mine tab, hover over the main branch and click the ellipsis (…). Select *Branch Policies*. Disable the option: *Require minimum number of reviewers*. Disable the option: *Check for linked work items*. This cleanup step removes the policies to prevent them from blocking future lab exercises.

### Review - You have completed the Version Control with Git in Azure Repos lab by:

Configuring Git and cloning the eShopOnWeb repository locally 
Making and committing changes with descriptive messages, then pushing them to Azure Repos 
Staging selective files before committing 
Reviewing commit history and browsing repository state at historical commits 
Creating, publishing, deleting, and restoring branches 
Configuring branch policies (minimum reviewers and linked work items) 
Creating a Pull Request, linking a work item, approving, and completing the merge Applying a release tag (v1.1.0-beta) to the main branch 
Cleaning up branch policies for future labs