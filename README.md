# k8s-fleet

This repository contains Kubernetes configuration files for k8s clusters.

It uses the following top directory structure:

- `clusters` dir contains the Flux configuration per cluster
- `sources` dir contains source definitions common to all clusters
- `apps` dir contains kustomizations or Helm releases with a custom configuration per cluster

Each cluster in the `clusters` dir has a `flux-system` directory with automatically-generated configuration (`clusters/<cluster>/flux-system`).

The `apps` dir is structured as a kustomization directory with a base (`apps/base`) and various overlays, one per cluster (`app/<cluster>`).

The `apps/base` dir contains common configurations for all clusters. Overlays dirs contain patches per cluster.

