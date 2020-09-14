---
title: "Deploying"
weight: 30
---

# Deploying

All deployments should be triggered from a "config repo". This is a repository where the sole purpose is to describe runnable software (not develop it - no source code here).

![Deployment repo branching](/images/deployment-repo.jpg)

## Environment Promotion

There are two top concerns when promoting software across environments:

* **Isolation** - Seperating changes across environments
* **Parity / Variety** - Minimizing / accounting for differences

### Isolation

Isolation is addressed using branching. The top concern is to isolate non-production and production changes. To do this, two branches are used: `non-prod` and `prod`.

> *Why not use a branch per environment?*
> 
> Every environment could be mapped to its own branch. However, this increases the complexity of merges. Maintaining two long lived branches where merges flow in one direction (always fast-forward `prod` to the head of `non-prod`) greatly simplifies workflows while still minimizing risk to production.

### Parity / Variety

As software moves from non-production to production it is important maintain the maximum level of parity possible. One way to do this is by sharing a base configuration and maintaining diffs (patches) to account for environmental variance. [Kustomize](https://kustomize.io/) is the tool of choice for doing this. Environments are seperated out by directories with all commonality living in `base`:

```
base/       # Common config across all environments
staging/    # Deployed from non-prod branch (references base/)
feature-x/  # Deployed from non-prod branch (references base/)
production/ # Deployed from prod branch     (references base/)
```

A `base/` directory is used to account for commonality (parity) across environments. Changes that are to be promoted from non-production to production should be made in base (i.e. updated image version). Once the `non-prod` branch gets merged into the `prod` branch, these changes will be promoted.

Individual `<environment>/` directories are used to account for variety across environments. Variances that should always exist across environments (i.e. replica count / resource allocation) should be maintained in these directories as patches to base resources or additional resources to be added.

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

## Summary

```
* Environments seperated by directory within repo.

* Production and non-production environments seperated by branch (and directory).

* Branch merges: main --> non-prod --> prod
```
