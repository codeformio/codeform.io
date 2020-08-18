---
title: "Developing"
weight: 20
---

# Developing

All development should occur in a "code repo". This is a repository where the sole purpose is to produce deployable software. In the cloud native world, this means the goal is to push container images to a registry.

The branching strategy described here mirrors [oneflow](https://www.endoflineblog.com/oneflow-a-git-branching-model-and-workflow). We feel like it is a good fit for most teams. A single long-lived branch keeps merging simple. Git tagging triggers container image tagging. Deployment is decoupled, avoiding the complexity of maintaining environment-specific branches.

![Development repo branching](/images/development-repo.jpg)

## 1. Features 

To add new features, first create a `feature/<feature-name>` branch.

```sh
# From the main branch (git checkout main)...
git checkout -b feature/my-feature
```

Add your code changes and any documentation updates. Push the branch.

```sh
git push -u origin feature/my-feature
```

Create a Pull Request from your branch `feature/my-feature` into `main`.

Now the git hosting service you are using should require the following before enabling a merge:

* Unit tests
* End-to-end tests
* Linting
* Image build success
* Human approval

Use the following checklist when approving pull requests:

* [&check;] Code logic is easy to follow.
* [&check;] Code follows conventions.
* [&check;] Dense code has commented explanations.
* [&check;] Unit/e2e tests for feature exist.
* [&check;] User documentation is updated (if seperate from repo, add link in PR description).

Once the merge occurs, the feature branch should be automatically removed to keep the repository clean.

## 2. Releases

To prepare for releasing software (updating versions / documentation / etc), create a `release/<semantic-version>` branch.

```sh
# From the main branch (git checkout main)...
git checkout -b release/2.3.0
```

This release branch should follow [semantic versioning](https://semver.org/):

* Backwards compatible changes (same major version, a new minor version, zero'ed patch version): `2.3.0`
* Non-Backwards compatible changes (a new major version, zero'ed minor/patch versions): `3.0.0`

Once all of the documentation and version updates are done, push the branch.

```sh
git push -u origin release/2.3.0
```

All of the same automated checks that apply to feature branches should be run before the release branch can be merged. The human approval checklist should include:

* [&check;] [Changelog](https://keepachangelog.com) is updated.
* [&check;] Any version references in config/code are updated.
* [&check;] User documentation is updated.\*

> *\*A note about example snippets in documentation...*
>
> Examples of how to run the software (command line snippets, etc) should be ran manually or automated. Often these snippets go out of date. This is the perfect checkpoint for making sure they still work (as opposed to feature pull requests which happen more frequently).

To trigger the release, an admin creates a git tag on the most recent commit in the release branch. The tag should match the branch version, prefixed with a "v" (i.e. `v2.3.0`). In GitHub this is done by creating a "release" (which creates a tag). In BitBucket there is a button for creating tags. Upon creation of the tag, the same automated checks that are in place for Pull Request should be run, followed by an automated push of the built container image to a registry. The image tag should match the git tag.

The software is now released. Now merge the release branch back into main to ensure all documentation updates make their way into future releases. The release branch should be automatically deleted after merged.

> *A note about git tags...*
> 
> Git tags are used to name a commit. Unlike branches, which are intended to point to an ever-growing list of commits, tags are intended to always point at the same commit.

## 3. Patches

Patches are needed when defects are found in released software. Commonly, the main branch has progressed past the current released software when the bug is found. The goal is to quickly fix the bug without hurriedly releasing a new set of features. This is commonly referred to as a "hotfix".

To start this process, first create a new `patch/<semantic-version>` branch off of the current released tag. The semantic version should be the same as the current release, except it should have the patch version bumped.

```sh
git checkout -b patch/2.3.1 v2.3.0
```

After fixing the bug, follow the same procedure used in a release branch: create a pull request, create a tag matching the branch version, and merge the hotfix back into the main branch to ensure it does not re-appear in future releases.

## CI Systems

In a cloud native world, all tests and builds should occur in containers. This means that your CI system does not need to be very complex. The biggest complexity here will come from linking the CI system to the repository (auth with service accounts, defining branch merging requirements, etc). Prefer integrated CI systems when possible to avoid the complexity of linking these two systems (i.e. GitHub workflows, GitLab pipelines).

## Summary

```
1 Long lived branch: "main"

3 Short-lived branch types (all merge to main):
- "feature/<name>"
- "release/<semver>" (new major/minor version, patch version = 0)
- "patch/<semver>" (same major/minor version, patch version ++)

* Pull requests trigger testing & image builds.
* Tagging triggers testing & image builds/pushes.
```


