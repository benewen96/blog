---
layout: post
title:  "GitLab Part One: What is GitLab?"
date:   2019-03-21 16:55:00 +0000
categories: gitlab
author: Ben Ewen
---

Welcome to the first part of a multi-part series on GitLab. 

In my previous post, I spoke about the influence Docker has had in the DevOps community. I also briefly mentioned GitLab as one of the leaders in adopting this technology:

> "...but the most striking example to me is an open-source web-based DevOps toolset called GitLab.
GitLab embraces the idea of containerisation and all the ideas that complement it. It's utterly brilliant for the following reasons:
> * Docker registry for each repository.
> * 'Runners' that run CI/CD build jobs, that can run themselves inside Docker containers.
> * Docker at its core - GitLab can run inside Docker!"

This series will explain GitLab in depth in four parts:

1. What is GitLab? (You are here)
2. Running GitLab in Docker
3. Configuring GitLab for your needs
4. Security in GitLab

So, without further ado;

## What is GitLab?

Let's borrow a couple of quotes from the GitLab website:

> "GitLab is a single application for the entire software development lifecycle. From project planning and source code management to CI/CD, monitoring, and security."

> "GitLab started as an open source project to help teams collaborate on software development. By delivering new functionality at an industry-leading pace, GitLab now provides a single application for the entire software development and operations lifecycle. GitLab provides everything you need to Manage, Plan, Create, Verify, Package, Release, Configure, Monitor, and Secure your applications."

So hold on - GitLab isn't just an open source and/or self-hosted version of GitHub? Absolutely not! It is probably the most complete DevOps tool that I have ever come across.

GitLab breaks itself down into ten fundamental components:

- Manage
- Plan
- Create
- Verify
- Package
- Secure
- Release
- Configure
- Monitor
- Defend

As you can see - it is not just about source control, but rather a complete DevOps solution. Forget having a toolchain consisting of multiple products chained together, GitLab includes everything you need to run a successful DevOps team.

And if you think I'm trying to sell you something... it's completely free and open-source! Okay, they do have a managed solution, but you can use most of it's features completely free-of-charge if you host it yourself.

Let's vist these ten areas to understand the value that GitLab delivers:

### Manage

#### Authentication

GitLab supports as standard LDAP (Lightweight Directory Authentication Protocol). This means that you can use Microsoft's Active Directory, Apple's Open Directory, Open LDAP etc. to on-board users and to authenticate with the GitLab platform.

#### Audit

Auditing is made easy on GitLab, enabling you to keep track of changes on your instance. Broadly speaking, they are split down into:

>* "Events scoped to the group or project, used by group / project managers to look up who made what change."
>* "Instance events scoped to the whole GitLab instance, used by your Compliance team to perform formal audits."

#### Analytics

Ever tried to estimate how long a project should take to complete? Realised how difficult it is to actually estimate? GitLab Cycle Analytics uses real data from your projects to provide accurate predictions on how long each phase of your project will take. It tracks your team's velocity and provides insights that help pinpoint areas of improvement.

### Plan

#### Project Management

GitLab offers all the tools required whether you work in a lean methodology or an agile methodology. It includes issue tracking that you can link to code changes; visualise and track the status of builds, testing, security scans, and delivery.

#### Kanban Boards

Not much to say here, GitLab offers Kanban boards for all of your projects, letting you stick to one DevOps tool rather than having another just for your Kanban boards.

#### Time Tracking

I really like this one - GitLab includes time tracking commands such as `/estimate` and `/spend` so that you can tightly couple time tracking with the day-to-day commands you use with source control. No more forgetting how much time you spent on something!

### Create

#### Source Code Management

Clue's in the name - GitLab adopts the industry-standard distributed version of Git. If you're migrating from GitHub, your developers won't even notice the difference - promise!

It's also worth mentioning that all your Git repositories from GitHub can be imported straight into GitLab, along with all issues and metadata.

#### Code Review

What's a good development team without code reviews? GitLab offers a complete code review process. Features include, commenting, inline improvements, and approval processes.

#### Wiki

How big's your `README.md`?

Seriously, I love a wiki, and GitLab provides one built in with every repository! Share that information in the familiar Markdown syntax and GitLab will render a lovely wiki page for you. Oh - and it's source controlled so you have a history of all changes you can easily revert to.

#### Web IDE

Whenever I see "Web IDE", I scream a little inside. I was always left dissapointed by GitHub's online editor. GitLab doesn't misuse "IDE" here. You can see everything you'd need. Syntax highlighting, IntelliSense for common languages, code reviews, merge requests, and even your CI jobs.

### Verify

#### Continuous Integration (CI)

GitLab implements continuous integration beautifully. As you push code to your repositories, GitLab automatically picks it up, builds and tests it with unit and integration testing, before merging the changes.

#### Code Quality

What I really enjoy with the GitLab ecosystem is the adoption of existing technologies. Rather than re-inventing the wheel, GitLab will use existing tools. I believe this has a lot to do with how open-source it is - GitLab themselves recognise and utilise the power of open-source software.

This is shown with their code quality feature, it is adopted from [Code Climate](https://codeclimate.com/) engines.

#### Performance Testing

GitLab enables you to measure the impact of upcoming code changes before they go live with [Sitespeed.io](https://www.sitespeed.io/). All of this is shown within the merge request window, meaning you don't have to leave GitLab to obtain these metrics.

### Package

#### Docker Registry

Remember back in the last post when I mentioned GitLab has it's own Docker registry for each repository? This is that feature!

I have had success pushing to the Docker registry as part of the CI process, meaning that the images can be easily re-deployed for review apps (see the Release section for more information on review apps).

#### NPM Registry

Just as with Docker images, GitLab can also provide it's own NPM registry for your projects. This feature is still in development, so look out for it in the future.

### Secure

#### SAST & DAST

GitLab offers comprehensive static and dynamic application security testing so that you can be confident your application is secure. It can first statically analyse your code to find known code-level exploits. Additionally, it can analyse you live application dynamically for run-time level exploits and report back.

#### Dependency Scanning

A big anxiety for DevOps teams is the vulnerabilities that can come with external packages. One [notable example](https://blog.npmjs.org/post/180565383195/details-about-the-event-stream-incident) was the NPM `event-stream` package to be compromised in handing over private keys for Bitcoin wallets. 

GitLab will help mitigate this risk by automatically scanning any third-party packages used in your application.

### Release

#### Continuous Delivery (CD)

#### Release Orchestration

#### Pages

#### Review Apps

#### Incremental Rollout


### Configure

#### Auto DevOps

### Monitor


### Defend