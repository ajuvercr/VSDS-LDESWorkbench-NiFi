# LDES Workbench NiFi

The workbench is a customized [Apache NiFi](https://nifi.apache.org) docker image containing processors built by VSDS to allow processing of data that is intended to be ingested by an LDES server.

- [Docker image](#docker-image)
    - [Custom NiFi image](#custom-nifi-image)
    - [VSDS LDES Workbench NiFi image](#vsds-ldes-workbench-nifi-image)
- [VSDS LDES NiFi processors](#vsds-ldes-nifi-processors)
    - [LDES Client wrappers NiFi](#ldes-client-wrappers-nifi)
    - [NGSIv2 to NGSI-LD processor](#ngsiv2-to-ngsi-ld-processor]
    - [NGSI-LD to LDES processor](#ngsi-ld-to-ldes-processor]

## Docker image

### Custom NiFi image

The workbench docker image contains a customized [NiFi](https://nifi.apache.org) instance, built with JDK 17.

#### github

A [github workflow](.github/workflows/0.build_push-docker-nifi.yml) checks if the custom NiFi image is present on github and builds it otherwise.

The built customized NiFi docker image can be found [here](https://github.com/Informatievlaanderen/VSDS-LDESWorkbench-NiFi/pkgs/container/nifi)

The configuration used in the github workflow can be found below. Github workflows don't allow referencing previously defined environment settings in another setting, hence the repetitions.

```yaml
REGISTRY: ghcr.io
USERNAME: informatievlaanderen
BASE_IMAGE_TAG: 17-slim-buster
NIFI_VERSION: 1.17.0
DISTRO_PATH: 1.17.0
TARGET_IMAGE_TAG: 1.17.0-jdk17
NIFI_IMAGE: nifi
NIFI_DOCKER_IMAGE_NAME: ghcr.io/informatievlaanderen/nifi
NIFI_DOCKER_IMAGE_VERSION: 1.17.0-jdk17
```

The generated docker image is reused as base image of the [VSDS LDES Workbench NiFi image](#vsds-ldes-workbench-nifi-image).

#### building locally

To build the custom NiFi image locally, follow these steps:

1. Fetch the official Apache NiFi image and extract it

```shell
curl -L https://github.com/apache/nifi/archive/refs/tags/rel/nifi-1.17.0.tar.gz | tar -zxv --strip-components=2 nifi-rel-nifi-1.17.0/nifi-docker/dockerhub/
```

2. Allow the dockerfile to find local context

```shell
sed -i 's/xmlstarlet procps$/xmlstarlet procps curl unzip/' dockerhub/Dockerfile
```

3. Build the docker image

```shell
docker build --build-arg IMAGE_TAG="${BASE_IMAGE_TAG}" --build-arg NIFI_VERSION="${NIFI_VERSION}" --build-arg DISTRO_PATH="${DISTRO_PATH}" -t "${NIFI_DOCKER_IMAGE_NAME}:${NIFI_DOCKER_IMAGE_VERSION}" -f dockerhub/Dockerfile dockerhub/
```

Environment variables can be taken from the github workflow and adjusted as desired.

### VSDS LDES Workbench NiFi image

A docker is provided that contains a NiFi instance (based on the [Custom NiFi image](#custom-nifi-image)) and the [processor NAR files](#vsds-ldes-nifi-processors). The docker doesn't contain any workflows, these can be uploaded by logging into the NiFi instance (make sure to set the proper environment variables for username and password when building) or by using the [NiFi toolkit](https://hub.docker.com/r/apache/nifi-toolkit).

#### github

A [github workflow](.github/workflows/1.b.pr_build-docker-workbench-nifi.yml) builds the docker image.

The built customized NiFi docker image can be found [here](https://github.com/Informatievlaanderen/VSDS-LDESWorkbench-NiFi/pkgs/container/ldes-workbench-nifi)

The configuration used in the github workflow can be found below.

```yaml
REGISTRY: ghcr.io
NIFI_DOCKER_IMAGE_VERSION: 1.17.0-jdk17
LDES_NIFI_DOCKER_IMAGE_NAME: ghcr.io/informatievlaanderen/ldes-workbench-nifi
SINGLE_USER_CREDENTIALS_USERNAME: ${{ secrets.SINGLE_USER_CREDENTIALS_USERNAME }}
SINGLE_USER_CREDENTIALS_PASSWORD: ${{ secrets.SINGLE_USER_CREDENTIALS_PASSWORD }}
```

Be sure to set the `SINGLE_USER_CREDENTIALS_USERNAME` and `SINGLE_USER_CREDENTIALS_PASSWORD` to allow login to the NiFi instance. When ommitted, the build will succeed, but values will be generated for you (logged to the console), making automation impossible.

#### building locally

```shell
git clone git@github.com:Informatievlaanderen/VSDS-LDESWorkbench-NiFi.git
cd VSDS-LDESWorkbench-NiFi/ldes-workbench-nifi
docker build --build-arg NIFI_DOCKER_IMAGE_VERSION="1.17.0-jdk17" --build-arg SINGLE_USER_CREDENTIALS_USERNAME="<a username>" --build-arg SINGLE_USER_CREDENTIALS_PASSWORD="<a password>" -f Dockerfile
```


## VSDS LDES NiFi processors

### LDES Client Wrappers NiFi

A wrapper around the [LDES Client SDK](https://github.com/Informatievlaanderen/VSDS-LDESClient4J) that fetches LDES fragments, processes them and sends the members to an [LDES server](https://github.com/Informatievlaanderen/VSDS-LDESServer4J).

Documentation is available [here](./ldes-client-wrappers-nifi/README.md).

### NGSIv2 to NGSI-LD processor

A processor that translates [NGSI v2](https://fiware.github.io/specifications/ngsiv2/stable/) data to [NGSI-LD](https://www.etsi.org/deliver/etsi_gr/CIM/001_099/008/01.01.01_60/gr_CIM008v010101p.pdf) data using [the protocol deliver by fiware](https://fiware-datamodels.readthedocs.io/en/stable/ngsi-ld_howto/index.html#steps-to-migrate-to-json-ld)

Documentation is available [here](./ngsiv2-to-ngsi-ld-processor/README.md).

### NGSI-LD to LDES processor

A processor that adds data elements required to turn the NGSI-LD stream into an LDES stream.

Documentation is available [here](./ngsi-ld-to-ldes-processor/README.md).
