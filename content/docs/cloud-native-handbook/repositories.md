---
title: "Repositories"
weight: 10
aliases:
- /docs/cloud-native-handbook/workflow/organizing-code-and-config/
---

# Repositories

When building on top of a cloud native platform all code and configuration should be version controlled. In Kubernetes, all deployment events are merely a result of updating a configuration file (referred to here as a Kubernetes manifest). Since all changes to code and config must be version controlled, we will use our repository as the origin of all changes to running software. This process is known as GitOps.

When implementing a GitOps process, using a seperate repository for code and config (manifests) is recommended. The purpose of the code repo is solely to produce versioned software artifacts. The purpose of the config repo is to deploy software. Seperating these concerns provides multiple benefits detailed below.

## Small Merges

Multiple environments typically correspond to multiple git branches. By keeping the concept of deployments local to config repositories (and out of code repos) you are able to handle environment promotion with smaller merges. Config repos only need to merge a reference to a new container image tag rather than merging all of the code changes across N branches.

![Promoting software aross environments](/images/develop-deploy-promotion.jpg)

## Change Trail

As a result of seperating development from deployment, any commit to a config repo represents a change to running software. This produces a legible, easily auditable change trail.

## Logical Organization of Components

Applications are commonly composed of multiple components (REST APIs, databases, etc.). Some of these components may be built in-house while others may be from vendors or open source projects. A single config repo can provide this 1:N mapping of application to its components. This avoids the problem that arises when code and config co-exist in the same repositories: which repo should contain shared components?  

## Simple Automation

If code and deployment manifests live in the same repo, additional intelligence will be needed to determine if a code change or a config change occured in order to decide whether a build is necessary. This problem commonly requires slightly different code for each repository (due to variance in directory structures and language/tooling choices).

## Takeaways

```
GitOps - It all starts with the Repo

1 Config Repo : N Code Repos

Code Repo (for developing) --produces--> Built Software

Config Repo (for deploying) --produces--> Running Software
```

