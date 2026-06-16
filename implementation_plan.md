# Implementation Plan - Kubernetes Manifest Fixes

Now that local Docker Compose testing is working perfectly, we will prepare the Kubernetes manifests for deployment on AWS EKS by fixing the configuration errors.

## User Review Required
None. These changes only target standardizing deployment settings and fixing spec errors.

## Proposed Changes

We will modify the backend and database manifests and introduce a namespace definition manifest.

### [Kubernetes Manifests]

#### [MODIFY] [deployment.yaml]
Add `USE_DB_AUTH: "true"` environment variable so that backend NodeJS pods can successfully authenticate with MongoDB using credentials.

#### [MODIFY] [pv.yaml]
Remove the `namespace: three-tier` attribute. PersistentVolumes are cluster-scoped, not namespaced.

#### [NEW] [namespace.yaml]
Add a namespace definition manifest so the `three-tier` namespace is automatically created.

---

## Verification Plan

### Automated / Manual Verification
1. We will verify the YAML files are syntactically valid.
2. The user will be able to apply the files to EKS using:
   ```bash
   kubectl apply -f Kubernetes-Manifests-file/namespace.yaml
   kubectl apply -f Kubernetes-Manifests-file/Database/
   kubectl apply -f Kubernetes-Manifests-file/Backend/
   kubectl apply -f Kubernetes-Manifests-file/Frontend/
   kubectl apply -f Kubernetes-Manifests-file/ingress.yaml
   ```
