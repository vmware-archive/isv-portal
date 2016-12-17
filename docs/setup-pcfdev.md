# PCF Developer Environment

Pivotal provides a light-weight (vagrant packaged) instance of PCF with some
basic services as a free product named PCF Dev. This is a great environment
to develop and test everything that runs in the Cloud Foundry Elastic Runtime.

If your integration includes managed services, you will also need an instance
of bosh that can manage virtual machines and bosh releases for you.
[bosh-lite](https://github.com/cloudfoundry/bosh-lite) works well for that
purpose.

Between these two components, you will have everything you need to develop
tiles, except for Pivotal's Ops Manager. But if you followed the recommended
[staged development approach](development.md) you will not need an actual full
PCF environment until the later phases of your development.

## Setting up PCF Dev

- [Try PCF on your Local Workstation](http://pivotal.io/platform/pcf-tutorials/getting-started-with-pivotal-cloud-foundry-dev/introduction)

## Setting up bosh-lite

- [Install bosh-lite](https://github.com/cloudfoundry/bosh-lite)

!!! note "Note:"
	For this type of development environment, you only need bosh-lite
	itself to deploy managed service releases. You do **not** need to follow the
	instructions to Deploy Cloud Foundry in bosh-lite, as Cloud Foundry is
	provided by the PCF Dev installation above.
