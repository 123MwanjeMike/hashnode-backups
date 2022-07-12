## Project management with Github Issues.

Photo by [Roman Synkevych](https://unsplash.com/@synkevych?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)

Software projects might be different from other types of projects in terms of visibility, complexity, conformity, and flexibility as Fred Brooks points out; but one thing in common with all projects is change. Essentially, project management is change management and this is what project management tools help us with. Tools like Trello, ClickUp, Jira are built to enable us best plan and track our work — herein, you’ll learn to do the same with Github Issues.

### But first, why Github Issues?

Given that Github is a leading software development platform, most project management tools support integration with it as a way to bridge management and development teams. However, the underlying problem still remains: this is not where the code is and developers have to leave the platform to update their work item status on these project management tools. In addition, automation on these is not as easy and the pricing of these tools is maybe high.

### What are Github Issues.

[Github Issues](https://docs.github.com/en/issues) is a collection of Github products that include labels and milestones, issues, project boards, and projects(beta). All these products can be used together to easily plan and track your progress on your software project. The scope of this article will be limited to issues and Github project boards and that’s what will be covered in the tutorial section.

### Github project boards

*Project boards on GitHub help you organize and prioritize your work. You can create project boards for specific feature work, comprehensive roadmaps, or even release checklists. With project boards, you have the flexibility to create customized workflows that suit your needs.*

Depending on your needs, you can either create repository, user-owned, or organization-wide project boards all with different scope. A card on a project board represents either an issue, a pull request or a note. These cards contain metadata about the issue or pull request the like labels, assignee(s), the status, and who opened it. Github project boards also provides you with templates to create a new board some of which come with built in automation. They include basic kanban, automated kanban, automated kanban with review, and bug triage. Now, let’s see how this works.

### Tutorial

**Step 1:** Create the project board.First, [create a repository project board](https://docs.github.com/en/issues/organizing-your-work-with-project-boards/managing-project-boards/creating-a-project-board#creating-a-repository-project-board) from the Automated Kanban template under the **project template** drop-down. I’ll call my board User App.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1657651187123/JvOYFihtE.png)

You can rename the *To do* Column to *Product Backlog* and add a column called *Sprint Backlog* with the To do preset, and click the **Reopened** checkbox under **Move issues here when…** and move it after the *Product Backlog* column.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1657651188926/dDrr8IJDCi.png)

**Step 2:** Add notes to the project board  
Next, we will [add notes](https://docs.github.com/en/issues/organizing-your-work-with-project-boards/tracking-work-with-project-boards/adding-notes-to-a-project-board#adding-notes-to-a-project-board) to our project board under the *Product Backlog* list and after [convert them to issues](https://docs.github.com/en/issues/organizing-your-work-with-project-boards/tracking-work-with-project-boards/adding-notes-to-a-project-board#converting-a-note-to-an-issue). These are features we plan to work on.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1657651190786/I6grqdDFl.png)

**Step 3:** Assign issues.  
Now that we have a good understanding of what we plan to do, we shall start embarking on the issues. So say, for the first sprint we are working on *Signup with email* and *Login with email and password*. We shall move these cards to the *Sprint Backlog* column and [assign each issue](https://docs.github.com/en/issues/tracking-your-work-with-issues/assigning-issues-and-pull-requests-to-other-github-users#assigning-an-individual-issue-or-pull-request) to an individual.

**Step 4:** Update *In Progress* column automation.  
We will now [update automation](https://docs.github.com/en/issues/organizing-your-work-with-project-boards/managing-project-boards/configuring-automation-for-project-boards) for the *In Progress* column move pull requests here when newly added, reopened, and pending approval by reviewer. This will enable us know what tasks are currently being worked upon.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1657651192742/VkBCjS4PN.png)

**Moment of truth  
**With the above setup complete, we are now good to go. When an assignee opens a PR(Pull Request) and [links](https://docs.github.com/en/issues/tracking-your-work-with-issues/linking-a-pull-request-to-an-issue#manually-linking-a-pull-request-to-an-issue) it to the assigned issue, we shall see a new card under the *In Progress* column that corresponds to the PR, and when the PR is merged, both cards (for the PR and the linked issue) will automatically be moved to the *Done* column.

<iframe src="https://www.youtube.com/embed/djJI9dJtqZw?feature=oembed" width="700" height="393" frameborder="0" scrolling="no"></iframe>

Github Project Board: [https://github.com/123MwanjeMike/user-app/projects/1](https://github.com/123MwanjeMike/user-app/projects/1)

### Final thoughts.

This is only the tip of the iceberg that can be done with Github Issues and with good mastery of the Github project management products, you are spoilt for choice.  
One only challenge I’ve found with Github Issues is that clients and management may have little or totally no knowledge of how to use these products. In such cases, you want to rethink the solution to get a workaround.