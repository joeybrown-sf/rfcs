# Meta

[meta]: #meta

- Name: ArtifactHub.io Integration
- Start Date: 2024-01-16
- Author(s): @joeybrown-sf
- Status: Draft <!-- Acceptable values: Draft, Approved, On Hold, Superseded -->
- RFC Pull Request: (leave blank)
- CNB Pull Request: (leave blank)
- CNB Issue: (leave blank)
- Supersedes: (put "N/A" unless this replaces an existing RFC, then link to that RFC)

# Summary

[summary]: #summary

This RFC proposes that the Buildpacks project integrates and augments the
bespoke [registry service](https://registry.buildpacks.io/) with [ArtifactHub.io](https://artifacthub.io/).

This RFC proposes that in 12 months, the buildpacks registry index becomes read-only. Buildpack publishers
will have 12 months to change their publish process so that buildpacks are no longer published to the registry index and
instead buildpacks are published to ArtifactHub.io. During this time, the buildpack registry will be updated so that it
reads from the buildpack index and from the ArtifactHub.io API.

This RFC proposes to shut down the buildpacks registry index completely in 18 months. This gives time for modifications
to `pack` to read from ArtifactHub.io and for `pack` users to upgrade their `pack` version. After this time, `pack` will
read only from the ArtifactHub.io API.

# Definitions

[definitions]: #definitions

### [Artifact Hub](https://github.com/artifacthub/hub)

A CNCF project that is a web-based application that enables finding, installing, and publishing Cloud Native packages.

### [ArtifactHub.io](https://artifacthub.io/)

Public instance of the Artifact Hub project that is hosted with CNCF resources and maintained by ArtifactHub project
maintainers.

### [Artifact Kind](https://github.com/artifacthub/hub/blob/master/docs/repositories.md)

Artifacts that ArtifactHub knows about. ArtifactHub supports over 20 kinds of Cloud Native artifacts, including Argo
Templates, Backstage Plugins, Helm Charts, and others.

# Motivation

[motivation]: #motivation

Buildpacks and Builders should be easily discoverable to make them more usable and approachable. Today, the
centralized [buildpacks registry](https://registry.buildpacks.io/) service is our solution to discoverability. This
registry service is a very important tool, but there is room for improvement. There are many features that
ArtifactHub.io provides that the buildpack registry does not provide. Some features include rich descriptions, usage
information, activity, dependency information, email subscriptions to packages, webhooks, related packages, and,
furthermore, it is a service that the buildpacks team does not need to maintain.

We should consider utilizing this project instead of enhancing our own registry. In this way, we can support
the CNCF community while gaining a lot of features and dedicated user experience enhancements. ArtifactHub.io can
support the Buildpack project by offloading this responsibility.

Here is an [example of a helm chart](https://artifacthub.io/packages/helm/prometheus-community/prometheus) that is
indexed on ArtifactHub.io. You can see here there are a lot of interesting features like security reports,
vulnerabilities, usage stats, and all sorts of things.

# What it is

This RFC proposes two overlapping goals.

1. Add support in Artifact Hub for component buildpacks and builders.

   Adding support in Artifact Hub is an independent operation. The Artifact Hub project could add support for buildpacks
   and builders without any integration on the CNB project side. This could cause confusion to users because
   ArtifactHub.io could be interpreted as another official buildpack registry. This could also cause additional toil for
   buildpack authors if they are burdened with publishing to two indexes. Therefore, we should not do this unless we
   plan on migrating (in at least some capacity) to treat ArtifactHub.io as an official buildpacks registry. This work
   would happen in the Artifact Hub repository. Proposal discussion can be
   found [here](https://github.com/artifacthub/hub/issues/4352).

2. Integrate our tooling to (eventually) recognize ArtifactHub.io as **the** official CNB registry

   This should happen in phases so that buildpack authors and `pack` users have minimal disruption to established
   workflows. CNB is commited to backwards compatibility, but there are sometimes good reasons to break this backwards
   compatibility. This RFC proposes a slow phase-out and eventual decommission of the buildpacks registry. More details
   of this phase-out can be found in the migration section of this document.

   ArtifactHub.io periodically scans artifact repos to index them into its database. When those are indexed, they are
   available via the ArtifactHub.io API.

# How it Works

Component buildpacks and builders will be created as new types in Artifact Hub. ArtifactHub has processes
for [claiming ownership](https://artifacthub.io/docs/topics/repositories/#ownership-claim) of packages,
establishing [verified publishers](https://artifacthub.io/docs/topics/repositories/#verified-publisher) of packages, and
establishing [official status](https://artifacthub.io/docs/topics/repositories/#ownership-claim) of packages. These
statuses will be helpful for migrating packages to ArtifactHub.io.

Buildpack publishers will be able to continue publishing to the buildpack registry index, but they will likely want
to publish
to ArtifactHub.io instead. The buildpacks registry indexer will determine the packages to reference as follows:

# Migration

[migration]: #migration

This RFC proposes two artifact types in Artifact Hub. CNB does not have a registry for Builders, so adding support in
Artifact Hub for builders is a light lift and does not require a migration. Users will be able to discover builders via
the UI and API, but because CNB tooling does not support this kind of discoverability, there is little to do on the CNB
side.

Because CNB already has a buildpack registry, there is a migration story that is necessary. The following requirements
outline the steps required. The term "requirement" is used instead of "step" because some requirements can be done in
parallel, while some have dependencies on other requirements.

### Requirement 0a - Approval in Artifact Hub for work to proceed

The [issue](https://github.com/artifacthub/hub/issues/4352) must be accepted in Artifact Hub.

### Requrement 0b - CNB Announcement

We should proactively announce our intentions to the community. There will be a slow roll-out of features but this is
ultimately a destructive change. Publishers and users will be affected. We should make this announcement so that users
know what is coming and so that there is a reference we can direct users to in documentation and tooling.

This announcement should be a living document so that as complete the various requirements we can update this document.

### Requirement 1 - Add support in Artifact Hub for component buildpacks and builders.

The work here is in Artifact Hub. This author's understanding is that work is done mostly by Artifact Hub maintainers,
but CNB maintainers should be open and available for consulting.

> [!WARNING]
> We need to ensure the Artifact
> Hub [package metadata](https://github.com/artifacthub/hub/blob/master/docs/metadata/artifacthub-pkg.yml) suffices
> for Component Buildpacks and Builders. If not, we need to ensure we create
> adequate [custom annotations](https://artifacthub.io/docs/topics/annotations/keptn/).

### Requirement 2 - Mirror buildpack artifacts from the buildpack [registry index](https://github.com/buildpacks/registry-index) to ArtifactHub.io

Establish a CNB organization in ArtifactHub.io. This organization will be the ArtifactHub.io package owner for these
packages until publishers claim ownership themselves inside the ArtifactHub.io control panel.

In order to mirror the buildpack artifacts to ArtifactHub, the CNB registry index repository will set up a directory
structure that contains a metadata file for each package version. For reference, please see
the [Inspektor Gadget repository structure](https://artifacthub.io/docs/topics/repositories/inspektor-gadgets/#inspektor-gadgets-repositories).

In ArtifactHub.io, anyone is able to publish any packages. The CNB organization
will create [package metadata](https://github.com/artifacthub/hub/blob/master/docs/metadata/artifacthub-pkg.yml) for use
by Artifact Hub. These packages will be created for existing packages and any new buildpacks.

The file structure in the buildpack registry index will look something like this:

```
path/to/packages
├── artifacthub-repo.yml
├── heroku
│   ├── dotnet
│   │   ├── 0.1.0
│   │   │   └── artifacthub-pkg.yml
│   │   ├── 0.1.1
│   │   │   └── artifacthub-pkg.yml
│   │   ├── 0.1.2
│   │   │    └── artifacthub-pkg.yml
│   │   └── ...
│   └── java
│       ├── 0.1.1
│       │   └── artifacthub-pkg.yml
│       └── ...
├── initializ-buildpacks
│   ├── go
│   │   ├── 1.0.0
│   │   │   └── artifacthub-pkg.yml
│   │   ├── 1.0.1
│   │   │   └── artifacthub-pkg.yml
│   │   ├── 1.0.2
│   │   │    └── artifacthub-pkg.yml
│   │   └── ...
│   ├── php
│   │   ├── 1.0.0
│   │   │   └── artifacthub-pkg.yml
│   │   └── ...
│   └── ...
├── paketo
│   ├── python
│   │   ├── 0.0.1
│   │   │   └── artifacthub-pkg.yml
│   │   ├── 0.1.0
│   │   │   └── artifacthub-pkg.yml
│   │   ├── 0.2.0
│   │   │    └── artifacthub-pkg.yml
│   │   └── ...
│   └── ruby
│       ├── 0.4.0
│       │   └── artifacthub-pkg.yml
│       └── ...
└── ...
```

> [!WARNING]
> At this stage, the buildpack registry API still uses the CNB registry index as its source. This means that `pack` is
> not pulling from ArtifactHub.io. Publishers should not change their publishing method at this time. There could be
> confusion if buildpack artifacts exist in ArtifactHub.io but not the registry index, because `pack` only knows about
> the packages defined in the index.

### Requirement 3 - Update the buildpacks [registry indexer](https://github.com/buildpacks/registry-api/blob/main/cmd/index-buildpacks/main.go) to use to ArtifactHub.io instead of directly from the index file structure.

At this point, ArtifactHub.io will be scanning the registry index using the new file structure outlined in the previous
step. Instead of the CNB registry service scanning this as well, the registry service will populate its database based
on calls to the ArtifactHub.io API.

When this step is complete, publishers will be able to take full owership of their publishing process. The buildpacks
registry index will continue to be read/write for 12 months, so publishers can continue to publish to the buildpacks
registry.

At this point, publishers have the option of claiming ownership of their packages and publishing to ArtifactHub.io only.
The CNB registry will read any new buildpacks that are published to ArtifactHub.io outside the CNB registry index
publishing process.

`pack` will continue reading from the CNB registry API.

At this point, publishers can publish to either the buildpack registry index or ArtifactHub.io. By the end of 12 months,
publishers should only use the ArtifactHub.io method for publishing, because the buildpack registry index will become
readonly at this point and will not accept new publishes.

### Requirement 4 - Update `pack` to pull from ArtifactHub.io

We will introduce a new flag to `pack` called `registry-schema`. The options for this flag are `cnb` and `artifacthub`.

From month 0-12, the flag will default to `cnb` and users will be warned about the upcoming behavior changes.

From month 12-18, the flag will default to `artifacthub`. On failures, users will be warned about the changes, and they
will be able to set the flag to `cnb` for a period of time.

After month 18, the flag will be deprecated and the only behavior will be the `artifacthub` schema behavior.

### Requirement 5 - Terminate the buildpacks registry service

From month 0-12, there will be only additive changes to the registry service. It will maintain full backwards
compatibility for 12 months.

From month 12-18, the registry service will be read-only. No buildpacks will be able to publish to the registry service
after 12 months, but it will still serve `GET` requests so that `pack` does not break.

After month 18, the registry service will be terminated. At this point, ArtifactHub.io will be the official source for
buildpacks.

We will update the
buildpacks [registry indexer](https://github.com/buildpacks/registry-api/blob/main/cmd/index-buildpacks/main.go) so
that in addition to utilizing the buildpacks [registry index](https://github.com/buildpacks/registry-index), the
index will also utilize the [ArtifactHub.io API](https://artifacthub.io/docs/api/openapi.yaml).

Instead of buildpack authors publishing to the [buildpack registry index](https://github.com/buildpacks/registry-index),
publishers would update metadata files in their own repositories that they own.

# Drawbacks

[drawbacks]: #drawbacks

This is a disruptive change for publishers and `pack` users.

CNB discoverability will depend upon the uptime of ArtifactHub.io.

ArtifactHub.io is in the incubating stage with CNCF.

If ArtifactHub.io goes away in the future, CNB will have to adapt.

# Alternatives

[alternatives]: #alternatives

### Do Nothing

We could leave our discoverability story alone and maintain our current implementation.

### Run our own instance of Artifact Hub

This would require time, effort, and a new investment in architecture.

### Continue to run the CNB registry indefinitely

In this alternative, we fulfil Requirements 0a-3 and part of Requirement 4. Efforts could cease when we introduce the
`pack` flag `registry-schema`.

# Prior Art

[prior-art]: #prior-art

Helm went through this process in 2020 when it replaced its registry service, Helm Hub,
but [in 2020 the project replaced Helm Hub with Artifact Hub](https://helm.sh/blog/helm-hub-moving-to-artifact-hub/).

There are
many [repository types supported by Artifact Hub](https://github.com/artifacthub/hub/blob/master/docs/repositories.md).
