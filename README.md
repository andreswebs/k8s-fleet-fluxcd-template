# k8s-fleet

This repository is a template containing intial Kubernetes configuration files
for a multi-cluster k8s fleet.

## Structure

It uses the following top-level directory structure:

- `clusters/`: contains the Flux configuration per cluster under
  `clusters/<cluster>`
- `sources/`: source definitions common to all clusters
- `infrastructure-base/`: kustomizations or Helm releases with a custom
  configuration per cluster, for Kubernetes add-ons and other base applications
  that are dependencies for other apps
- `secrets/`: custom resources managed by the External Secrets Operator, to
  handle secrets stored in external platforms
- `infrastructure/`: kustomizations or Helm releases with a custom configuration
  per cluster, for Kubernetes add-ons and other base applications that are
  dependencies for other apps, and which depend on add-ons from
  `infrastructure-base/` and/or resources from `secrets/`
- `apps/`: kustomizations or Helm releases with a custom configuration per
  cluster, for any other apps

Each cluster in the `clusters` directory has a `flux-system` directory with
automatically-generated configuration (`clusters/<cluster>/flux-system`).

The `infrastructure-base`, `secrets`, `infrastructure` and `apps` directories
are structured as kustomization directories with a base directory named
`<dir>/base` and an overlays directory named `<dir>/overlays`, which contains
one overlay per cluster, each named `<dir>/overlays/<cluster>`. The `<dir>/base`
dirs contain common configurations for all clusters. Overlays dirs
(`<dir>/overlays/<cluster>`) contain patches per cluster.

The `secrets/base` directoy contains two sub-directories: `secrets/base/stores`
and `secrets/base/secrets`, which group these two types of resources managed by
the External Secrets Operator, respectively: secret stores and secrets.

## Example cluster: kind-local

Create a new repository using this template.

To test the configurations in this repo using [Kind](https://kind.sigs.k8s.io/)
(Kubernetes-in-Docker), assuming the `kind` cli is
[install](https://kind.sigs.k8s.io/docs/user/quick-start/#installation)ed, do:

```sh
kind create cluster
```

And follow the instructions for [FluxCD](https://fluxcd.io)
[installation](https://fluxcd.io/docs/installation/) and
[bootstrap](https://fluxcd.io/docs/get-started/#install-flux-onto-your-cluster),
using your k8s-fleet repository as the target.

Ensure that the path for the cluster is specified as `clusters/kind-local`.

## Authors

**Andre Silva** - [@andreswebs](https://github.com/andreswebs)

## License

This project is licensed under the [Unlicense](UNLICENSE.md).
