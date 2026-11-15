# Task Manager Kubernetes manifests

This repository contains the existing Task Manager Helm chart and the Argo CD
application definition. Argo CD is installed by Ansible in the separate
infrastructure repository and syncs `helm/task-manager-api` from this repo.

## Image

Set `image.repository` in
[`helm/task-manager-api/values.yaml`](helm/task-manager-api/values.yaml) to
`docker.io/<your-dockerhub-username>/task-manager-api`. The application
repository's GitHub Actions workflow publishes immutable `sha-...` image tags
to Docker Hub. Update `image.tag` here to deploy a particular image; Argo CD
then applies the change without GitHub having Kubernetes credentials.
If the Docker Hub repository is private, configure an `imagePullSecret` on the
cluster; otherwise make the image repository public.

## Validate and render locally

```bash
helm lint helm/task-manager-api
helm template task-manager-api helm/task-manager-api --namespace task-manager
```

GitHub Actions runs the same chart checks on pushes and pull requests. It does
not access the cluster.

## High availability and persistent data

The API is configured with three replicas, resource requests, health probes,
and a MetalLB `LoadBalancer` Service. Node-level failover is expected from
Proxmox HA.

The chart's existing MongoDB StatefulSet remains a single replica using the
`local-path` storage provisioner installed by Ansible. That volume is tied to
one node and is not an HA database. Use an HA MongoDB replica set or an
external database, plus a backup/restore plan, before relying on this setup
for production data.

## Certificates

Ansible installs cert-manager. Configure an issuer and either a DNS solver or
an ingress controller before requesting certificates; no issuer or public
domain is configured by default.
