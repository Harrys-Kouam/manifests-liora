# Task Manager Kubernetes manifests

This repository contains the existing Task Manager Helm chart and the Argo CD
application definition. Argo CD is installed by Ansible in the separate
infrastructure repository and syncs `helm/task-manager-api` from this repo.

## Image

`image.repository` in
[`helm/task-manager-api/values.yaml`](helm/task-manager-api/values.yaml) points to
`docker.io/harryskouam/task-manager-api`. The application repository's
pipeline builds the image and then commits the new `sha-...` value of
`image.tag` here; Argo CD applies it without GitHub having Kubernetes credentials.
The Docker Hub repository must stay public (no `imagePullSecret` is configured).

## Validate and render locally

```bash
helm lint helm/task-manager-api
helm template task-manager-api helm/task-manager-api --namespace task-manager
```

GitHub Actions runs the same chart checks on pushes and pull requests. It does
not access the cluster.

## Availability and persistent data

The API runs three replicas with health probes behind a MetalLB `LoadBalancer`
Service. The cluster has one master and two workers on a single Proxmox
server, so there is no host-level high availability.

MongoDB is a single StatefulSet replica on the `local-path` storage
provisioner, so its volume is tied to one node and it is not a replicated
database. Velero (installed by Ansible) backs up the `task-manager` volumes to
MinIO, so the data can be restored or moved to another cluster.

## Certificates

Ansible installs cert-manager. Configure an issuer and either a DNS solver or
an ingress controller before requesting certificates; no issuer or public
domain is configured by default.
