[![kustomize-checks](https://github.com/platformfix/k8s-region-recovery-flux/actions/workflows/kustomize-checks.yaml/badge.svg)](https://github.com/platformfix/k8s-region-recovery-flux/actions/workflows/kustomize-checks.yaml)

k8s-region-recovery-flux
========================

The Git repository Flux reads in versions V2 and V3 of Aleksander Laketa's
diploma on recovering Kubernetes after losing a whole cloud region. The infrastructure,
the runbooks, the timed runs and their results live in
[0Aleksander/k8s-region-recovery-infra](https://github.com/0Aleksander/k8s-region-recovery-infra);
this repository holds only what Flux installs from Git: the demo app k8coins.

The diploma defines four versions of the recovery, each adding one thing to the
one before. V0 is done by hand. V1 rebuilds the platform with Terraform and
restores the data by following a written runbook. V2 is
V1 with one change: Flux installs the app, from this repository, instead of
Terraform (in the source region) and the restore (in the recovery region). V3 is V2 with one change: one script restores the data instead of
a runbook. In V2 and V3, Terraform still builds the cluster, the ingress
controller, Velero (the backup tool) and Flux itself, which it installs with the
Flux Operator and points at the directory `kustomize/k8srr`.

This repository lives in Platform Fix's GitHub organisation and is due to move
to Aleks's account. The infra repo sets its address in one variable,
`flux_git_url` in `stacks/shared/variables.tf`; its `docs/flux-repo.md` lists
the steps of the move.

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
  because Velero never overwrites an object that is already there. The V2
  runbook restores the disk and then runs `flux resume kustomization k8coins
  -n flux-system`; in V3, the infra repo's `scripts/restore-data.sh` does the
  same steps. The file's comment explains the V2 case in full, including why
  the annotation `kustomize.toolkit.fluxcd.io/ssa: IfNotPresent` stops Flux
  from suspending it again.
- `kustomize/_base/k8coins/helmrelease.yaml` has the same values as the
  Terraform release in the infra repo (`stacks/region/k8coins.tf`), and
  `kustomize/_base/k8coins/ocirepository.yaml` pins the same chart version,
  0.7.0 (the default of `k8coins_chart_version` in the infra repo's
  `stacks/region/variables.tf`), so V1, V2 and V3 run the same app, whichever
  tool installs it.

How Flux reaches this repository
--------------------------------

The repository is public, and each cluster's Flux reads it anonymously over
HTTPS, with no credential. Nothing in it is secret: it holds only the app's
manifests and Helm values. A read-only deploy key was the first plan, but the
Platform Fix GitHub organisation disables deploy keys for its repositories. If
the repository moves to an account where it is private, Flux will need a
credential again; the infra repo's `docs/flux-repo.md` covers that cutover.

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
