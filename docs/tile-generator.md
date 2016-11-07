# Tile Generator

The Tile Generator is a tool to help you develop, package, test,
and deploy services and other add-ons to Pivotal Cloud Foundry. Tiles are the
installation package format used by Pivotal's Ops Manager to deploy these
components to both public and private cloud deployments. The tile generator
uses templates and patterns that are based on years of experience integrating
third-party services into Cloud Foundry, and eliminates much of the need for
you to have intimate knowledge of all the tools involved.

![Overview](img/tilegenerator.png)

Tile generator takes your software components, and a simple configuration file
that provides the minimal amount of information to describe and customize your
tile. It then creates everything that's required to deploy your software into
Pivotal Cloud Foundry:

- **BOSH errands** to deploy and delete your software, including blue/green
  deployments for zero-downtime upgrades
- A **BOSH release** suitable for deploying your software to the Elastic Runtime
  or open-source Cloud Foundry
- A **Pivotal Ops Manager Tile** that can be imported into Ops Manager, installed,
  configured, and deployed, including UI forms and automatic upgrades from
  previous versions
- A **Concourse pipeline configuration** to enable Continuous Integration of
  your software with the latest versions of Pivotal Cloud Foundry

Use the tile generator in combination with the [pcf utility](pcf-command.md)
to enable rapid deploy and test cycles of your software.

The current release of the tile generator supports tiles that have any
combination of the following package types:

- Cloud Foundry Applications
- Cloud Foundry Buildpacks
- Cloud Foundry Service Brokers (both inside and outside the Elastic Runtime)
- Docker images (both inside and outside the Elastic Runtime)

## Screencast

