Lab: Version Control with Git in Azure Repos

In this lab, you will use Git in Azure Repos to:

Clone an existing repository

Save work using commits

Review change history

Work with branches (create, delete, restore)

Configure branch policies

Create and complete Pull Requests

Apply release tags to commits

Open the Azure DevOps project **URL** provided in the lab and keep it open for the rest of the tasks.

Exercise 0 – Task 1: Configure Git and Visual Studio Code In this task, you will configure Git with your identity and set up the credential helper in Visual Studio Code.

Open Visual Studio Code on the lab computer. In the terminal, run the following command to configure the credential helper: git config --global credential.helper store

Run these commands to set your Git identity (replace placeholders with your details, without < > symbols):
    git config --global user.name *<John Doe>*
    git config --global user.email <[johndoe@example.com](mailto:johndoe@example.com)>