<!--
-----------------------------------------------------------------------------
Copyright (c) 2026 Vladimir Fonseca

This file is part of the platform-ai-factory-supercontrolplane project.

Defines platform architecture, integration patterns, and deployment models.

Third-party components referenced here remain the property of their
respective owners.

Licensed under the Apache License, Version 2.0.
-----------------------------------------------------------------------------

-->
# Kyverno cluster registration

This package includes a declarative Kyverno-based cluster-registration pattern.

## Why

The package already has an `onboarding/` layer, but the Kyverno variant benefits from a
policy-driven registration mechanism for the steady-state path because it is:

- declarative
- easier to audit
- easier to keep synchronized over time

## Key policy

- `kyverno/base/core/policies/10-generate-argocd-cluster-secret-from-child-kubeconfig.yaml`

## Flow

1. a child-cluster kubeconfig secret appears
2. Kyverno matches it
3. Kyverno generates an Argo CD cluster secret in `argocd`
4. labels such as `platform-profile=nvidia-ai` are applied
5. the ApplicationSet can target the cluster and attach `nvidia-ai-stack`

## Important note

This package still keeps the onboarding layer because onboarding may include additional
registration or labeling logic. The Kyverno policy is the preferred declarative
registration mechanism for the cluster-secret generation step.

## Kyverno policy numbering note

Policy numbering intentionally skips `04` in this package lineage. That gap is preserved on purpose and does not indicate a missing manifest.
