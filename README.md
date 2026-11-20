[![kustomize-checks](https://github.com/platformfix/k8s-region-recovery-flux/actions/workflows/kustomize-checks.yaml/badge.svg)](https://github.com/platformfix/k8s-region-recovery-flux/actions/workflows/kustomize-checks.yaml)

k8s-region-recovery-flux
========================

The Git repository Flux reads in version V2 of Aleksander Laketa's diploma on
recovering Kubernetes after losing a whole cloud region. The infrastructure,
the runbooks, the timed runs and their results live in
[0Aleksander/k8s-region-recovery-infra](https://github.com/0Aleksander/k8s-region-recovery-infra);
this repository holds only what Flux installs from Git: the demo app k8coins.

The diploma defines four versions of the recovery. V1 rebuilds the platform
with Terraform. V2 is V1 with one change: Flux, not Terraform, installs the app,
from this repository. Terraform still builds the cluster, the ingress
controller, Velero (the backup tool) and Flux itself, which it installs with the
Flux Operator and points at the directory `kustomize/k8srr`.

This repository lives in Platform Fix's GitHub organisation for now and is due
to move to Aleks's account. The infra repo keeps its address in one place
(`docs/flux-repo.md` there), so the move changes one line.

What is here
------------

```
kustomize
├── k8srr            # the directory both regions' Flux syncs: one Flux Kustomization, k8coins
└── _base
    └── k8coins      # the app: its namespace, where its Helm chart comes from, and its HelmRelease
```

- `kustomize/k8srr/k8coins.yaml` is the Flux Kustomization that installs the
  app. It starts **suspended**, in both regions. In the recovery region the
  Redis data has to be restored from the Velero backup before Redis exists,
  because Velero never overwrites an object that is already there; the V2
  runbook restores the disk and then runs `flux resume kustomization k8coins
  -n flux-system`. The file's comment explains this in full, including why
  the annotation `kustomize.toolkit.fluxcd.io/ssa: IfNotPresent` stops Flux
  from suspending it again.
- `kustomize/_base/k8coins/helmrelease.yaml` has the same chart version and
  the same values as the Terraform release in V1
  (`stacks/region/k8coins.tf` in the infra repo), so V1 and V2 run the same
  app and differ only in which tool installs it.

How Flux reaches this repository
--------------------------------

The repository is private. Each cluster's Flux reads it over SSH with a
read-only deploy key. The key is created by Terraform in the infra repo's
shared stack; its private half stays in Terraform's state and in the cluster,
and nothing secret is stored here. The infra repo's `docs/flux-repo.md`
explains how the key is registered and rotated.

Credit
------

Started from Steve Wade's
[swade1987/flux2-kustomize-template](https://github.com/swade1987/flux2-kustomize-template):
the `kustomize/_base` layout, the CI checks and the contribution rules. Its
example apps, its Istio check, its release workflow and its Scorecard workflow
were removed because this repository has no use for them.

Working on it
-------------

Install the pre-commit hooks once:

```
make initialise
```

Commits and pull request titles follow
[Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/), and
every commit is signed off (`git commit -s`,
[Developer Certificate of Origin](https://developercertificate.org/)). CI
builds every directory under `kustomize/` and validates it with kubeconform and
pluto ([docs/ci.md](docs/ci.md)). To run the checks locally:

```
make local-check-duplicate-release-name
make local-kubeconform-checks
make local-pluto-checks
make local-kustomization-yaml-fix
```

See [CONTRIBUTING.md](CONTRIBUTING.md) for the contribution rules.
