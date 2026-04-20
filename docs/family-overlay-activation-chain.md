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
# Family overlay activation chain

## Intent

This note explains how the family-specific overlays are actually activated in the package.

The `prod-h100`, `prod-b200`, and `prod-gb200` overlays are not standalone app-of-apps entries. They are selector-driven targets chosen at runtime by the `ApplicationSet` after a child cluster has been registered into Argo CD.

The goal of this document is to make that activation path explicit so operators can understand why those overlays exist, when they are selected, and why they may appear to be "present but not instantiated" when reading only the app-of-apps layer.

## Architectural model

The activation path has four distinct stages:

1. the management cluster creates or adopts a child cluster
2. the child cluster is registered into Argo CD
3. the Argo CD cluster secret carries the labels used for addon targeting
4. the `ApplicationSet` renders the family-specific addon `Application` and points it at the correct overlay

That means the family-specific overlays live behind the registration boundary. They are not created directly by a static `apps/base/*.yaml` manifest.

## Why the overlays are not direct app-of-apps entries

The app-of-apps layer installs the shared `services` application, not one application per GPU family.

That shared `services` layer includes the `ApplicationSet`:

- `managed-addons/nvidia-ai-stack/base/core/applicationset/core/applicationset.yaml`

The `ApplicationSet` is the component that decides which overlay path should be used for each registered child cluster.

So the family overlays are:

- package-authored
- valid Kustomize entrypoints
- real deployment targets

But they are not instantiated until the `ApplicationSet` evaluates an Argo CD cluster secret with the expected labels.

## Full activation chain

### Step 1. Child-cluster object exists in the management plane

A child cluster is created or managed through the catalog and Cluster API layers.

Example cluster objects in the package already show the intended label model:

- `catalog/base/core/clusters/core/cluster-h100-dev.yaml`
- `catalog/base/core/clusters/core/cluster-b200-prod.yaml`
- `catalog/base/core/clusters/core/cluster-gb200-lab.yaml`

Those objects carry labels such as:

- `platform-profile: nvidia-ai`
- `gpu-family.nvidia.com/name: h100|b200|gb200`

Those labels represent the intended addon-targeting identity for the cluster.

### Step 2. Registration creates an Argo CD cluster secret

The child cluster must be registered into Argo CD before the addon stack can be attached.

In this package, that happens through the registration path documented in:

- `docs/kyverno-registration-path-assessment.md`

The important output of that step is an Argo CD cluster secret in the `argocd` namespace.

That cluster secret is the object the `ApplicationSet` actually watches.

### Step 3. The Argo CD cluster secret carries targeting labels

The `ApplicationSet` does not read the original Cluster API `Cluster` object directly.

It evaluates the labels on the Argo CD cluster secret.

The minimum gating label is:

- `platform-profile: nvidia-ai`

The family-routing label is:

- `gpu-family.nvidia.com/name`

If the registration path fails to preserve or generate those labels correctly, the `ApplicationSet` cannot select the intended family overlay.

### Step 4. The `ApplicationSet` selects the overlay path

Once the cluster secret exists and carries the expected labels, the `ApplicationSet` renders one addon `Application` per cluster.

The routing model is:

```text
gpu-family.nvidia.com/name   Rendered overlay path
--------------------------   -----------------------------------------------
h100, h200                   managed-addons/nvidia-ai-stack/overlays/prod-h100
b200, b300                   managed-addons/nvidia-ai-stack/overlays/prod-b200
gb200, gb300                 managed-addons/nvidia-ai-stack/overlays/prod-gb200
missing or unrecognized      managed-addons/nvidia-ai-stack/overlays/prod
```

So `prod-gb200` is not activated because it exists in Git. It is activated because the registered cluster secret carries `gpu-family.nvidia.com/name=gb200` and the `ApplicationSet` resolves that label into the `prod-gb200` overlay path.

## Practical example

For a GB200 child cluster, the activation chain is:

1. `cluster-gb200-lab.yaml` exists with `gpu-family.nvidia.com/name: gb200`
2. the registration path creates an Argo CD cluster secret for that cluster
3. that secret carries:
   - `platform-profile: nvidia-ai`
   - `gpu-family.nvidia.com/name: gb200`
4. the `ApplicationSet` renders:
   - `<cluster-name>-nvidia-ai-stack`
5. that generated `Application` uses:
   - `managed-addons/nvidia-ai-stack/overlays/prod-gb200`

At that point the `prod-gb200` overlay is instantiated indirectly through the generated Argo CD `Application`.

## Fallback behavior

If the family label is absent, the package falls back to:

- `managed-addons/nvidia-ai-stack/overlays/prod`

That fallback is intentional. It keeps the addon stack renderable even when the family label is missing, but it also means the family-specific MIG posture will not be applied.

Operationally, that makes missing family labels a correctness issue even if rendering still succeeds.

## Design intent

This design keeps the package cleaner than a flat model with one top-level app-of-apps entry per hardware family.

It separates:

- management-plane application installation
- cluster registration
- hardware-family targeting
- runtime addon attachment

That separation is deliberate. The family overlays are runtime-selected deployment targets, not static bootstrap objects.

## What to check when a family overlay does not activate

If `prod-h100`, `prod-b200`, or `prod-gb200` is not being selected, check the chain in this order:

1. is the child cluster registered into Argo CD at all
2. does the Argo CD cluster secret exist in `argocd`
3. does that secret carry `platform-profile: nvidia-ai`
4. does that secret carry the expected `gpu-family.nvidia.com/name`
5. does the generated addon `Application` point to the expected overlay path

If any of those steps fail, the family overlay will appear to exist in Git but never become active at runtime.