For a 7-minute introduction into what tile generator is and does, see
[this screencast](https://www.youtube.com/watch?v=_WeJbqNJWzQ).

## How to Use

1. Install the tile-generator python package.
   **Note**: tile-generator requires **Python 2**, and will *not* work with Python 3.
   We recommend using a
   [virtualenv](https://virtualenv.pypa.io/en/stable/) environment to
   avoid conflicts with other Python packages:

        virtualenv -p python2 tile-generator
        source tile-generator/bin/activate
        pip install tile-generator

    This will put the `tile` and `pcf` commands in your `PATH`.

2. Install the [BOSH CLI](https://bosh.io/docs/bosh-cli.html)

3. Then, from within the root directory of the project for which you wish to create a tile, initialize it as a tile repo (we recommend that this be a git repo, but this is not required):

        cd <your project dir>
        tile init

4. Edit the generated `tile.yml` file to define your tile (more details below)

5. Build your tile

        tile build

The generator will first create a BOSH release (in the `release` subdirectory),
then wrap that release into a Pivotal tile (in the `product` subdirectory).
If required for the installation, it will automatically pull down the latest
release version of the Cloud Foundry CLI.

Tile generator is also available pre-installed in a docker image on
[Docker Hub](https://hub.docker.com/r/cfplatformeng/tile-generator/).
This image contains the tile-generator `tile` and `pcf` commands, all the
necessary Python dependencies, as well as the BOSH CLI.

You can use this in Concourse pipelines by specifying it as the base image
for your tasks:

```
  - task: tile-build
    config:
      platform: linux
      image: cfplatformeng/tile-generator
```

Or you can derive your own docker images from this one by using it as the base
image in your Dockerfile:

```
FROM cfplatformeng/tile-generator
```

## Building the Sample

The repository includes a sample tile that exercises most of the features of the
tile generator (it is used by the CI pipeline to verify that things work correctly).
You can build this sample using the following steps:

```bash
cd sample
src/build.sh
tile build
```

!!! note "Note:"
    The sample tile includes a Python application that is re-used in several packages,
    sometimes as an app, sometimes as a service broker. One of the deployments (app3)
    uses the sample application inside a docker image that is currently only modified
    by the CI pipeline. If you modify the sample app, you will have to build your own
    docker image using the provided `Dockerfile` and change the image name in
    `sample/tile.yml` to include the modified code in app3.

## Defining your Tile

All required configuration for your tile is in the file called `tile.yml`.
`tile init` will create an initial version for you that can serve as a template.
The first section in the file describes the general properties of your tile:

```yaml
name: tile-name # By convention lowercase with dashes
icon_file: resources/icon.png
label: Brief Text for the Tile Icon
description: Longer description of the tile's purpose
```

The `icon_file` should be a 128x128 pixel image that will appear on your tile in
the Ops Manager GUI. By convention, any resources used by the tile should be
placed in the `resources` sub-directory of your repo, although this is not
mandatory. The `label` text will appear on the tile under your icon.

### Packages

Next you can specify the packages to be included in your tile. The format of
each package entry depends on the type of package you are adding.

#### Pushed Applications

Applications (including service brokers) that are being `cf push`ed into the
Elastic Runtime use the following format:

```yaml
- name: my-application
  type: app # or app-broker
  manifest:
    # any options that you would normally specify in a cf manifest.yml, including</i>
    buildpack:
    command:
    domain:
    host:
    instances:
    memory:
    path:
    env:
    services:
  health_check: none                 # optional
  configurable_persistence: true     # optional
  needs_cf_credentials: true         # optional
  auto_services: p-mysql p-redis     # optional
```

Note: for applications that are normally pushed as multiple files (node.js for example)
you should zip up the project files plus all dependencies into a single zip file, then
edit tile.yml to point to the zipped file:

```bash
cd <your project dir>
zip -r resources/<your project name>.zip <list of file and dirs to include in the zip>
```

If your application is a service broker, use `app-broker` as the type instead of just
`app`. The application will then automatically be registered as a broker on install,
and deleted on uninstall.

`health_check` lets you configure the value of the cf cli `--health_check_type`
option. Expect this option to move into the manifest as soon as CF supports it there.
Currently, the only valid options are `none` and `port`.

`configurable_persistence: true` results in the user being able to select a backing
service for data persistence. If there is a specific broker you want to use, you can
use the `auto-services` feature described below. If you want to bind to an already
existing service instance, use the `services` proeprty of the `manifest` instead.

`needs_cf_credentials` causes the application to receive two additional environment
variables named `CF_ADMIN_USER` and `CF_ADMIN_PASSWORD` with the admin credentials
for the Elastic Runtime into which they are being deployed. This allows apps and
services to interact with the Cloud Controller.

`auto_services` is described in more detail below.

#### Service Brokers

Most modern service brokers are pushed into the Elastic Runtime as normal
CF applications. For these types of brokers, use the Pushed Application format
specified above, but set the type to `app-broker` or `docker-app-broker` instead
of just `app` or `docker-app`:

```
- name: my-broker
  type: app-broker
  manifest:
    command:
    domain:
    path:
    # ...
  needs_cf_credentials: true           # optional
  auto_services: p-mysql p-redis       # optional
  enable_global_access_to_plans: true  # optional
```

!!! note "Note:"
    Unless you specify the `enable_global_access_to_plans: true` option, your
    broker's services will not appear in the user's marketplaces.
    Operators will have to use the `cf enable-service-access` command to allow
    specific users, orgs, and spaces to access your services.

Your broker will be automatically registered with the Cloud Controller. The
Cloud Controller will invoke your broker's endpoints, and it will use basic
authentication to secure those API calls. The credentials it will use are
passed to your broker in two environment variables:

```
SECURITY_USER_NAME
SECURITY_USER_PASSWORD
```

Your broker is expected to accept those credentials. If it doesn't, automatic
broker registration will fail.

Some service brokers support operator-defined service plans, for instance when
the plans reflect customer license keys. To allow operators to add plans from
the tile configuration, add the following section at the top level of your tile.yml:

```
service_plan_forms:
- name: service_plans_1
  label: Service 1 Plans
  description: Specify the plans you want Service 1 to offer
  properties:
  - name: description
    type: string
    description: "Some Description"
    configurable: true
  - name: license_key1
    type: string
    configurable: true
    description: The license key for this plan
  - name: num_seats1
    type: integer
    configurable: true
    description: The number of available seats for this license
    default: 1
    constraints:
      min: 1
      max: 500
```

Name and GUID fields will be supplied by default for each plan, but all other fields
are optional and customizable. Multiple forms are supported. The operator-configured
plans will be passed to your service broker in JSON format in an environment variable
named after your form but in ALL CAPS (in this case `SERVICE_PLANS_1`).

For an external service broker, use:

```
- name: my-application
  type: external-broker
  uri: http://broker3.example.com
  username: user
  password: #secret
  internal_service_names: 'service1,service2'
```

#### Bosh Releases
You can include [BOSH releases](http://bosh.io/docs/release.html) in
your tile with the `bosh-release` package type. For example, here is a
package definition to include a Redis BOSH release:

```
- name: redis
  type: bosh-release
  path: resources/redis-12+dev.1.tgz
  jobs:
  - name: redis_leader_z1
    templates:
    - name: redis
      release: redis
    memory: 512
    ephemeral_disk: 4096
    persistent_disk: 4096
    cpu: 2
    static_ip: 1
    max_in_flight: 1
    properties:
      network: redis1
      redis:
        password: red!s
  - name: redis_z1
    templates:
    - name: redis
      release: redis
    instances: 2
    memory: 512
    ephemeral_disk: 4096
    persistent_disk: 4096
    cpu: 2
    static_ip: 1
    properties:
      network: redis1
      redis:
        master: (( .redis_leader_z1.first_ip ))
        password: red!s
  - name: redis_test_slave_z1
    templates:
    - name: redis
      release: redis
    instances: 1
    memory: 512
    ephemeral_disk: 4096
    persistent_disk: 4096
    cpu: 2
    static_ip: 1
    properties:
      network: redis1
      redis:
        master: (( .redis_leader_z1.first_ip ))
        password: red!s
  - name: acceptance-tests
    templates:
    - name: acceptance-tests
      release: redis
    lifecycle: errand
    post_deploy: true
    memory: 512
    ephemeral_disk: 4096
    persistent_disk: 0
    cpu: 2
    dynamic_ip: 1
    properties:
      redis:
        master: (( .redis_leader_z1.first_ip ))
        password: red!s
        slave: (( .redis_test_slave_z1.first_ip ))
```



#### Buildpacks

```
- name: my-buildpack
  type: buildpack
  path: resources/buildpack.zip
  buildpack_order: 99     # optional, 99 means end of the list
```

#### Docker Images

Applications packages as docker images can be deployed inside or outside the Elastic
Runtime. To push a docker image as a CF application, use the *Pushed Application*
format specified above, but use the `docker-app` or `docker-app-broker` type instead
of just `app` or `app-broker`. The docker image to be used is then specified using
the `image` property:

```
- name: app1
  type: docker-app
  image: test/dockerimage
  manifest:
    ...
```

If this app is also a service broker, use `docker-app-broker` instead of just
`docker-app`. This option is appropriate for docker-wrapped 12-factor apps that
delegate their persistence to bound services.

Docker applications that require persistent storage can not be deployed into
the Elastic Runtime. These can be deployed to separate BOSH-managed VMs instead
by using the `docker-bosh` type:

```
- name: docker-bosh1
  type: docker-bosh
  cpu: 5
  memory: 4096
  ephemeral_disk: 4096
  persistent_disk: 2048
  instances: 1
  manifest: |
    containers:
    - name: redis
      image: "redis"
      command: "--dir /var/lib/redis/ --appendonly yes"
      bind_ports:
      - "6379:6379"
      bind_volumes:
      - "/var/lib/redis"
      entrypoint: "redis-server"
      memory: "256m"
      env_vars:
      - "EXAMPLE_VAR=1"
    - name: mysql
      image: "google/mysql"
      bind_ports:
      - "3306:3306"
      bind_volumes:
      - "/mysql"
    - name: elasticsearch
      image: "bosh/elasticsearch"
      links:
      - mysql:db
      depends_on:
      - mysql
      bind_ports:
      - "9200:9200"
```

If a docker image cannot be downloaded by BOSH dynamically, its better to provide a ready made docker image and package it as part of the BOSH release. In that case, specify the image as a local file.

```
- name: docker-bosh2
  type: docker-bosh
  files:
  - path: resources/cfplatformeng-docker-tile-example.tgz
  cpu: 5
  memory: 4096
  ephemeral_disk: 4096
  persistent_disk: 2048
  instances: 1
  manifest: |
    containers:
    - name: test_docker_image
      image: "cfplatformeng/docker-tile-example"
      env_vars:
      - "EXAMPLE_VAR=1"
      # See below on custom forms/variables and binding it to the docker env variable
      - "custom_variable_name=((.properties.customer_name.value))"
```

### Custom Forms and Properties

You can pass custom properties to all applications deployed by your tile by adding
the to the properties section of `tile.yml`:

```
properties:
- name: author
  type: string
  label: Author
  value: Tile Ninja
```

If you want the properties to be configurable by the tile installer, place them on
a custom form instead:

```
forms:
- name: custom-form1
  label: Test Tile
  description: Custom Properties for Test Tile
  properties:
  - name: customer_name
    type: string
    label: Full Name
  - name: street_address
    type: string
    label: Street Address
    description: Address to use for junk mail
  - name: city
    type: string
    label: City
  - name: zip_code
    type: string
    label: ZIP+4
    default: '90310'
  - name: country
    type: dropdown_select
    label: Country
    options:
    - name: country_us
      label: US
      default: true
    - name: country_elsewhere
      label: Elsewhere
- name: account-info-1
  label: Account Info
  description: Example Account Information Form
  properties:
  - name: username
    type: string
    label: Username
  - name: password
    type: secret
    label: Password
```

Properties defined in either section will be passed to all pushed applications
as environment variables (the name of the environment variable will be the same
as the property name but in ALL_CAPS). They can also be referenced in other parts
of the configuration file by using `(( .properties.<property-name> ))` instead
of a hardcoded value.

All properties supported by Ops Manager may be used. The syntax is the same
as used by Ops Manager, except that for simplicity property blueprints for
form fields do not need to be declared separately. Instead, the declaration
is included in the form itself. For a complete list of supported property
types and syntax, see the
[Ops Manager Product Template Reference](https://docs.pivotal.io/partners/product-template-reference.html).

Properties of type `secret` will have their value hidden on the forms, and
obfuscated in the installation logs (all but the first two characters will be
replaced by `*****`). But their value will be passed to your applications in
plain text as all other value types.

### Automatic Provisioning of Services

Tile generator automates the provisioning of services. Any application (including
service brokers and docker-based applications) that are being pushed into the
Elastic Runtime can automatically be bound to services through the `auto_services`
feature:

```
- name: app1
  type: app
  auto_services:
  - name: p-mysql
    plan: 100mb-dev
  - name: p-redis
```

You can specify any number of service names, optionally specifying a specific
plan. During deployment, the generated tile will create an instance of each
service if one does not already exist, and then bind that instance to your
package.

Service instances provisioned this way survive updates, but will be deleted
when the tile is uninstalled.

!!! note "Note:"
    The name is the name of the provided *service, not the broker*.
    In many cases these are not the same, and a single broker may even offer
    multiple services. Use `cf service-access` to see the services and plans
    offered by installed service brokers.

If you do not specify a plan, the tile generator will use the first plan
listed for the service in the broker catalog. It is a good idea to always
specify a service plan. If you *change* the plan between versions of your
tile, the tile generator will attempt to update the plan while preserving
the service (thus not causing data loss during upgrade). If the service
does not support plan changes, this will cause the upgrade to fail.

`configurable_persistence` is really just a special case of `auto_services`,
letting the user choose between some standard brokers.

### Declaring Product Dependencies

When your product has dependencies on others, you can have Ops Manager
enforce that dependency by declaring it in your `tile.yml` file as follows:

```
requires_product_versions:
- name: p-mysql
  version: '~> 1.7'
```

If the required product is not present in the PCF installation, Ops Manager
will display a message saying
`<your-tile> requires 'p-mysql' version '~> 1.7' as a dependency`, and will
refuse to install your tile until that dependency is satisfied.

When using automatic provisioning of services as described above, it is
often appropriate to add those products as a dependency. Tile generator can
not do this automatically as it can't always determine which product provides
the requested service.

### Orgs and Spaces

By default, the tile generator will create a single new org and space for any
packages that install into the Elastic Runtime, using the name of the tile and
appending `-org` and `-space`, respectively. The default memory quota for a
newly created or will be 1024 (1G). You can change any of these defaults by
specifying the following properties in `tile.yml`:

```
org: test-org
org_quota: 4096
space: test-space
```

### Security

If your cf packages need outbound access (including access to other packages
within the same tile), you will need to apply an appropriate security group.
The following option will remove all constraints on outbound traffic:

```
apply_open_security_group: true
```

### Stemcells

The tile generator will default to a recent stemcell supported by Ops Manager.
In most cases the default will be fine, as the stemcell is only used to execute
CF command lines and/or the docker daemon. But if you have specific stemcell
requirements, you can override the defaults in your `tile.yml` file by including
a `stemcell-criteria` section and replacing the appopriate values:

```yaml
stemcell_criteria:
  os: 'ubunty-trusty'
  version: '3146.5'     #NOTE: You must quote the version to force the type to be string
```

### Custom Errands

Tile generator supplies standard errands to deploy and delete CF type packages. You can
replace or augment those errands by specifying errand shell commands in your tile.yml
file. For example:

```
packages:
- name: meta-buildpack
  type: buildpack
  buildpack_order: 0 # Go to head of list
  path: meta_buildpack.zip
  deploy: |
    cp meta_buildpack.zip meta_buildpack-v{{context.version}}.zip
    existing=`cf buildpacks | grep '^meta_buildpack'`
    if [ -z "$existing" ]; then
      cf create-buildpack meta_buildpack meta_buildpack-v{{context.version}}.zip 0
    else
      semver=`echo "$existing" | sed 's/.* meta_buildpack-v\(.*\)\.zip/\1/'`
      if is_newer "{{context.version}}" "$semver"; then
        cf update-buildpack meta_buildpack -p meta_buildpack-v{{context.version}}.zip
      else
        echo "Newer version ($semver) of meta_buildpack is already present"
      fi
      cf update-buildpack meta_buildpack -i 0
    fi
  delete: |
    # Intentional no-op, as others may have a dependency on this
```

`deploy` and `delete` will completely replace the standard errand commands for the
package in which you include them. If you want to keep the standard commands, but
add additional commands to execute before or after the standard errand, use
`pre_deploy`, `post_deploy`, `pre_delete`, and/or `post_delete` instead.

## Versioning

The tile generator uses [semver versioning](http://semver.org/). By default, `tile build` will
generate the next patch release. Major and minor releases can be generated
by explicitly specifying `tile build major` or `tile build minor`. Or to
override the version number completely, specify a valid semver version on
the build command, e.g. `tile build 3.4.5`.

No-op content migration rules are generated for every prior release to the
current release, so that Ops Manager will allow tile upgrades from any
version to any newer version. This depends on the existence of the file
`tile-history.yml`. In a pinch, if you need to be able to upgrade from a
random old version to a new one, you can edit that file, or do:

```
tile build <old-version>
tile build <new-version>
```

The new tile will then support upgrades from `old-version`.

<a name="upgrades"></a> 
## Upgrades

By default, tile generator produces all code necessary to do a blue/green,
zero-downtime deployment of all tile components when installing a newer version
over an older one. For most tile versions this will be all that is needed.

Ops Manager has support for performing upgrade actions, like database migrations,
during a tile upgrade, but this capability is not yet exposed through tile
generator.

## Example

```bash
$ tile build
name: tibco-bwce
icon: icon.png
label: TIBCO BusinessWorks Container Edition
description: BusinessWorks edition that supports deploying to Cloud Foundry
version: 0.0.2

bosh init release
bosh generate package cf_cli
bosh generate package bwce_buildpack
bosh generate job install_bwce_buildpack
bosh generate job remove_bwce_buildpack
bosh create release --final --with-tarball --version 0.0.2

tile generate release
tile generate metadata
tile generate errand install_bwce_buildpack
tile generate errand remove_bwce_buildpack
tile generate content-migrations

created tile tibco-bwce-0.0.2.pivotal
```

This tile includes a single large buildpack, and takes less than 15 seconds
to build including the CF CLI download and the BOSH release generation.

## Supported Commands

```bash
tile init [<tile-name>]
tile build [patch|minor|major|<version>]
```

## Credits

- [sparameswaran](https://github.com/sparameswaran) supplied most of the actual template content, originally built as part of [cf-platform-eng/bosh-generic-sb-release](https://github.com/cf-platform-eng/bosh-generic-sb-release.git)
- [frodenas](https://github.com/frodenas) contributed most of the docker content through [cloudfoundry-community/docker-boshrelease](https://github.com/cloudfoundry-community/docker-boshrelease.git)
- [joshuamckenty](https://github.com/joshuamckenty) suggested the jinja template approach he employed in [opencontrol](https://github.com/opencontrol)
