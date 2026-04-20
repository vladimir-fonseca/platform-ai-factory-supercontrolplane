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
# Placeholder audit before deployment

## Intent

This note records the placeholder values that are intentionally left in the example manifests and must be replaced before the package is used in any real environment.

## Required substitutions

The following must be substituted with real values before any environment use:

- BMC addresses in `inventory/base/core/*/core/*.yaml`
- Image URLs such as `images.example.com` in the inventory layer and Metal3 machine templates
- API server values in `bootstrap/common/supercontrolplane-project.yaml` and `apps/overlays/*/patches/services.yaml`
- OCI chart registries such as `registry.example.com` in the `cluster-api`, `capm3`, and `managed-addons` base kustomizations
- Storage backend endpoints such as `block-target.storage.example.com`, `posix-nfs-vip.storage.example.com`, and the OVN northbound / southbound database endpoints
- In the Kyverno package, policy `10-generate-argocd-cluster-secret-from-child-kubeconfig.yaml` derives the Argo CD `server:` field as `https://{{ clusterName }}-api.example.com:6443`. That value must match the actual cluster API DNS convention used in the target environment.

## Review posture

Treat these placeholders as review markers, not defaults. They are useful in a reference package because they make the required integration points explicit, but they should not survive unchanged into an environment-specific deployment repository.
