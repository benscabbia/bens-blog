---
layout: post
title: Migrating Source Control
categories: programming
tags: [source control, git, tfs]
excerpt_separator: <!--more-->
---

Over the last few weeks, I have been responsible in migrating our companies source control system from tfs to git! This post - if you like - is a checklist on how I was able to encourage, remove uncertainty and persuade the company to migrate!

<!--more-->
My task was to migrate our existing TFS system to git. I tried to make this guide source-control agnostic, but it was written with a TFS -> git migration in mind, so I have included useful links related to this when applicable.

The migration checklist is effectively broken down in four main sections:
* **Infrastructure** - You have to find out how to set things up and where these things will live.
* **Repository Data** - This step is to focus on only the current repository and how it will be ported across to the new repository
* **Teams & Workflows** - A new source control system will require training, but also a change in workflow!
* **Concrete Steps** - A step by step guide based on the first three sections!

---

## Infrastructure
* **Installation of source control** clients on all users machines
  * **Configurations**, can you automate this via a package manager?
  * **Is the tooling good enough** in your IDE for all users or are GUIs necessary?
    * Identify which GUI clients vs CLI
* **Where to put your repository**
  * In the [cloud or keep it inhouse](https://docs.microsoft.com/en-us/azure/devops/user-guide/about-azure-devops-services-tfs?view=vsts) on some server?
    * you should setup a repo [in the cloud](https://azure.microsoft.com/en-gb/services/devops/repos/) and in-house and weigh up the pros and cons
  * [Configuration options](https://docs.microsoft.com/en-us/azure/devops/repos/git/git-config?view=vsts&amp%3Btabs=visual-studio&tabs=visual-studio) and **make it strict**!
    * the azure devops, for instance, enables you to set user permissions on who and where one can push code to
    * if you use a workflow like gitflow, see if your tools lock down the master branch for only those who deploy!
    * identify how your source control system can reject pull requests if tests fails etc ([Azure devops](https://docs.microsoft.com/en-us/azure/devops/repos/git/pull-requests?view=vsts&amp%3Btabs=new-nav&tabs=new-nav))
* **Backup procedure**
  * An important one to keep everyone calm. Ensure you understand the existing backup procedure and identify how you will [backup the new repo](https://stackoverflow.com/questions/5578270/fully-backup-a-git-repo)
* **CI Pipelines**
  * Do you require to update your existing CI workflows?
* Identify a **day and time** to make the switch
  * You'll do this last once you have completed all other sections. If you have a solid plan, you can keep pushing ;)

---

## Repository Data
* Do you migrate your old **source control history** to the new one?
  * [is it possible](https://docs.microsoft.com/en-us/azure/devops/repos/git/import-from-tfvc?view=vsts&tabs=new-nav)? Are there limitations? Are there risks?
  * Do it and make sure you can!
* Identify **which projects** you want to migrate across
  * Exclude binaries if you can!
  * look at [gitignore](https://github.com/github/gitignore) or equivalents!
* Once you know every project that you want to migrate across, identify if you want a **single** **repository** or split into **multiple repositories**

---

## Teams and Workflows
* **Training and tutorials**
  * how many will need training?
  * do you need a local expert(s) in each team? If so, they need extra training!
  * creates guides, but ensure you cover how they connect to the repository
* **Source control workflows**
  * centralized, gitflow (git) etc - pros and cons and ensure you understand it
  * Branching naming strategy?
    * we went with gitflow, this is an [amazing explanation](https://www.atlassian.com/git/tutorials/comparing-workflows). Make sure you can visualize your team(s) using this!
* **Your team and other teams**
  * My company had a number of different teams using source controls in various ways. Ensure you can concretely put forward a workflow for each i.e.
    * team A on branch A (if gitflow), and team B on branch B. Everyone in B can only push to B etc.

---

## Concrete Actions
These are the steps that each time must take in order to migrate. We have multiple teams using source control, so I make a doc for each time with the steps they will have to make. Here is an example how my document looked:

1. **Create new repository** ->  doc/instructions on how to do this
  1. Repo configuration -> doc/instructions on how you do this
  2. Migrating history -> doc/instructions

2. **Setup repository workflow** - If you're using a more elaborate workflow, like gitflow:
  1. Create the branches -> doc/instructions on how you do this
  2. Configure the branches -> doc/instructions on how you setup branch policies
    1. This should configure all steps for the workflow

3. **Client Installations and configurations**
  1. Step by step guide on how to configure a client

Hopefully, by having structure and providing your manager with a step-by-step guide, it will make the transition less risky! Good luck!

---

## Other useful links:
* [Microsoft Migration checklist](https://docs.microsoft.com/en-us/azure/devops/learn/git/centralized-to-git)
* Git tutorials:
  * [Minimum you need to work with git](http://michaeljswart.com/2018/07/the-bare-minimum-you-need-to-know-to-work-with-git/)
  * [Hands on experience](https://learngitbranching.js.org/)
  * [Git book](https://git-scm.com/book/en/v2)
* Git workflows - [podcast](https://www.codingblocks.net/podcast/comparing-git-workflows/)!
