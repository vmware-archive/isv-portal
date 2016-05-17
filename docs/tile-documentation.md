# Tile Documentation Template

This document offers a template for preparing documentation for partner services that appear on PivNet. The documentation will be posted on the front page of [http://docs.pivotal.io](http://docs.pivotal.io)
under “Partner Services for Pivotal Cloud Foundry.”

While the specifics will vary depending on the product, we have provided a basic blueprint below. At minimum, documentation should include #1 (Overview) and #2 (Installing/Configuring).

For a good example of a partner service doc, see the
[JFrog Artifactory docs](http://docs.pivotal.io/jfrog/index.html).

If you have questions or want to collaborate on drafting the documentation, feel free to hop on our Slack channel #pcf-docs. We’re always happy to help!

## Overview

General overview of Partner Product. What does it do? What are its features?

Key Features

- Feature one
- Feature two
- Feature three

## Partner Service Broker

A Service Broker allows Cloud Foundry applications to bind to services and consume the services easily from App Manager UI or command line. The Partner Service Broker will enable you to use one or more Partner accounts and is deployed as a Java Application on Cloud Foundry. The Broker exposes the Partner service on the Cloud Foundry Marketplace and allows users to directly create a service instance and bind it to their applications either from the Pivotal Apps Manager Console or from the command line. 

The PCF (Pivotal Cloud Foundry) Tile for Partner installs the Partner Service Broker as an application and registers it as a Service Broker on Cloud Foundry and exposes its service plans on the Marketplace.  This makes the installation and subsequent use of Partner on your Cloud Foundry applications simple and easy. 

If trial license available => Customers interested in using Partner can obtain a 60 day free trial license from edit link here.

## Product Snapshot

Current Partner Tile for Pivotal Cloud Foundry Details:

- Version:
- Release Date: 
- Software components versions: Partner product version
- Compatible Ops Manager Version(s): 1.5.x, 1.6.x
- Compatible Elastic Runtime Version(s): 1.4.x, 1.5.x, 1.6.x

## Requirements

(or Prerequisites, Packaging Dependencies for Offline Buildpacks, etc.)

Provide any general or specific requirements here. A general requirement might be something like, “An AppDynamics account.” A specific requirement might be something like, “Packaging Dependencies for Offline Buildpacks.”

## Limitations

Any known limitations.

## Feedback

Please provide any bugs, feature requests, or questions to the Pivotal Cloud Foundry Feedback list.

## Installing / Configuring the Tile

This topic provides instructions for how to install and configure the tile. Typically this includes procedures for how to download the tile from PivNet, install it on Ops Manager, configure the tile, and do any required third-party configuration. Screenshots should be provided where necessary. Consult the following format:

### Install via Pivotal Ops Manager
 
- Download the product file from Pivotal Network (edit link).
- Upload the product file to your Ops Manager installation.
- Click Add next to the uploaded product description in the Ops Manager Available Products view to add this product to your staging area.
- Click the newly added tile to review any configurable options.
- Click Apply Changes to install the service.

### Upgrading to the Latest Version

If there are any specific instructions for upgrading the tile, you can include those here. If the procedures are complicated, create a new Upgrading topic.

### Configuring the Partner Tile 

(add snapshots for each step when possible or add details as required)

- Login into Pivotal Ops Manager 
- Click on “Import a Product” and import the Partner Tile
- Select the Partner option
- Click Add on the Partner Tile
- Select the Partner Tile
- Configure the Partner Tile
- Apply your changes.

On completion of Partner Tile install, check Services Marketplace in Apps Manager

- View Partner Service Plans
- Bind the Partner Service to an Application
- Check the service or dashboard for the partner for more data…

## Other Configurations / Third-Party Configurations

Provide information for specific configurations like configuring for HTTP proxy, or doing any necessary configurations on a third-party service portal.

## Using the Tile

This topic provides instructions for how to use the tile. Typically this includes 
procedures for how to perform the different functions offered by the service. Screenshots should be provided where necessary. You can also include information about Architecture here if necessary.

## Troubleshooting

This topic provides troubleshooting information for known errors, following the Symptom/Explanation format used here:
[http://docs.pivotal.io/p-identity/okta/troubleshooting.html](http://docs.pivotal.io/p-identity/okta/troubleshooting.html)

## Release Notes

Include the release notes as the final topic, following the format below. For reference, see the
[JFrog Artifactory Release Notes](http://docs.pivotal.io/jfrog/release.html).

- Version number:
- Release date:

Features included in this release:

- First feature.
- Second feature.
- Third feature.
