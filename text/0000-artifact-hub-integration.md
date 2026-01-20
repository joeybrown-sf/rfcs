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

This RFC proposes adopting ArtifactHub.io as the officially supported platform for component buildpack and builder (index) publishing and discoverability.

Leveraging ArtifactHub.io as the officially supported platform for buildpack index publishing and discoverability allows the CNB team to provide a better experience for buildpack authors and CNB community members. The CNB team will have less services to maintain, hopefully providing more bandwidth to focus more on the core CNB spec and lifecycle. The CNB team will not be burdened by the maintenance of these essential, yet orthogonal services.

ArtifactHub.io will serve as a community registry for builders as well. (There is no specification for builder URIs or distribution methods for builders today, so there is no migration to orchestrate.)

# Definitions

[definitions]: #definitions

__[Artifact Hub](https://github.com/artifacthub/hub)__ - CNCF project that is a web-based application that enables finding, installing, and publishing Cloud Native packages.

__[ArtifactHub.io](https://artifacthub.io/)__ - Public good instance of the ArtifactHub project that is hosted with CNCF resources and maintained by ArtifactHub project maintainers.

__[ArtifactHub Kind](https://github.com/artifacthub/hub/blob/master/docs/repositories.md)__ - OCI artifacts that Artifact Hub knows about. Artifact Hub supports over 20 kinds of Cloud Native artifacts, including Argo Templates, Backstage Plugins, Helm Charts, and others. This RFC proposes 2 new Artifacts _Kinds_: __cnb-buildpack__ & __cnb-builder__.

__[ArtifactHub Repository](https://artifacthub.github.io/hub/api/#/Repositories)__ - Git repository owned by cnb-buildpack/cnb-buildpack authors that contains metadata consumable by ArtifactHub.io. [Here is an example](https://github.com/joeybrown-sf/h-buildpacks/tree/main/catalog/paketo-buildpacks/go). Repositories can be at the buildpack level or organization level. That is to say, the following are both valid repository URLS:
- https://github.com/joeybrown-sf/h-buildpacks/catalog/paketo-buildpacks
- https://github.com/joeybrown-sf/h-buildpacks/catalog/paketo-buildpacks/go

# Motivation

[motivation]: #motivation

### As Discoverability Platform
Component Buildpacks and Builders should be easily discoverable to make them more usable and approachable. Today, the [buildpacks registry app](https://registry.buildpacks.io/) is CNB bespoke solution to buildpack discoverability. It has basic metadata, but there are many features and metadata attributes that ArtifactHub.io could provide, but that the buildpack registry does not provide in its current state. Some features and metadata include:
- Rich descriptions
- Usage information
- Dependency information
- Email subscriptions, webhooks to buildpack version updates
- Related packages
- Hyperlinks

This integration would provide a robust platform for both builder and buildpack discoverability.

Furthermore, CNB could relinquish the maintenance burden of the services and processes that support the buildpacks registry app.

### As Buildpack and Builder Index Publishing Platform
ArtifactHub.io integration will provide buildpack authors more control over their buildpack metadata. Authors will write buildpack index metadata to a git repository that they have permission to write to (ArtifactHub.io tracks these repositories for updates).

ArtifactHub.io will allow buildpack authors more metadata attributes and an intentional design with embeddable widgets. ArtifactHub.io supports a hierarchy of organizations and users.

# What it is

This RFC proposes changes that would affect both the Artifact Hub project and the CNB project.

### Add support in Artifact Hub for component buildpacks and builders.

Adding support for cnb-buildpack and cnb-builder kinds in Artifact Hub is a discrete operation and could technically happen without official CNB integration or endorsement. This being said, adding support without a commitment on the CNB side could cause confusion to users because ArtifactHub.io could be interpreted as another official buildpack index registry. 

CNB support should migrate completely to ArtifactHub.io as the one officially supported community buildpack index publishing platform so that buildpack and builder authors should not be burdened with publishing to two indexes registries.

Supporting buildpack and builder artifact hubs in ArtifactHub.io requires [minimal work from the CNB team¹](#minimal-work). This work will happen in the Artifact Hub repository. New artifact kind proposals are introduced to ArtifactHub via github issues. I have an open dialog with ArtifactHub.io maintainers on this [thread](https://github.com/artifacthub/hub/issues/4352).

### Integrate the CNB Registry to use ArtifactHub.io as the one source of data for the repo

The CNB Registry will be backwards compatible in the fact that it will continue to adhere completely to the spec. Buildpack authors are the only members of the CNB community with work they must do.

Because the process that reads data from ArtifactHub.io and writes it to the CNB Registry will be cabable of mapping the ArtifactHub repository name & cnb-buildpack name to the [CNB Registry] buildpack ID with high trust, platforms and CNB apps will require no changes.

# How it Works

## CNB Registry Publishing Deprecated

CNB will __deprecate index publishing capabilities__ at the CNB Registry ([registry-index](https://github.com/buildpacks/registry-index)) and adopt ArtifactHub.io as the _one_ officially supported index publishing platform. CNB Registry becomes readonly from the Buildpack author's perspective. The CNB Registry metadata will be populated from an API endpoint in ArtifactHub.io that provides all buildpack metadata in ArtifactHub.io.
- Publishing directly to CNB Registry will no longer be supported.
- The CNB Registry remains the [spec-compliant](https://github.com/buildpacks/spec/blob/main/extensions/buildpack-registry.md) reference implementation and source for community buildpacks.

Buildpack authors will have a period of about 6 months to update their index publishing process from the CNB Registry to ArtifactHub.io. After the migration period, buildpack authors will not be able to publish to the CNB Registry. Any buildpack metadata in the CNB Registry that was not migrated to ArtifactHub.io will be deleted in the CNB Registry. After the migration period, the CNB Registry will be populated with metadata sourced only from ArtifactHub.io.

Today, buildpack authors submit a specially formatted github Issue to the CNB Registry in order to publish index metadata. To make buidpack metadata available for ArtifactHub.io consumption, buildpack authors must write metadata files to a git repository tracked by ArtifactHub.io.

During the migration period, the CNB Registry will continue to function as a publishing platform. It will use buildpack metadata already present in the CNB Registry in addition to metadata provided by ArtifactHub.io. In the event that buildpack package metadata exists in both sources, the ArtifactHub.io data will take precedence over CNB Registry data. This gives buildpack authors time to change from one git workflow to another git workflow.

After the migration CNB will archive the [Registry Namespace](https://github.com/buildpacks/registry-namespaces), which is used by the CNB Registry for ownership and authorization.

## CNB Registry Search App Deprecated

CNB will __deprecate the Registry App__ ([registry.buildpacks.io](https://registry.buildpacks.io)) and direct users to ArtifactHub.io.

## ArtifactHub.io Artifact Kinds Introduced

Component buildpacks and builders will be introduced as new kinds in Artifact Hub. 

## ArtifactHub.io Export Endpoint Added

An [integration endpoint](https://artifacthub.github.io/hub/api/#/Integrations) will be added to ArtifactHub.io to easily export all the `cnb-buildpack` data with one API call.

A process will run on a regular cadence to keep the CNB Registry up to date with community buildpack index metadata ingested from ArtifactHub.io.

## CNB Registry Data Ingestion Process

The process that exports data from ArtifactHub.io to CNB Registry will be able to trust that
```
ArtifactHub.io repository name + ArtifactHub.io buildpack name = CNB Registry ID
```

This trust is possible because I have [claimed several organizations and buildpacks](#artifacthub-claims) on behalf of community contributors in order to maintain trust between ArtifactHub.io and the CNB Registry. This allows us to maintain trust without requiring a mapping of ArtifactHub.io attributes to CNB attributes.

## Pack Changes

Also after the migration, we will remove the experimental buildpack publishing commands from `pack`. If users are on an old version of pack and execute this command, it will open a github issue that will be immediately closed with `Not Planned`, a short explanation, and a link to the docs.

## CNB App Changes

No changes necessary*. The CNB Registry is backwards compatible.

*If you rely on a buildpack that is not migrated to ArtifactHub.io, and you reference that buildpack via a Registry URI, then your build will likely fail in the future.

## CNB Platform Changes

No changes necessary*. The CNB Registry is backwards compatible.

*If your developers rely on a buildpack that is not migrated to ArtifactHub.io, and you or they reference that buildpack via a Registry URI, then their builds will likely fail in the future.

# Migration

[migration]: #migration

This section guides the migration of buildpack index metadata from the CNB Registry to ArtifactHub.io. This section explicitly does not deal with buider index metadata because it's a new feature, not a migration.

### Step 1 - ArtifactHub Issue Approval (does not include release)

The maintainers at ArtifactHub.io are [ready to integrate](https://github.com/artifacthub/hub/issues/4352) when we let them know we want to proceed.

### Step 2 - Make CNB Registry ArtifactHub.io Aware

Implement a scheduled process that hits the (non-existant) ArtifactHub.io integration endpoint and writes metadata to the CNB Registry.

The endpoint won't exist yet, but we can build the process now so that we start sucking up ArtifactHub.io data as soon as the `cnb-buildpack` artifact kind is released in ArtifactHub.io.

### Step 3 - The following will happen close together:

#### ArtifactHub.io Introduces Support for `cnb-buildpacks`

We prepared for this in Step 2. As soon as any cnb-buidpack packages are present in ArtifactHub.io, they will be included in the CNB Registry.

We have not completely hammered out the schema for cnb-buildpack metadata in [`artifacthub-pkg.yml`](https://github.com/artifacthub/hub/blob/master/docs/metadata/artifacthub-pkg.yml), but I have a proof of concept and have determined it will accommodate cnb-buildpacks.

#### CNB Documentation

Documentation is updated to reference ArtifactHub.io as the officially supported buildpack index publishing platform.

We can have a page that describes the migration efforts if we want.

Leave references to registry.buildpacks.io in place during the migration period. These references will be changed after the migration is done.

#### Publish Blog Posts
1. Introduce the move to ArtifactHub.io (audience is entire CNB community)
2. Migration walkthrough (audience is buildpack authors). I will provide reference tooling and material.
3. Builder artifact kind introduction (audience is platform operators that maintain builders)
4. ArtifactHub.io blog post for CNCF community? Published by ArtifactHub.io? Maybe.

#### Add ArtifactHub.io Banner to registry.buildpacks.io

registry.buildpacks.io will be going away at the end of the migration deadline. Add a banner so people know that

#### Step 4 - Buildpack Author Migration with 6 Month Deadline

The 6 month countdown begins now.

At this point, Buildpack Authors are equipped to migrate to ArtifactHub.io as their buildpack index publishing platform. Authors will have 6 months, during which they can publish indexes to either or both indexes (CNB Registry and ArtifactHub.io), where ArtifactHub.io metadata takes precedence.

During this time, the github bot that reads CNB Registry Issues and adds package metadata will be modified so that on a certain date 6 months in the future, its behavior changes to automatically close (not-planned) metadata issues.

During this time, the experimental pack feature for publishing buildpacks is removed.

During this time, we'll update the export process from Step 2 so that at a certain date 6 months in the future, it's behavior changes to source metadata only from ArtifactHub.io and delete any metadata that does not exist in ArtifactHub.io.

#### Step 5 - CNB Service Termination

Add redirect to registry.buildpacks.io to ArtifactHub.io. Can be automatic redirect or static page with more detail, hyperlinks, etc. Archive the [registry-api](https://github.com/buildpacks/registry-api) repo. Shut down the API and services.

Update documentation, CNB website to no longer reference registry.buildpacks.io.

Archive the [registry-namespaces](https://github.com/buildpacks/registry-namespaces) repo.

We will figure out how to handle unclaimed repository names or organizations in ArtifactHub.io in a way that maintains community trust.

#### End State of Migration

At this point, CNB no longer maintains a buildpack index publishing platform or a buildpack discoverability platform.

CNB continues to maintain the CNB Registry as a reference implementation/community good instance.

Folks will use ArtifactHub.io for buildpack discoverability.

Folks will use ArtifactHub.io for buildpack index and package metadata publishing.

# Drawbacks

[drawbacks]: #drawbacks

This is a disruptive change for buildpack publishers. The ownership and publishing model are different. I think 6 months is a good time frame, but I'm not a buildpack publisher. It should be a one-time change.

We will be relying and trusting a 3rd party for accurate and safe buildpack metadata. ArtifactHub.io is an incubating project in the CNCF and there are several other CNCF projects that leverage it.

We will be relying and trusting a 3rd party for metadata availability. The CNB Registry will be regularly exporting this data, so ArtifactHub.io will not be necessarily on platforms' hotpath. Platforms could choose to put ArtifactHub.io on their builds hotpath, but that is not recommended.

ArtifactHub.io does not have an SLA or status page. There is a #artifact-hub channel in the CNCF Slack and github issues that can be used to communicate with maintainers. ArtifactHub has a [99 CLO Score](https://clomonitor.io/projects/cncf/artifact-hub) (funny enough the core maintainers of ArtifactHub.io also run the [CLOMonitor](https://github.com/cncf/clomonitor) project).

# Alternatives

[alternatives]: #alternatives

### Do Nothing

We could leave our discoverability story alone and maintain our current implementation.

### Make the CNB Registry Better

We could invest time and effort to make our existing services better.

### Run our own instance of Artifact Hub

This is possible, but we would continue to be responsible for services orthogonal to core CNB spec and lifecycle focus.

### Do not implicitly map ArtifactHub.io repository name to CNB namespace

Implicitly mapping repository names to CNB namespaces gives us trust that current buildpack namespace owners remain the same during the migration so that we do not need an explicit "trust map" between ArtifactHub.io and CNB. This map would belong to the CNB team and is just one more moving part that would ideally be avoided.

### <a href="#indefinite-registry"></a> Continue to run the CNB registry indefinitely as the official source for `pack`

In this alternative, we fulfill the requirements through Requirement 5. Efforts could cease after the registry indexer
pulls sources from ArtifactHub.io. The CNB registry would continue to operate in a fully backwards-compatible manner and
there would be no necessary changes to `pack`.

# Prior Art

[prior-art]: #prior-art

Helm went through this process in 2020 when it [replaced its registry service, Helm Hub](https://helm.sh/blog/helm-hub-moving-to-artifact-hub/). Here is Helm Hub's [deprecation timeline](https://github.com/helm/charts#deprecation-timeline).

There are many [repository types supported by Artifact Hub](https://github.com/artifacthub/hub/blob/master/docs/repositories.md).

# Spec. Changes

[spec-changes]: #spec-changes

None. This RFC proposes no changes to the [buildpack-registry spec](https://github.com/buildpacks/spec/blob/main/extensions/buildpack-registry.md).

# Proof of Concept

I am running an instance of ArtifactHub where I proved out concepts for Buildpacks and Builders. This is not the final implementation and is subject to change, but does give a feel for cnb-buildpacks and cnb-builders in the ArtifactHub UI.

It can be found [here](https://stark-eyrie-63979-07c4c3c5c6d9.herokuapp.com/).

The metadata found [here](https://github.com/joeybrown-sf/h-buildpacks/tree/main/catalog) populates that ArtifactHub instance.

I created the metadata by running this [cli tool](https://github.com/joeybrown-sf/ah-integration) that reads metadata from the CNB Registry and the buildpack.toml file present on the images referenced by the CNB Registry metadata. This tool will be available for buildpack authors and I will reference it in my migration walkthrough blog post.

# Footnotes

#### <a id="minimal-work"></a> 1. There is minimal work required from CNB team to implement the cnb-buildpack & cnb-builder kinds in Artifact Hub. The following quote is from one of the ArtifactHub maintainers:
> Please note that we'd rather to take care ourselves of the changes to add support for the new artifact kind to AH once we've agreed on the plan.

[source](https://github.com/artifacthub/hub/issues/4352#issuecomment-3751449070)

#### <a id="artifact-claims"></a> 2. I have personally claimed several organizations and repository names in ArtifactHub.io.

Here is what I have claimed:
- Buildpacks.io Organization
  - Buildpacks.io repository (name: `buildpacks`)
  - dmikusa repository (name: `dmikusa`)
  - fagiani repository (name: `fagiani`)
  - hone repository (name: `hone`)
  - Initializ Buildpacks repository (name: `initializ-buildpacks`)
  - jkutner repository (name: `jkutner`)
  - Laraboot Buildpacks repository (name: `laraboot-buildpacks`)
  - tomh4 repository (name: `tomh4`)
- Heroku Organization
  - Heroku Buildpacks (name: `heroku`)
- Paketo Organization
  - Paketo Buildpacks (name: `paketo-buildpacks`)
  - Paketo Community Buildpacks (name: `paketo-community`)

To build this list of organizations and namespaces I claimed in ArtifactHub.io, I checked out some cnb heruistics gleaned from the CNB Registry to try to snag popular buildpack namespaces and namespaces for which I know there are references and maintainers github handles that I recognized. I'm sorry if I missed your buildpack. Please feel free to let me know and I'm happy to claim it on your behalf until you have time to claim it yourself. I didn't want to claim all buildpacks in the CNB Registry because this is an opprotunity for us to clean stuff up and squatting all namespaces in ArtifactHub.io feels bad.

I would like to pass off ownership of these organizations and repository names in ArtifactHub.io as soon as possible. The custody of the organization and repository names is critical for maintaining trust in buildpacks.
