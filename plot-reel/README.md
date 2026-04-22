# Helm Deployment for Plot Reel

This document provides a comprehensive guide on how to deploy the Plot Reel application using Helm. It includes step-by-step instructions for building the container image, preparing secrets, and deploying to an OpenShift cluster.


## Repository

[Plot‑Reel Deployment Repository](https://gitlab.science.gc.ca/hpc-aafc/aafc-k8-on-hpc/-/tree/protractor/deployments/plot-reel-dev)

## Goals
- Provide clear commands to build and push the image.
- Explain how to create secrets for the deployment.
- Show step-by-step deployment commands using Helm.

## Prerequisites
- Access to an OpenShift cluster and a project/namespace.
- `podman` (or `docker`) to build images.
- `oc` (OpenShift CLI) or `kubectl` for applying manifests.
- Access to an image registry you can push to (Quay, Docker Hub, private registry).
- Helm installed where your OpenShift CLI is installed.

## Step 1: Prepare Secrets and Configuration
Prepare the secrets that will be passed to Helm at deployment time:

1. Generate the encryption key (if not already generated). Navigate to the `plot-reel-dev` directory, then:
   ```bash
   cd server
   python generate_key.py  # writes server/key
   ```

2. Within the same `server` directory, create (or update) `server/users.csv` with headers `username,password` and your initial user row. Then, encrypt the file:
   ```bash
   python encryption.py encrypt
   ```

3. Create base64-encoded versions (single-line) of the secrets:
   ```bash
   base64 -w0 key > key.b64
   base64 -w0 users.csv > users.csv.b64
   ```
   
   These b64 files will be referenced during Helm installation using `--set-file`.

## Step 2: Build and Push the Container Image

1. Change directory into the root folder of the Plot Reel project (`plot-reel-dev`).

2. Build the container image:
   ```bash
   podman build --no-cache -t <IMAGE_NAME>:<IMAGE_TAG> .
   ```
   Example:
   ```bash
   podman build --no-cache -t plot-reel:latest .
   ```

3. Log in to your registry (if not already logged in):
   ```bash
   podman login <REGISTRY>
   ```

4. Tag and push the image to your registry:
   ```bash
   podman tag plot-reel:<IMAGE_TAG> <REGISTRY>/<IMAGE_NAME>:<IMAGE_TAG>
   podman push <REGISTRY>/<IMAGE_NAME>:<IMAGE_TAG>
   ```
   
   **Note:** If you encounter certificate errors, you can use the `--tls-verify=false` flag:
   ```bash
   podman push <REGISTRY>/<IMAGE_NAME>:<IMAGE_TAG> --tls-verify=false
   ```

## Step 3: Deploy with Helm
1. Log in to your OpenShift cluster (if not already logged in):
   ```bash
   oc login <CLUSTER_URL>
   ```
   
2. Select or create your namespace:
   ```bash
   oc project <NAMESPACE>
   ```

3. Install the Helm chart using `--set-file` to provide sensitive and configuration values and `--set` for strings:
   ```bash
   helm install plot-reel-helm ./plot-reel-helm \
     -n <NAMESPACE> \
     --set image.repository=<IMAGE_REPOSITORY> \
     --set image.tag=<IMAGE_TAG> \
     --set-file secrets.keyBase64=server/key.b64 \
     --set-file secrets.userscsvBase64=server/users.csv.b64
   ```
   
   **Example:**
   ```bash
   helm install plot-reel-helm ./plot-reel-helm \
     -n default \
     --set image.repository=<IMAGE_REPOSITORY> \
     --set image.tag=<IMAGE_TAG> \
     --set-file secrets.keyBase64=server/key.b64 \
     --set-file secrets.userscsvBase64=server/users.csv.b64
   ```

4. If you need to customize other values (e.g., `config`, `persistence`, `resources`), you can use additional `--set` flags:


5. Verify the deployment:
   ```bash
   helm list -n <NAMESPACE>
   oc get pods -n <NAMESPACE>
   ```

## Step 4: Updating the Deployment
To update the deployment with new changes:
1. Make your changes to the code or configuration.
2. Rebuild and push the updated image.
3. Upgrade the Helm release:
   ```bash
   helm upgrade plot-reel-helm ./plot-reel-helm \
     -n <NAMESPACE> \
     --set image.repository=<IMAGE_REPOSITORY> \
     --set image.tag=<IMAGE_TAG> \
     --set-file secrets.keyBase64=server/key.b64 \
     --set-file secrets.userscsvBase64=server/users.csv.b64
   ```

## Step 5: Uninstalling the Deployment
To uninstall the Helm release:
```bash
helm uninstall plot-reel-helm -n <NAMESPACE>
```

## Conclusion
This guide provides a straightforward approach to deploying the Plot Reel application using Helm. Ensure you follow each step carefully to achieve a successful deployment.