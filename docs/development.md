# Tile Development Process

The ultimate deliverable of a PCF integration is almost always a "tile", a
Pivotal Cloud Foundry installation package that is delivered through our
[marketplace](http://network.pivotal.io) and installed through Pivotal's
Ops Manager. But when developing an integration, it is advisable to start
with smaller components of that tile, as it allows you to iterate on those
components much faster. Our recommendation is to approach development in
phases:

1. Develop and test each of the components to be deployed individually
2. Describe your tile and generate the tile artifacts
3. Test the generated artifacts individually
4. Test deployment of the complete tile
5. Implement Continuous Integration (CI) for the complete tile

If you follow this approach, you may not have a dependency on a complete
PCF installation until step 4, and your iterations on the components will
be much faster than if you attempt to test them through actual deployment
to PCF.

Each of the development phases is described in more detail below.

<a name="components"></a> 
## Develop the Tile Components

Tiles are a packaging format to deliver ISV software to PCF customers. Most
tiles contain one or more of the following types of components:

- Service Brokers
- Managed Services
- Buildpacks
- Applications

It is much more efficient to develop and test these components individually
than it is to test them through tile deployment. So before you start generating
and deploying tiles, *always* make sure that the components you are deploying
already work, individually and in whatever combination you intend to deploy
them as a tile.

In most cases, you will not need a full PCF installation to complete these
early phases. You can set up a
[light-weight PCF development environment](setup-pcfdev.md) on your laptop
or desktop, possibly including bosh-lite if your are developing managed
services.

<a name="tile-generator"></a> 
## Describe and Generate your Tile

*After* your components are in working order, download and install the
[Tile Generator](tile-generator.md) and follow the instructions to describe
and generate your actual tile. This is where you list all the components that
are to be included, add an icon and a description, and have the option to add
forms for values that are to be configured by the PCF operator at installation
time.

<a name="test-errands"></a> 
## Test the Deploy and Delete Errands

This is the first step for which you will need a
[complete PCF deployment](setup-pcf.md). But before you deploy your complete
tile to Ops Manager, you can manually deploy your individual
[components](#components) and test the errands created by the Tile Generator
using the [pcf utility](pcf-command.md#errands). Doing this is significantly
faster than testing the errands through Ops Manager and bosh, as bosh will
run each errand in a newly created virtual machine.

<a name="deploy"></a> 
## Deploy and Test your Tile

After you have verified that all individual [components](#components) and
[errands](#test-errands) work, you are ready to deploy your tile as your customers
would, by uploading it to Ops Manager, installing and configuring the tile,
and having Ops Manager apply your changes to the PCF deployment.

The only things you should be testing in this phaase are the things that
could not be tested in earlier ones:

- The appearance of the tile forms in Ops Manager
- The upgrade/migration steps from one version of a tile to another (if applicable)

Everything else should be working exactly as it did in prior steps.
