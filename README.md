
# FastAPI GitOps Deployment using FluxCD

This repository contains the **GitOps configuration for deploying the FastAPI application using FluxCD and Helm** to a Kubernetes cluster.

The setup uses the **Flux Helm Controller** to automatically deploy the FastAPI Helm chart stored in a separate Helm repository.

---

# Architecture

```text
FastAPI Helm Repository (fastapi-helm)
        │
        │  contains
        │  - fastapi-chart-0.1.0.tgz
        │  - index.yaml
        ▼
Flux HelmRepository
        │
        ▼
Flux HelmRelease
        │
        ▼
Kubernetes Cluster (KIND)
        │
        ▼
FastAPI Application Running
```

---

# Repository Structure

```text
fastapi-gitops
│
├── helmrepo.yaml
├── helmrelease.yaml
└── kustomization.yaml
```

---

# Prerequisites

Install the following tools:

* Kubernetes cluster (KIND recommended)
* kubectl
* Helm
* Flux CLI

---

# Step 1: Create Local Kubernetes Cluster

Create a cluster using KIND.

```bash
kind create cluster --name fastapi-cluster
```

Verify cluster:

```bash
kubectl get nodes
```

---

# Step 2: Install FluxCD

Install Flux controllers in the cluster.

```bash
flux install
```

Verify controllers:

```bash
kubectl get pods -n flux-system
```

Expected controllers:

```
source-controller
helm-controller
kustomize-controller
notification-controller
```

---

# Step 3: Configure Helm Repository

Flux must know where the Helm charts are stored.

Create **helmrepo.yaml**

```yaml
apiVersion: source.toolkit.fluxcd.io/v1
kind: HelmRepository
metadata:
  name: fastapi-repo
  namespace: flux-system

spec:
  interval: 1m
  url: https://raw.githubusercontent.com/<your-username>/fastapi-helm/main
```

This points Flux to the Helm repository that contains:

```
fastapi-chart-0.1.0.tgz
index.yaml
```

---

# Step 4: Create HelmRelease

The HelmRelease resource tells Flux which Helm chart to install.

Create **helmrelease.yaml**

```yaml
apiVersion: helm.toolkit.fluxcd.io/v2
kind: HelmRelease

metadata:
  name: fastapi
  namespace: default

spec:
  interval: 5m

  chart:
    spec:
      chart: fastapi-chart
      version: "0.1.0"

      sourceRef:
        kind: HelmRepository
        name: fastapi-repo
        namespace: flux-system

  values:
    service:
      type: NodePort
      port: 80
```

Flux will install the chart using Helm.

---

# Step 5: Create Kustomization

Create **kustomization.yaml**

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - helmrepo.yaml
  - helmrelease.yaml
```

---

# Step 6: Apply GitOps Manifests

Apply the configuration to the cluster.

```bash
kubectl apply -f .
```

Flux will detect the Helm repository and deploy the Helm chart.

---

# Step 7: Verify Deployment

Check Helm repositories:

```bash
flux get sources helm
```

Check Helm releases:

```bash
flux get helmreleases
```

Check pods:

```bash
kubectl get pods
```

Check services:

```bash
kubectl get svc
```

---

# Application Access

If the service type is **NodePort**, access the application using:

```
http://localhost:<nodeport>
```

Example:

```
http://localhost:30007
```

---

# Update Process

To deploy a new version of the application:

1. Update Helm chart version
2. Package the chart

```bash
helm package fastapi-chart
```

3. Update the Helm repository index

```bash
helm repo index .
```

4. Push changes to the Helm repository

5. Update the chart version in `helmrelease.yaml`

Flux will automatically upgrade the application.

---

# Summary

This GitOps repository enables automated deployment of the FastAPI application by:

* Reading the Helm repository
* Installing the Helm chart using Flux Helm Controller
* Continuously reconciling the cluster state

This ensures **consistent, automated, and version-controlled Kubernetes deployments**.


**Helm Controller** is the most powerful Flux CD feature for managing Helm releases declaratively in Git.



# Bootstrap

flux bootstrap github `
  --owner=soumen321 `
  --repository=fastapi-gitops `
  --branch=main `
  --path=clusters/dev `
  --personal

# Monitor
flux get sources all
flux get hr
kubectl get ns
kubectl get all -n flux-system