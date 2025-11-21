# OpenShift deployment configs

This repository contains OpenShift configuration files for building and deploying the application stack to an OpenShift cluster.

## Contents

- **BuildConfigs + ImageStreams**
    - Build and tag container images for:
        - Frontend
        - Node.js server
        - Python (FastAPI) server

- **Application deployments**
    - Frontend deployment, service and route
    - Node.js server deployment, service and route
    - Python (FastAPI) server deployment, service and route

- **Database**
    - PostgreSQL deployment with persistent storage
    - PostgreSQL service
    - Database secret (credentials + connection URL)

- **Application secrets**
    - Secret for application-level configuration (API keys, storage config etc.)

## Prerequisites

- Access to an OpenShift cluster (via `oc` CLI or UI)
- A project/namespace selected in OpenShift:
  ```bash
  oc project <your-project-name>
  ```

## How to apply the configuration

1. **Create secrets**

   Edit the secret manifests to fill in the required values and apply them:

   ```bash
   oc apply -f app-secrets.yaml
   oc apply -f db.yaml
   ```

   > Ensure all placeholders (API keys, database URL, usernames, passwords, etc.) are set before applying.

2. **Set up builds and image streams**

   Apply the build and image configuration:

   ```bash
   oc apply -f frontend-buildconfig.yaml
   oc apply -f server-node-buildconfig.yaml
   oc apply -f server-py-buildconfig.yaml
   ```

   This will:
    - Create `BuildConfig`s that build images from the application Git repository.
    - Create corresponding `ImageStream`s that hold the built images.

3. **Start initial builds (if not triggered automatically)**

   ```bash
   oc start-build frontend --wait
   oc start-build server --wait
   oc start-build server-fastapi --wait
   ```

4. **Deploy the components**

   Apply the deployment, service and route manifests:

   ```bash
   oc apply -f postgres.yaml        # or db.yaml, depending on your filename
   oc apply -f frontend.yaml
   oc apply -f server-node.yaml     # or server.yaml, depending on your filename
   oc apply -f server-py.yaml
   ```

5. **Check status**

   ```bash
   oc get pods
   oc get svc
   oc get routes
   ```

   Once pods are running and routes are created, the application should be accessible via the route hostnames.

## Updating the application
- Configure the webhooks in openshift and add them to GitHub
- Push changes to the configured Git branch.
- The corresponding `BuildConfig` webhooks (or image change triggers) will:
    - Rebuild the image.
    - Update the deployment via the `image.openshift.io/triggers` annotations.

If needed, you can manually trigger a rollout:
