# How to Build a Managed Service

A managed service is a service deployed on Cloud Foundry infrastructure
(in other words, the same IaaS that your particular Cloud Foundry instance
is deployed on) by the same orchestration tool, [bosh](http://bosh.io).
Offering your software as a managed service means that your PCF customers
will not have to learn different ways to deploy, manage, and monitor
different components of their application platform.

For bosh to manage your service, the major components needed are:

- **Packages** that can be installed on PCF stemcells to create virtual machine images
- **Jobs** that describe how to install, run, and remove your software
- A **Monitor** script, that describes how to monitor the health of your
service components and stop or restart them

## BOSH overview

- [BOSH Documentation](http://bosh.io/docs)
- [BOSH Problem Statement](http://bosh.io/docs/problems.html)
- [BOSH Basic Workflow](http://bosh.io/docs/basic-workflow.html)

## Creating a BOSH Release

- [Creating a Release](http://bosh.io/docs/create-release.html)
- [Defining your Jobs](http://bosh.io/docs/jobs.html)
- [Defining your VMs](http://bosh.io/docs/vm-struct.html)
- [Monitoring the Health of your Service](http://bosh.io/docs/monitoring.html)

## Shortcut - Start with Docker images

If you have already packaged your service as docker images, you can emulate
a managed service deployment using the [Tile Generator](tile-generator.md)'s
support for docker-bosh packages. This feature lets you deploy pre-existing
docker images into bash managed virtual machines on the PCF infrastructure.

While this is a great, easy way to deploy your service on PCF, we don't
recommend this as a long-term, production-ready solution. There is really no
benefit of running your service in containers on the VMs, and it does have
a number of operational ("day 2") drawbacks:

- You introduce more software (docker) which needs to be kept up-to-date, and
has the potential for bugs, downtime, and security vulnerabilities.
- You can no longer take advantage of the patching capabilities of PCF for
stemcells and application dependencies, like frameworks and libraries. Instead,
you become directly responsible for managing all software that is in the docker
images you deploy.

