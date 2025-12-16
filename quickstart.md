# DataPower Container Installation Guide

## Prerequisites

### Verify Docker or Podman Installation

Ensure that either Docker or Podman is installed on your system. You can verify the installation by running one of the following commands:

**For Docker:**
```bash
docker --version
```

**For Podman:**
```bash
podman --version
```

If neither command returns a version number, you'll need to install Docker or Podman before proceeding.
### Retrieve Available Image Tags

To retrieve the available image tags:

```bash
skopeo list-tags --creds "cp:${ENTITLEMENT_KEY}" docker://cp.icr.io/cp/datapower/datapower-prod
```

Find the image location for different versions here: [IBM DataPower Operator Documentation](https://www.ibm.com/docs/en/datapower-operator/1.17.0?topic=features-entitled-registry)

### Upload Image to Mirror Registry (Air Gap Installation)

For air gap installations, you'll need to copy the DataPower image to your mirror registry. Using `skopeo` is the most efficient method as it copies images directly between registries without requiring local storage.

**Copy image to mirror registry using skopeo:**

```bash
skopeo copy \
  --src-creds "cp:${ENTITLEMENT_KEY}" \
  --dest-creds "${MIRROR_USERNAME}:${MIRROR_PASSWORD}" \
  docker://cp.icr.io/cp/datapower/datapower-prod:10.5.4.0 \
  docker://${MIRROR_REGISTRY}/datapower-prod:10.5.4.0
```

**Note:** 
- Replace `${MIRROR_REGISTRY}`, `${MIRROR_USERNAME}`, and `${MIRROR_PASSWORD}` with your actual mirror registry details, or export them as environment variables.
- Replace `10.5.4.0` with the version tag you want to copy.
- Ensure `skopeo` is installed on your system. You can verify by running `skopeo --version`.

After copying, you can use the image from your mirror registry by replacing `cp.icr.io/cp/datapower/datapower-prod:10.5.4.0` with `${MIRROR_REGISTRY}/datapower-prod:10.5.4.0` in the run commands below.

### Differences Between DataPower Products

For information about differences between DataPower products, see: [DataPower Product Differences](https://www.ibm.com/docs/en/datapower-gateway/10.6.0?topic=overview-differences-among-datapower-products)

> **Note:** Only the Application Optimized Module, B2B Module, and Integration Module are available in the Docker version.

## Getting Started

### Create Persistent Volumes

Create folders for persistent storage (config, local, certs). For more information, see: [Virtual DataPower Gateway Docker Documentation](https://www.ibm.com/docs/en/datapower-gateway/10.6.0?topic=virtual-datapower-gateway-docker)

Make sure that the container have read and write access

```bash
mkdir -p config local certs
chmod -R 777 config local certs 
```

## Using Docker

### Step 1: Login to IBM Container Registry

First, login to the IBM Container Registry:

```bash
docker login cp.icr.io -u cp -p ${ENTITLEMENT_KEY}
```

**Note:** Replace `${ENTITLEMENT_KEY}` with your actual IBM entitlement key, or export it as an environment variable.

### Step 2: Run the Container

Then run the container:

```bash
docker run -it --name idg \
  -v $(pwd)/config:/opt/ibm/datapower/drouter/config \
  -v $(pwd)/local:/opt/ibm/datapower/drouter/local \
  -v $(pwd)/certs:/opt/ibm/datapower/root/secure/usrcerts \
  -e DATAPOWER_ACCEPT_LICENSE="true" \
  -e DATAPOWER_INTERACTIVE="true" \
  -p 9090:9090 \
  cp.icr.io/cp/datapower/datapower-prod:10.5.4.0
```

## Using Podman

### Step 1: Login to IBM Container Registry

First, login to the IBM Container Registry:

```bash
podman login cp.icr.io -u cp -p ${ENTITLEMENT_KEY}
```

**Note:** Replace `${ENTITLEMENT_KEY}` with your actual IBM entitlement key, or export it as an environment variable.

### Step 2: Run the Container

Then run the container:

```bash
podman run -it --name idg \
  -v $(pwd)/config:/opt/ibm/datapower/drouter/config \
  -v $(pwd)/local:/opt/ibm/datapower/drouter/local \
  -v $(pwd)/certs:/opt/ibm/datapower/root/secure/usrcerts \
  -e DATAPOWER_ACCEPT_LICENSE="true" \
  -e DATAPOWER_INTERACTIVE="true" \
  -p 9090:9090 \
  cp.icr.io/cp/datapower/datapower-prod:10.5.4.0
```

## Configuration

### Initial Setup

1. Login to the container as `admin` with password `admin`
2. Run the following configuration commands:

```
configure terminal; web-mgmt; admin-state enabled; local-address 0.0.0.0; exit; write mem;
```

### Access the Web UI

Access the DataPower Web UI using this URL: `https://<serverhostname>:9090`
