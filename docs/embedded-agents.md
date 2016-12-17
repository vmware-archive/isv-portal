# How to Build an Embedded Agent

Some integrations depend on the ability to inject code into the application container.
Examples of this include:

- Application Performance Monitoring (APM) agents
- Container-embedded API gateways
- Client-side routers

We refer to these injected components as "container-embedded agents".
[Buildpacks](buildpacks.md) provide a mechanism to inject components into the application
container image, and the `.profile.d` directory provides a way to start agents before or
alongside the customer application.

- [Agent Injection with the meta-buildpack](https://github.com/guidowb/meta-buildpack)
- [Using .profile.d](http://docs.pivotal.io/pivotalcf/devguide/deploy-apps/deploy-app.html#profiled)
