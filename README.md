# k8s-fleet

This repository is a template containing intial Kubernetes configuration files for a multi-cluster k8s fleet.

It uses the following top directory structure:

- `clusters` dir contains the Flux configuration per cluster under `clusters/<cluster>`
- `sources` dir contains source definitions common to all clusters
- `configs` dir contains kustomizations or Helm releases with a custom configuration per cluster

Each cluster in the `clusters` dir has a `flux-system` directory with automatically-generated configuration (`clusters/<cluster>/flux-system`).

The `configs` dir is structured as a kustomization directory with a base directory named `configs/base` and an overlays directory named `configs/overlays`, which contains one overlay per cluster, each named `configs/overlays/<cluster>`.

The `configs/base` dir contains common configurations for all clusters. Overlays dirs contain patches per cluster.

## Example cluster: kind-local

Create a new repository using this template.

To test the configurations in this repo using [Kind](https://kind.sigs.k8s.io/) (Kubernetes-in-Docker), assuming the `kind` cli is [install](https://kind.sigs.k8s.io/docs/user/quick-start/#installation)ed, do:

```sh
kind create cluster
```

And follow the instructions for [FluxCD](https://fluxcd.io) [installation](https://fluxcd.io/docs/installation/) and [bootstrap](https://fluxcd.io/docs/get-started/#install-flux-onto-your-cluster), using your k8s-fleet repository as the target.

## Authors

**Andre Silva** - [@andreswebs](https://github.com/andreswebs)

## License

This project is licensed under the [Unlicense](UNLICENSE.md).
