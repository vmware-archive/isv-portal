# Stages of Integration

When integrating third-party software with Cloud Foundry, the integration
typically progresses through the same set of stages. We recommend this
staged approach because it enables early feedback on the value and the
design of the integration, which helps make better decisions about future
stages.

For service type integrations, the typical stages of integration are:

1. User-Provided Service
2. Brokered Service
3. Managed Service
4. Dynamic Service

Each of these is described in more detail below. In general, user-experience
and production-readiness improves as the integration
progresses through the stages. But none of the later stages is required.
Integration can stop and be declared complete (enough) after any of these.

For non-service integrations (such as applications or buildpacks), a similar
staged integration approach is often possible and desirable.

<a name="ups"></a> 
## Stage 1. User-Provided Service

Either your software is available as a SaaS-offering, or you already have a
way to install software on-premise at a customer site. Or also likely, your
customer already has your software, is now adopting PCF, and wants to be
able to consume your software from applications deployed on PCF.

In most cases, customers can immediately start consuming your software from
their PCF applications through the user-provided service mechanism available
in Cloud Foundry. Tell them to create a user-provided service in their
application org and space using the command:

```bash
cf create-user-provided-service <my-service-name> -p <credentials>'
```

or `cf cups` for short. The `<credentials>` argument should be a valid JSON
string that contains the URL and credentials necessary to connect to your
externally-deployed service.

By doing this, application developers can bind
to your service and write all code necessary to access it through a Cloud
Foundry service binding. It is a great way to determine what information
needs to be passed in the credential structure (useful in later integration
stages), verify that the integration works, and develop a test application
that can continue to be used for later stages. And from the application
developer perspective, once this works, later stages will not require any
further code changes. User-provided service bindings are fully compatible with
brokered service bindings.

<a name="broker"></a> 
## Stage 2. Brokered Service

The first real improvement in user experience is achieved by creating a
[Service Broker](service-brokers.md) for your service. Service brokers allow
you to expose your services and plans in the `cf marketplace`, from which
your customers can then create their own service instances with a simple
`cf create-service`. It eliminates the need for them to know the URLs and
credentials for your services - those are managed automatically by the
broker instead.

Building a broker for a (still) externally deployed service is generally
a good way to be able to publish a first tile that adds real value for
customers who have both your software and PCF.

<a name="managed"></a> 
## Stage 3. Managed Service

The next step is to get your service to actually be deployed on PCF rather
than externally as you've traditionally done it. This is usually one of the
more involved stages as you will have to change your packaging to allow your
service components to be deployed by [bosh](http://bosh.io) onto the PCF
infrastructure.

You will have to learn about stemcells, bosh releases, and manifests. You
will also have to decide how your service maps to virtual machines and how
persistent storage is managed.

For a Minimal Viable Product (MVP) version of a managed service, we typically
recommend that you aim for a single, shared service instance, and don't yet
worry too much about High Availability of this instance. This stage is mostly
about getting the bosh packaging, deployment, and monitoring working
correctly.

<a name="dynamic"></a> 
## Stage 3b. High Availability

Once you have a managed service, you may decide to prioritize either
[dynamic provisioning](#dynamic) of service instances, *or* making your
single shared service instance more highly available.

When properly configured, bosh monitors and restarts any failing processes
and virtual machines that are part of your service deployment. But to
further increase availability, you will have to think about spreading your
resources across multiple availability zones or even regions, and replicating
your persistent storage across those as well.

<a name="dynamic"></a> 
## Stage 4. Dynamic Service

All prior stages assume that you have a single instance of your software
deployed. That instance can be multi-tenant, and it can possibly be manually
scaled to accomodate many concurrent applications. But for real production
deployments, most of your customers will want dedicated instances of your
service for each application.

With bosh 2.0, Cloud Foundry has the ability to dynamically provision a
completely new bosh deployment of your software for each service instance.
This is known as "dynamic" or "on-demand" service provisioning.

## Stage 4b. High Availability

If you hadn't already in Stage 3b, the final step would be to consider how
each of your dynamically provisioned service instances can be made more
highly available.
