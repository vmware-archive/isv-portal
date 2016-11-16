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
pipeline that you can clone for your own tile. 

We have provided a docker image to help you automate your tile creation tasks via concourse. 
To make use of this image, you will want to do he following:

1. Declare a resource source repository (for example, git) for your tile in your pipeline.yml file:

```yml
- name: tile-source
  type: git
  source:
    branch: master
    uri: https://github.com/your-tile-project-repo
```

2. Declare a resource to store the artifacts of your build process (for example, an aws s3
bucket) in your pipeline.yml file:

```yml
- name: tile-build
  type: s3
  source:
    access_key_id: your-aws-key-id-don't-check-this-in!
    bucket: your-aws-bucket-name
    regexp: .*-(?P<version>.*)\.jar
    secret_access_key: your-aws-access-key-don't-check-this-in!
```

3. Declare a resource to store your tile in your pipeline.yml file:

```yml
- name: tile
  type: s3
  source:
    access_key_id: your-aws-key-id-don't-check-this-in!
    bucket: your-aws-bucket-name
    regexp: .*-(?P<version>.*)\.pivotal
    secret_access_key: your-aws-access-key-don't-check-this-in!
```

4. Declare a resource to store a tile-history.yml file. This file is needed by the tile-generator process.
It makes sense to store this in the same place as your build artifacts (but you can store it in another bucket if you wish):

```yml
- name: tile-history
  type: s3
  source:
    access_key_id: your-aws-key-id-don't-check-this-in!
    bucket: your-aws-bucket-name
    regexp: tile-history-(?P<version>.*)\.yml
    secret_access_key: your-aws-access-key-don't-check-this-in!
```

5. (Optional) consider managing the version numbers of your project via the use of [semver](http://semver.org/). Concourse has
support for this. If you decide to do this, add something like the following to your pipeline.yml file:


```yml
- name: version
  type: semver
  source:
    bucket: your-aws-bucket-name
    key: current-version
    access_key_id: your-aws-key-id-don't-check-this-in!
    secret_access_key: your-aws-access-key-don't-check-this-in!
    initial_version: 1.0.0
```

6. add a job such as the following, to your pipeline:


```yml
- name: build-tile
  serial_groups: [version]
  plan:
  - aggregate:
    - get: tile-source
    - get: tile-build
    - get: version
    - get: tile-history
  - task: build-tile
    file: tile-source/ci/build-tile/task.yml
  - put: tile-history
    params: {file: tile-history-new/*.yml}
  - put: tile
    params: {file: broker-tile/*.pivotal}
```

7. define the tile build task via a task.yml file (per the above job configuration, this file would
be added to the ci/build-tile directory in your source repository):

```yml
platform: linux

image: docker:///cfplatformeng/tile-generator

inputs:
- name: tile-source
- name: tile-build
- name: version
- name: tile-history

outputs:
- name: tile
- name: tile-history-new

run:
  path: tile-repo/ci/build-tile/task.sh
```

8. create a task.sh script to build the tile (per the above job configuration, this file would
be added to the ci/build-tile directory in your source repository):

```sh
#!/bin/sh -ex

cd tile-repo

cp ../../tile-build/* the-place-in-tile.yml-where-the-build-goes

ver=`more ../../version/number`
tile build ${ver}

file=`ls product/*.pivotal`
filename=$(basename "${file}")
filename="${filename%-*}"

cp ${file} ../../tile/${filename}-${ver}.pivotal
cp tile-history.yml ../../tile-history-new/tile-history-${ver}.yml
```

9. string this job together with the other jobs in your pipeline (probably after a successful tile-build).
The job will then build your tile and place it in your s3 bucket along with an updated tile-history file.


<a name="pool"></a> 
## Setting up PCF for your CI Pipeline

Pivotal partners who have us host their pipeline have access to a pool of PCF
instances that are managed by us and are regularly updated with the latest
(pre-)release versions of PCF. If you set up your own concourse server, you
will have to target your pipeline at a [PCF instance you have setup](setup-pcf.md).

Concourse has a resource type to manage a pool of resources that are shared
between pipelines, which is what we use to serialize PCF access between the
partner pipelines that run on our concourse server.
