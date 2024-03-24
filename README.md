# k8s-fleet

## Structure

This repository has the following directory structure:

```
.
├── README.md
├── .gitignore
├── .sourceignore
├── .editorconfig
├── .yamllint
├── applications
│   ├── base
│   └── overlays
├── clusters
│   ├── kind-local
|   |   ├── flux-system
|   |   ├── apps.yaml
|   |   ├── infra-base.yaml
|   |   ├── infra.yaml
|   |   ├── secrets.yaml
|   |   └── sources.yaml
│   └── shared-deployer
├── infrastructure
│   ├── base
│   └── overlays
├── infrastructure-base
│   ├── base
│   └── overlays
├── secrets
│   ├── base
│   └── overlays
└── sources
    ├── <source-n>.yaml
    ├── (...)
    └── kustomization.yaml
```

### Directories

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
- `applications/`: kustomizations or Helm releases with a custom configuration
  per cluster, for any other apps

Each cluster in the `clusters` directory has a `flux-system` directory with
automatically-generated configuration (`clusters/<cluster>/flux-system`)
injected by FluxCD on bootstrap (described below).

### Order of kustomizations

The `infrastructure-base`, `secrets`, `infrastructure` and `applications`
directories are structured as kustomization directories. These directories are
deployed as FluxCD `Kustomization` resources.

Each kustomization directory has a base directory named `<dir>/base` and an
overlays directory named `<dir>/overlays`. Overlays directories contain one
overlay per cluster, each named `<dir>/overlays/<cluster>`. The `<dir>/base`
directories contain common configurations ("bases") used across clusters. The
base directories contain "bases" distributed in subdirectories of the
`<dir>/base`. Overlays directories (`<dir>/overlays/<cluster>`) contain patches
per cluster over the defined bases.

The `source` directory doesn't contain a base and overlays subdirectories, but
it is also considered a kustomization. It currently doesn't have any
cluster-specific configuration, and is applied without variance to all clusters.
This could be further configured to accomodate cluster differences in sources if
desired, by following the same structure.

The set of Kustomization resources in this repository is applied in a defined
order in which each kustomization depends on the previous one.

The order is defined implicitly within the Flux configuration directory for each
cluster (`clusters/<cluster>`). Each `Kustomization` used by a cluster is
defined in a separate file in the cluster directory. Kustomizations declare the
dependencies for that kustomization in their `spec.dependsOn` property.

The configured order for this repository is the following (the list uses the
deployed Kustomization resource names):

1. sources
2. infra-base
3. secrets
4. infra
5. apps

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
using your repository as the target.

Ensure that the path for the cluster is specified as `clusters/kind-local`.

## Example: Bootstrap Flux with deploy key on GitHub

First generate a deploy key. Assuming the keys to be saved under
`./keys/kind-local`, run:

```sh
export KEY_DIR="$(pwd)/keys/kind-local"
```

```sh
mkdir -p "${KEY_DIR}"
ssh-keygen -t ed25519 -f "${KEY_DIR}/identity" -q -N "" -C "" < /dev/null
```

To get the contents of the public key:

```sh
cat "${KEY_DIR}/identity.pub"
```

Add the public key to the GitHub repository (see
[instructions](https://docs.github.com/en/developers/overview/managing-deploy-keys#setup-2)).

Bootstrap flux:

```sh
export CLUSTER_NAME="mu"
export GITHUB_OWNER="example-org"
export GITHUB_REPO="example-k8s-fleet"
export GITHUB_BRANCH="main"
```

```sh
flux bootstrap git \
  --url="ssh://git@github.com/${GITHUB_OWNER}/${GITHUB_REPO}" \
  --branch="${GITHUB_BRANCH}" \
  --private-key-file="${KEY_DIR}/identity" \
  --path="clusters/${CLUSTER_NAME}"
```

(Note: add the `--password` flag if the private key has a password.)

## Authors

**Andre Silva** - [@andreswebs](https://github.com/andreswebs)

## License

This project is licensed under the [Unlicense](UNLICENSE.md).
