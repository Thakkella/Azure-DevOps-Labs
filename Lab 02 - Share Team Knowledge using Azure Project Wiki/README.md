# Lab: Share Team Knowledge using Azure Project Wiki

In this lab, you will use the Azure DevOps Project Wiki to capture and share team knowledge, creating the wiki, adding and formatting pages, linking pages, and (optionally) publishing Markdown from a repository as a code wiki.

This lab covers the following exercises and tasks:

Exercise 1: Create and Populate the Project Wiki Task 1: Create the Project Wiki Task 2: Add a Page Task 3: Format and Link Content

Exercise 2: Publish Code as Wiki Task 1: Publish a Markdown Folder as a Code Wiki

## Exercise 1 – Task 1: Create the Project Wiki

In this task, you will create the project wiki and author its home page.

The project wiki is a built-in, Git-backed wiki for your project.

Steps: In your Azure DevOps project, navigate to Overview > Wiki in the left sidebar. Click *Create project wiki*. The wiki is created with an empty home page and opens in edit mode. Enter some content for the home page (for example, paste the contents of /root/code/wiki-content/Home.md). Click *Save*. Tip – Paste raw Markdown. The Edit pane is a Markdown editor. Copy the raw file text (open it in the code editor, or cat it and copy the output), not a rendered preview. If pasting adds a blank line between each line, paste as plain text (Ctrl+Shift+V) instead. Use the Preview pane to check the formatting.

Note: Each project can have one project wiki. It is stored in a hidden Git repository named <project>.wiki.

## Exercise 1 – Task 2: Add a Page

In this task, you will add a new wiki page and confirm that it appears in the page tree.

Add a new page named Calculator Overview.

Steps: In the wiki page tree on the left, select the Home page. Add the new page using either of Azure DevOps' two options: Click New page at the bottom of the page tree to add a page at the same level as Home, or Hover over the Home page, click ⋯ (More options), and choose Add subpage to nest it under Home. For the Page title, enter Calculator Overview exactly (page titles are case-sensitive). Paste the contents of /root/code/wiki-content/Calculator-Overview.md into the editor. Click *Save* (or press Ctrl+S). Confirm that the Calculator Overview page appears in the wiki tree on the left. Important – Paste raw Markdown, not rendered text. The wiki Edit pane is a Markdown editor. Copy the raw file text (e.g., open it in the code editor, or run cat /root/code/wiki-content/Calculator-Overview.md and copy the output); do not copy from a rendered/preview view. If a blank line appears between each table row after pasting (which breaks the table into plain text), paste again as plain text (Ctrl+Shift+V), or delete the extra blank lines so the table rows are on consecutive lines. Watch the Preview pane to confirm the Structure table renders as a grid.

## Exercise 1 – Task 3: Format and Link Content

In this task, you will practice the wiki's Markdown formatting features and link pages together.

The wiki supports rich Markdown. Practice the formatting features that make team knowledge easy to consume.

Try these in your pages: Tables – Structured data (see the Calculator Overview page) Code fences – ```python blocks for examples Task lists – - [ ] checkboxes (see Onboarding.md) Page links – Link between wiki pages with standard Markdown and an absolute page path, e.g., [Calculator Overview](/Calculator-Overview) Mentions – @ to mention a teammate Table of contents – [[_TOC_]] to auto-generate one (this tag is case-sensitive) Note: Azure DevOps wiki links use standard Markdown [text](/Page-Path), not the double-bracket [[Page Name]] style. In a page path, spaces in the page title are replaced with hyphens (Calculator Overview -> /Calculator-Overview).

Add an Onboarding page and link it Create another page titled Onboarding the same way you created the Calculator Overview page (select Home, then New page or ⋯ > Add subpage), and paste the contents of /root/code/wiki-content/Onboarding.md. Open the Home page and click Edit (or press e). In the editor, add these two links on the home page: [Calculator Overview](/Calculator-Overview) [Onboarding](/Onboarding) Tip: You can also use the Markdown toolbar's link button (or Ctrl+K) and enter the display text and page path. Click Save (or press Ctrl+S). Confirm that the links render as hyperlinks and navigate between pages when clicked. Tip: Reorder pages by dragging them in the tree, and use the page revisions to see edit history.

## Exercise 2 – Task 1: Publish Code as Wiki

In this task, you will publish Markdown files stored in a Git repository as a code wiki.

Besides the project wiki, you can publish Markdown stored in a Git repository as a wiki, keeping documentation versioned alongside code.

Prerequisite: Publishing code as a wiki requires a Git repository in your project. In this task, you will create a new repository for the documentation.

Steps: Create a new Git repository. Go to Repos > Files, open the repository selector at the top of the page, and choose New repository. In the dialog, set Repository type to Git, enter a name (for example, project-docs), and click Create. In the new project-docs repository, create a folder named docs, add the Markdown files from /root/code/wiki-content/ to it, and commit to the main branch. In Overview > Wiki, open the wiki selector dropdown (top) and choose *Publish code as wiki*. In the Publish code as wiki dialog, select the repository (project-docs), branch (main), and folder (/docs). Provide a wiki name and click *Publish*. The Markdown files now appear as wiki pages, and edits made in the repo are reflected in the wiki. Project wiki vs code wiki: A project wiki is best for free-form team knowledge; a code wiki is best for documentation that should live and version with the codebase.

Congratulations! You have successfully completed the Share Team Knowledge using Azure Project Wiki lab.

In this lab, you:

Created a project wiki and authored its home page Added and formatted pages using Markdown (tables, code, task lists, page links [text](/Page-Path), mentions, [[_TOC_]]) Learned the difference between a project wiki and a code wiki, and how to publish code as wiki A well-maintained wiki centralizes team knowledge such as onboarding guides, runbooks, and architecture notes, so it is discoverable and stays current alongside your work.