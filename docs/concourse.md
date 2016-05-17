# Concourse Continuous Integration

Cloud Foundry is a fast moving platform as we are constantly extending and
enhancing it. When you integrate your software with Cloud Foundry, it is
important to make sure that your integration continues to work with every
new release of the platform. A great way to ensure that is to set up a CI
pipeline for your tile against a PCF deployment that is constantly updated
with the latest Alpha release of the platform.

Our tool of choice for setting up CI is [concourse](http://concourse.ci/).
While you are of course free to use whetever system you are familiar with,
our tools and documentation are built to make concourse CI as easy as
pssoble.

<a name="server"></a> 
## Setting up a Concourse Server

You will need a concourse server to host your pipeline. If you partner with
us, we have servers that can host your pipeline, and S3 storage that can be
used to transfer artifacts to and from your servers. If you choose to set
up your own, instructions can be found here:

- [Setting up concourse](http://concourse.ci/setting-up.html)

<a name="pipeline"></a> 
## Creating a Concourse Pipeline for your Tile

A typical CI pipeline for a tile consists of the following jobs:

- Build the tile
- Deploy it to PCF
- Run a set of deployment tests to verify that it deployed and works correctly
- Remove it from PCF

You describe this pipeline in a pipeline.yml file that is then uploaded to the
concourse server. [Tile Generator](tile-generator.md) contains a sample
pipeline that you can clone for your own tile. We are working on automating
the process of generating a pipeline template for you.

<a name="pool"></a> 
## Setting up PCF for your CI Pipeline

Pivotal partners who have us host their pipeline have access to a pool of PCF
instances that are managed by us and are regularly updated with the latest
(pre-)release versions of PCF. If you set up your own concourse server, you
will have to target your pipeline at a [PCF instance you have setup](setup-pcf.md).

Concourse has a resource type to manage a pool of resources that are shared
between pipelines, which is what we use to serialize PCF access between the
partner pipelines that run on our concourse server.
