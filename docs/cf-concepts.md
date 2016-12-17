# Cloud Foundry Concepts

There are many ways to integrate products with Cloud Foundry.
The right one for each product depends on what the product does, and how
customer applications consume it. To determine the best way to integrate your
product, you'll need a good understanding of Cloud Foundry concepts
like applications, containers, services, brokers, and buildpacks.

This page provides a collection of links to documentation for the most relevant
concepts. If you prefer to learn through guided training,
[ask us](mailto:mjoseph@pivotal.io) about available training options.

## General Overview

For general overview of Cloud Foundry, and the various ways to interact with it,
use the following links:

- [Cloud Foundry Subsystems](http://docs.pivotal.io/pivotalcf/concepts/overview.html)
- [Cloud Foundry Command Line Interface](http://docs.pivotal.io/pivotalcf/cf-cli/index.html)
- [Pivotal Ops Manager](http://docs.pivotal.io/pivotalcf/customizing/pcf-interface.html)
- [Pivotal Apps Manager](http://docs.pivotal.io/pivotalcf/console/index.html)

<a name="applications"></a> 
## Applications

Cloud Foundry is primarily a cloud native application platform. To understand how to
integrate your services with Cloud Foundry, it is important to understand how your
customers are using the platform to develop, deploy, and operate their applications:

- [Deploying Applications](http://docs.pivotal.io/pivotalcf/devguide/index.html)
- [Logging and Monitoring](http://docs.pivotal.io/pivotalcf/loggregator/index.html)

<a name="services"></a> 
## Services

Most value-add integrations are done by exposing your software to customer applications
as services. To understand the service concepts, and what a service integration
looks like, read the following documentation:

- [Services Overview](http://docs.pivotal.io/pivotalcf/devguide/services/index.html)
- [Custom Services](http://docs.pivotal.io/pivotalcf/services/index.html)

<a name="buildpacks"></a> 
## Buildpacks

When application code is deployed to Cloud Foundry, it is processed by a language-specific
buildpack. Language buildpacks provide a convenient integration hook for any service that
needs to inspect or embellish application code. The meta-buildpack also provides a
language-agnostic way to inject your code into the application container image.

- [Application Staging Process](http://docs.pivotal.io/pivotalcf/concepts/how-applications-are-staged.html)
- [Language Buildpacks](http://docs.pivotal.io/pivotalcf/buildpacks/index.html)
- [Meta Buildpack](https://github.com/guidowb/meta-buildpack)

<a name="agents"></a> 
## Embedded Agents

Some integrations depend on the ability to inject code into the application container.
We refer to these injected components as "container-embedded agents".
[Buildpacks](#buildpacks) provide a mechanism to inject components into the application
container image, and the `.profile.d` directory provides a way to start agents before or
alongside the customer application.

- [Agent Injection with the meta-buildpack](https://github.com/guidowb/meta-buildpack)
- [Using .profile.d](http://docs.pivotal.io/pivotalcf/devguide/deploy-apps/deploy-app.html#profiled)

<a name="nozzles"></a>
## Nozzles
Cloud Foundry's logging system, Loggregator,
has a feature called firehose. The firehose includes the combined stream of logs from all apps,
plus metrics data from CF components, and is intended to be used by operators and administrators.

A nozzle takes this data and forwards it to an external logging and/or metrics solution.

- [Loggregator system](https://github.com/cloudfoundry/loggregator)
