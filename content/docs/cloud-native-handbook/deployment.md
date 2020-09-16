---
title: "Deployment"
weight: 30
aliases:
- /docs/cloud-native-handbook/workflow/deploying/
---

# Deployment

Since Kubernetes is a declarative system, the act of deploying software translates to pushing Kubernetes configuration manifests into a cluster. These manifests contain descriptions of applications that should be running (what container image, how many replicas, any environment variables, dns names, etc). The cluster will then work to make sure what was declared becomes a reality.

## Branching Strategy

All manifests should be version controlled in a git repository (referred to here as a config repo). This usually translates to a "cluster config repo" for the platform team and at least one "app config repo" for each app team. Any commit to a config repo should result in the those manifests being applied into a cluster. The following branching strategy should work for most use-cases.

![Deployment repo branching](/images/deployment-repo.jpg)

## Environment Promotion

There are two main concerns when promoting software across environments:

* **Isolation** - Seperating changes across environments
* **Parity / Variety** - Minimizing / efficiently accounting for differences between environments

### Isolation

Environment isolation is addressed using branching. On the path-to-production pull requests are used to promote changes across environments. These merges should be fast-forward-only, eliminating merge conflicts.

### Parity / Variety

As software moves from non-production to production it is important maintain the maximum level of parity possible. One way to do this is by sharing a base configuration and maintaining diffs (patches) to account for environmental variance. [Kustomize](https://kustomize.io/) is the tool of choice for doing this. Environments are seperated out by directories with all commonality living in `base`:

```
base/       # Common config across all environments
staging/    # Deployed from "staging" branch    (references base/)
production/ # Deployed from "production" branch (references base/)
```

A `base/` directory is used to account for commonality (parity) across environments. Changes that are to be promoted from non-production to production should be made in base (i.e. updated image version). As the `base/` changes make their way into environment-specific branches, these changes will be promoted.

Individual `<environment>/` directories are used to account for variances that should always exist across environments (i.e. replica count / resource allocation). These variations are usually maintained in the form of patches on the base resources.

> *Why not use staging as a base for production?*
>
> Tools like kustomize do not allow for removing resources from a common base [for the sake of simplicity](https://kubernetes-sigs.github.io/kustomize/faq/eschewedfeatures/#removal-directives). For this reason, `base/` is used to represent only the commonality across environments.

## CD System

The action of deploying can be done either through a pipeline (i.e. `kubectl apply` in a Jenkins job) or a GitOps engine (i.e. [ArgoCD](https://argoproj.github.io/argo-cd/), [FluxCD](https://fluxcd.io/)).

A GitOps engine is preferable to a pipeline because it provides active reconciliation as opposed to point-in-time reconciliation. To address config drift the engine constantly ensures that the state of a cluster matches the state of your repository.

> *The Kubernetes API and GitOps...*
> 
> The Kubernetes API Server is a REST API that allows clients to [watch for changes](https://kubernetes.io/docs/reference/using-api/api-concepts/#efficient-detection-of-changes) to resources. This allows a GitOps engine to sync API resources with corrsponding files that live in a Git repo and quickly remedy config drift.

## Repo Scope

The scope of a config repo should be determined by identifying a [bounded context](https://martinfowler.com/bliki/BoundedContext.html) (which should ideally correspond to a team of people). Using GitOps objects to point to other repositories, a tree structure emerges. The root of the tree is usually a repository that the cluster operations team manages. Commonly, all cluster-level configuration is held in another repo, maintained by the same team. Application-specific repositories are also referenced, delegating access to app teams to deploy specific resources (i.e. Deployments, Services, etc.) into whitelisted namespaces.

![Config repo tree](/images/config-repo-tree.jpg)

## Takeaways

```
* Environments seperated by directory within repo.

* Staging/Production/etc. environments seperated by branch (and directory).

* Fast-forward branch merges: staging --> prod
```
