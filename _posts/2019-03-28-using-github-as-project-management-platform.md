---
layout: post
title: "Using GitHub as your Agile Project management platform"
description: "Using GitHub as a Kanban board and track the progress of the development of your project"
category: Software engineering
tags: [Agile, Scrum]
---

Recently, the dev-team to which I belong decided to abandon Jira in favor of GitHub as the sole platform for the project management.
You can argue that Jira and GitHub are not really comparable, and I'd agree with you if not because - let's be honest to each other - the former is also very very slow and clunky. As a matter of fact you will eventually find yourself dealing with unannounced UI/UX updates and a constant feeling of slowness followed by a sense of frustrastion, which is like a trademark when using a _mastodon_. What is more, you have to pay to use it and you need another set of credentials to remember/store. 

Okay, enough talking I guess, let's see how to use your GitHub projects, milestones and issues to create a lean Agile workflow. All it takes is a bit of strictness and a self imposed protocol to follow. 

---

## Agile concepts

You might know the basic concepts that lay behind an agile methodology like _Scrum_, but if it's not your case or if you simply feel the urge to refresh some of your memories I'll try to recap the basic bits that compose a Scrum-like process.

### Epics

An epic is **a large body of work that can be broken down into a number of smaller stories**,in our case "issues" in GitHub. Epics often encompass multiple teams, on multiple projects, and can even be tracked on multiple boards.

> Epics are almost always delivered over a set of sprints. As a team learns more about an epic through development and customer feedback, user stories will be added and removed as necessary. That’s the key with agile epics: Scope is flexible, based on customer feedback and team cadence. 

&rarr; [Further readings about Epics](https://www.atlassian.com/agile/project-management/epics)

### Stories (a.k.a. _User Stories_)

This is probably the core concept behind each agile methodology, and it's ofter misunderstood because we forget about the **Users**. We can define a User Story as a unit of value for the end users.

You will probably have already read something like that when it comes to User Story:

```
As a <type of user>, I want <some goal> so that <some reason>.
```

A story can be also broken down to its essential bits using a set of ordered **Tasks** that will eventually give the completion of the story itself as their final product. In our team we refer to this process as _"breaking down a story"_.

&rarr; [Further readings about User Stories](https://www.atlassian.com/agile/project-management/user-stories)

### Spikes

There are situations when a story cannot be estimated, which means that either the story is perhaps too large and complex or the team hasn't really understood the nature of the problem that has to be solved. Hence, the story can't be be scheduled directly because the team doesn't know if it will be possible to complete the tasks in the assigned time frame. 

In these cases, the developers might want to build a functional or technical experiment to better understand and then to be able to give an estimation of the complexity of the story. This could involve some research time:

- looking into something for a day;
- looking for alternatives;
- doing some googling or wandering on StackOverflow for hours;
- doing some experiments with some other library or software package;
- considering alternative refactoring paths.

Normally, we call all this process a **"spike"**. Moreover, you can address them as _research spikes_, or _architectural spikes_, or _refactoring spikes_.

```
In order to <achieve some goal> <a system or persona> needs to <some action>.
```

> A spike as an investment the Product Owner makes to figure out what needs to be built and how the team is going to build it — in advance of actually building it. 

&rarr; [Further readings about Spikes](https://www.leadingagile.com/2016/09/whats-a-spike-who-should-enter-it-how-to-word-it/)

### Sprints

A Sprint is basically a fixed-length iteration cycle, that usually lasts circa 2 weeks. The outcome of a Sprint is usually a working piece of software that will be added to the main "trunk". We can think of it like an single entry in a release changelog. You will probably set a milestone that will eventually represent a new release to be published, and that milestone will be made of several sprints that will add feature over feature to your final product.

&rarr; [Further readings about Sprints](https://www.atlassian.com/agile/scrum/sprints)

### Sprint Backlog

Essentially, the backlog is a collection of stories that have to be finished and tested before the end of the Sprint. At the beginning of each Sprint the team will decide which stories have to be pulled out from the Global Backlog. Moreover, the stories that belongs to a Sprint should be ordered by their priority and tagged with a number of points, representing the complexity of the story itself. 

### Global Backlog

The global backlog is a prioritised list of stories that has not been scheduled to be completed. As new work comes in, it gets added to the global backlog and it possibly gets an assigned priority and maybe a label. In an Agile workflow, the development team pulls from the backlog as opposed to a manager pushing work onto the developers. **The goal is to keep the backlog as small and as organized as possible**.

## Being Agile in GitHub

Our goal is to take advantage of the simplicity and lightness of GitHub to map the Agile concepts depicted before into its features. 

<img alt="Agile methodologies using GitHub" src="/assets/img/github_scrum_agile_dev.png">

### A GitHub issue for each user story

You can simply create a new issue per story, and to do so I would suggest to use a proper set of labels to mark its priority, its type and who's going to work on that story.

### Labels

#### Theme

`Theme` labels allow virtual matrix of stories across epics:

  - `Theme: strategic goals`
  - `Theme: product modules`
  - `Theme: project phases`

#### Epic

`Epic` labels allow grouping body of work that can be broken down into specific stories:

  - `Epic: business flow`

#### Type   

`type` labels allow you to annotate backlog items:

  - `type: User Story`
  - `type: Product Infra`
  - `type: Team Infra`
  - `type: Spike`
  - `type: Bug`
  - `type: Refactor`
  - `type: Improvement`

#### Point

`point` labels allow you to assign velocity points to backlog items:

  - `point: 1`
  - `point: 2`
  - `point: 3`
  - `point: 5`
  - `point: 8`
  - `point: 13`
  - `point: 21`

### A Sprint will be tracked by using a Milestone and a Project

Projects, in GitHub, corresponds to Kanban boards, where you'll find at least three columns to put your stories into:

- To do
- In progress
- Done

You will use a project board, like a Kanban board, moving stories around like in a physical board. It will also give you a visual representation of your Sprint. Now, in order to collect all the stories into sprints you can create milestones naming them after each Sprint. Doing so, you'll be able to prepare new Sprints moving the selected stories into a specific milestone.

### Issues not related to any project will represent the global backlog

It's as simple as that, you put all the new stories into the global backlog just using GitHub's issues, without assigning them to a specific milestone. This set of unassigned and unrelated issues will represent your Global Backlog. You will move each story into a milestone during your Sprint Plannings. That's easy, isn't it?

---

# The final Recap

| Global Backlog | Sprint Backlog | In Progress | In Review | Done |  
| :---: | :---: | :---: | :---: | :---: |  
| open<br>unmilestoned<br>unassigned | open<br>milestoned<br>unassigned | open<br>milestoned<br>assigned | open<br>milestoned<br>assigned<br>pull request reference | closed |  

## Resources

- [What is Scrum?](https://www.atlassian.com/agile/scrum)
- [GitHub Issues Can be Agile if You Do it Right](https://zube.io/blog/agile-project-management-workflow-for-github-issues/)
- [GitHub scrum workflow](https://github.com/jvandemo/github-scrum-workflow)
- [Scrum Ceremonies](https://www.atlassian.com/agile/scrum/ceremonies)
- [Technical User Stories – What, When, and How?](http://rgalen.com/agile-training-news/2013/11/10/technical-user-stories-what-when-and-how)
- [GitHub - Scrum template](https://github.com/GlanceLeads/scrum-template)
- [Epical epic. How to write epics in Agile](https://www.scrumdesk.com/epic-epically/)
