# Security Policy

## Supported versions

This template ships one rolling `latest` semantic-release version; there's no long-term-support branch to track. Fixes land on `main` and are published as the next tagged release.

## Reporting a vulnerability

Please report security issues privately rather than opening a public GitHub issue: use [GitHub's private vulnerability reporting](https://github.com/platformfix/k8s-region-recovery-flux/security/advisories/new) for this repository (Security tab → Report a vulnerability).

Include what you'd include in any good bug report: the affected version or commit, what you found, and how to reproduce it. We'll acknowledge new reports within 5 business days and aim to have a fix or mitigation plan within 30 days, depending on severity.

## Scope

The CI checks (kubeconform, pluto, duplicate-release-name-checks, kustomize-diff), the scripts under `bin/` and the manifests under `kustomize/` are in scope. Flux in each cluster reads this repository with a read-only deploy key; nothing secret is stored here.
