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
# platform-ai-factory-supercontrolplane

This package defines a Kubernetes management plane that does three things:

- provisions bare-metal child clusters through Cluster API and CAPM3
- registers those child clusters into Argo CD
- attaches a structured NVIDIA AI stack to each child cluster through ApplicationSet-driven overlays

The management plane owns cluster lifecycle, registration, and policy. The child-cluster side keeps the NVIDIA stack explicit so networking, DPF, storage integration, virtualization, and runtime services can be reviewed and changed independently.


## What this package does

At runtime, the package is intended to work as a control loop chain:

1. the management cluster installs networking, Cluster API, CAPM3, inventory, catalog, policy, services, and onboarding in a defined sync-wave order
2. Cluster API creates or manages bare-metal child clusters from the catalog and inventory model
3. the registration path creates Argo CD cluster secrets for those child clusters
4. the ApplicationSet selects the correct NVIDIA AI overlay for each cluster
5. the child cluster receives the NVIDIA platform stack, including GPU, DPF, networking, storage-facing services, and optional virtualization layers

This means the package is not just a collection of manifests. It is a management-plane design for building, registering, and attaching an NVIDIA-oriented child-cluster platform.

## What is modeled here

The package is organized around four management-plane responsibilities:

**catalog**  
Defines the reusable cluster blueprint, templates, and example child clusters.

**inventory**  
Defines the host inventory and the infrastructure-facing objects used for bare-metal selection and provisioning.

**onboarding**  
Handles the registration boundary between a newly available child cluster and the GitOps control plane.

**services**  
Attaches the NVIDIA AI platform stack to registered child clusters through `ApplicationSet`.

That separation is intentional. It keeps cluster definition, host inventory, registration, and addon attachment as distinct concerns instead of collapsing them into one layer.

The package should be read as a reference architecture and implementation scaffold for that control flow, not as a finished environment-specific deployment.

## What this repo contains

This repository is split into logical layers:

1. **Management Cilium**
   - installs the management-cluster networking baseline

2. **Cluster API**
   - installs Cluster API core and kubeadm providers
   - defines the management-plane cluster lifecycle foundation

3. **CAPM3**
   - installs the Metal3-backed infrastructure provider

4. **Inventory**
   - defines example management-plane `BareMetalHost` inventory objects

5. **Catalog**
   - defines reusable ClusterClass, templates, and example child clusters

6. **Onboarding**
   - registers newly created child clusters back into Argo CD
   - prepares them for `ApplicationSet` targeting and addon fan-out

7. **Kyverno**
   - installs the Kyverno policy engine
   - applies cluster policies for admission and default-deny baselines

8. **Managed addons**
   - defines the structured `nvidia-ai-stack` profile attached to child clusters

## Repository layout

```text
platform-ai-factory-supercontrolplane/
├── README.md
├── RUNBOOK.md
├── docs/
│   ├── architecture.md
│   ├── cluster-registration.md
│   └── service-profile-model.md
├── apps/
│   ├── root.yaml
│   ├── management-cilium.yaml
│   ├── cluster-api.yaml
│   ├── capm3.yaml
│   ├── inventory.yaml
│   ├── catalog.yaml
│   ├── onboarding.yaml
│   └── services.yaml
├── management-cilium/
│   ├── base/
│   └── overlays/
│       └── prod/
├── cluster-api/
│   ├── base/
│   └── overlays/
│       └── prod/
├── capm3/
│   ├── base/
│   └── overlays/
│       └── prod/
├── inventory/
│   ├── base/
│   └── overlays/
│       └── prod/
├── catalog/
│   ├── base/
│   └── overlays/
│       └── prod/
├── onboarding/
│   ├── base/
│   └── overlays/
│       └── prod/
└── managed-addons/
    └── nvidia-ai-stack/
        ├── base/
        └── overlays/
            └── prod/
```

---

## High-level flow

### 1. Management cluster bootstraps itself
Argo CD reconciles:

- Cilium
- CAPI core
- CAPM3 / Metal3
- inventory
- catalog
- onboarding flows
- addon service profiles

### 2. Cluster catalog defines “golden path” shapes
A `ClusterClass` plus provider templates define:

- control plane shape
- worker machine shape
- versioning pattern
- bootstrap defaults
- network defaults
- host selectors

### 3. Cluster objects instantiate child clusters
A `Cluster` object declares:

- the chosen `ClusterClass`
- Kubernetes version
- replica counts
- family/profile labels

### 4. Onboarding registers child clusters into Argo CD
Once a child cluster exists and its kubeconfig is available, the onboarding flow registers it with Argo CD so `ApplicationSet` can target it.

### 5. Service profile applies the NVIDIA AI bundle
Clusters labeled `platform-profile=nvidia-ai` receive:

- Cilium
- GPU Operator
- DPF
- KubeVirt
- CDI

---

## Deployment order

```text
Stage  Layer / concern                                 Path
-----  ----------------------------------------------  -----------------------------------------------
1      Bootstrap project boundary                      bootstrap/common/supercontrolplane-project.yaml
2      Site bootstrap entrypoint                       bootstrap/<site>/root-application.yaml
3      Site app-of-apps overlay                        apps/overlays/<site>/
4      Management-plane networking baseline            networking/management-cilium/overlays/<site>/
5      Cluster API control plane                       cluster-api/overlays/<site>/
6      CAPM3 bare-metal integration                    capm3/overlays/<site>/
7      Inventory and host selection                    inventory/overlays/<site>/
8      Catalog and reusable cluster definitions        catalog/overlays/<site>/
9      Child-cluster onboarding workflow               onboarding/overlays/<site>/
10     Management-plane policy boundary                kyverno/overlays/<site>/
11     Child-cluster NVIDIA AI addon stack            managed-addons/nvidia-ai-stack/overlays/<site>/
```

This is the operational dependency chain. The sync-wave table above is the source of truth for Argo CD ordering.


## Important caveats

This repo intentionally uses:

- **placeholders** for BMC endpoints, chart repos, and image URLs
- **modeled examples** for CAPI provider templates
- **example onboarding job/manifests** for cluster registration

You must adapt them to:

- your exact CAPI provider versions
- your exact CAPM3 / Metal3 topology
- your secret-management model
- your Argo CD authentication / cluster-registration model
- your GPU / DPU support matrix

---

## Bootstrap

```bash
kubectl apply -f apps/root.yaml -n argocd
```

---

## Local render checks

```bash
kustomize build --enable-helm networking/networking/management-cilium/overlays/prod
kustomize build --enable-helm cluster-api/overlays/prod
kustomize build --enable-helm capm3/overlays/prod
kustomize build inventory/overlays/prod
kustomize build catalog/overlays/prod
kustomize build onboarding/overlays/prod
kustomize build --enable-helm managed-addons/nvidia-ai-stack/overlays/prod
```

---

## High-level validation

```bash
kubectl get applications -n argocd
kubectl get pods -A | grep cilium
kubectl get clusterclasses.cluster.x-k8s.io -A
kubectl get clusters.cluster.x-k8s.io -A
kubectl get metal3clusters.infrastructure.cluster.x-k8s.io -A
kubectl get metal3machinetemplates.infrastructure.cluster.x-k8s.io -A
kubectl get baremetalhosts -A
kubectl get applicationsets -n argocd
kubectl get secrets -n argocd | grep cluster
```

## Onboarding layer

These CAPI packages include an explicit **`onboarding/`** layer, and it is part of the intended package flow.

Its job is to bridge the gap between:

- child-cluster creation through the management-plane templates
- registration of the new child cluster into Argo CD
- application of the labels and metadata needed for targeting
- `ApplicationSet` fan-out of `managed-addons/nvidia-ai-stack/`

In practical terms, **cluster creation is not the end of the flow**. The onboarding layer is what turns a newly created child cluster into a GitOps-managed child cluster that can actually receive the NVIDIA AI addon stack.

### What to review in `onboarding/`

Review this layer in the same way as the others:

1. `onboarding/base/kustomization.yaml`
2. `onboarding/base/core/`
3. `onboarding/base/values/`
4. `onboarding/overlays/prod/`

Focus on:

- service account and RBAC scope
- registration script or job behavior
- cluster registration mechanics into Argo CD
- labels required for `ApplicationSet` selection
- failure handling when the cluster exists but addon attachment does not happen
## Argo CD sync-wave plan

```text
Wave  Application         Path                                           Target namespace   Operational intent
----  ------------------  ---------------------------------------------  -----------------  ------------------------------
root  supercontrol-root   apps/overlays/<site>                           argocd             Root app-of-apps bootstrap
-30   management-cilium   networking/management-cilium/overlays/<site>   kube-system        Child platform layer rollout
-20   cluster-api         cluster-api/overlays/<site>                    capi-system        Child platform layer rollout
-15   capm3               capm3/overlays/<site>                          capm3-system       Child platform layer rollout
-10   inventory           inventory/overlays/<site>                      metal3-inventory   Child platform layer rollout
-5    catalog             catalog/overlays/<site>                        fleet              Child platform layer rollout
-3    kyverno             kyverno/overlays/<site>                        kyverno            Child platform layer rollout
-2    services            managed-addons/nvidia-ai-stack/overlays/<site> argocd             Child platform layer rollout
0     onboarding          onboarding/overlays/<site>                     argocd             Child platform layer rollout
```

Concrete site examples shipped in this package:

- `dev`
- `prod`
- `prod-vast`
- `prod-weka`


## Other important package elements

## Other important package elements

These packages should keep the following elements explicit in the docs:

- Argo CD sync-wave ordering
- actual repository layout
- review order for `apps/`, `base/core/`, `base/values/`, and overlays
- environment-specific placeholders that must be replaced
- distinction between foundational control-plane objects and example objects
- common validation commands and failure modes
- package-specific intent, not only generic platform text

These should not be removed when simplifying or reformatting the docs.

## Kyverno-based cluster registration

This package adds a dedicated Kyverno generate policy for child-cluster registration into Argo CD.

Policy:
- `kyverno/base/core/policies/10-generate-argocd-cluster-secret-from-child-kubeconfig.yaml`

Intent:
- watch child-cluster kubeconfig secrets
- generate the Argo CD cluster secret in `argocd`
- apply labels such as `platform-profile=nvidia-ai`
- keep the generated secret synchronized

This is closer to a declarative GitOps pattern than relying only on a scaffold onboarding job.

---

## Rich child-cluster networking content

This package includes a richer child-cluster networking path under:

- `managed-addons/nvidia-ai-stack/base/multus/`
- `managed-addons/nvidia-ai-stack/base/sriov/`

That content expands the package beyond the earlier minimal stack by adding:

1. **Multus**
   - installs the NetworkAttachmentDefinition CRD
   - installs the Multus daemonset and its RBAC objects
   - defines example SR-IOV-backed secondary networks for management and tenant traffic

2. **SR-IOV**
   - installs the SR-IOV device plugin daemonset
   - installs the SR-IOV CNI binary daemonset
   - publishes explicit device-plugin resource mappings
   - includes an example `SriovNetworkNodePolicy`

This makes the package materially richer for KubeVirt plus DPU plus accelerated-networking scenarios.

---

## Base folder consistency guarantee

Every module in this package now follows the same base layout discipline:

- `base/core/`
- `base/values/`

This applies to:

- `management-cilium/`
- `cluster-api/`
- `capm3/`
- `inventory/`
- `catalog/`
- `onboarding/`
- `kyverno/`
- `managed-addons/nvidia-ai-stack/`

Within `managed-addons/nvidia-ai-stack/`, the richer child-cluster content is still expanded, but it is now organized under `base/core/` instead of mixing top-level folders directly inside `base/`.


## Multus and SR-IOV packaging model

The child-cluster networking add-on path uses the same structural discipline as the rest of the package:

- `managed-addons/nvidia-ai-stack/base/core/multus/core/`
- `managed-addons/nvidia-ai-stack/base/core/multus/values/`
- `managed-addons/nvidia-ai-stack/base/core/sriov/core/`
- `managed-addons/nvidia-ai-stack/base/core/sriov/values/`

Design intent:

- install **Multus** through `helmCharts` where possible
- install **SR-IOV operator/device-plugin** through `helmCharts` where possible
- keep authored CRDs, example `NetworkAttachmentDefinition` objects, device-plugin config, and example SR-IOV policies in `core/`
- keep chart configuration in `values/`

This keeps the package richer while preserving the base/core + base/values model you requested.


## Child-module consistency inside the addon stack

Every child module under `managed-addons/nvidia-ai-stack/base/core/` now follows the same pattern:

- `core/`
- `values/`

This applies to:

- `applicationset/`
- `namespaces/`
- `platform-core/`
- `dpf/`
- `dpu-services/`
- `multus/`
- `sriov/`

This avoids the earlier mixed folder styles and keeps even the richer networking and DPU content structured the same way as the rest of the package.

## Kubernetes 1.33 target

This package now targets **Kubernetes `v1.33.10`** for the example child clusters and the DPU cluster control plane.

Versioning intent:

- child-cluster examples in `catalog/base/core/` now use `v1.33.10`
- the DPU cluster object in `managed-addons/nvidia-ai-stack/base/core/dpf/core/dpucluster.yaml` uses `v1.33.10`

Before adopting this in a real environment, validate the exact NVIDIA operator versions, host OS, runtime, and hardware combination that matches your deployment baseline.

## Support matrix reference

See:

- `docs/support-matrix-kubernetes-1.33.md`

That file records the Kubernetes 1.33 compatibility summary and the caution about keeping package chart pins aligned with the exact validated platform combination.


## Catalog child-module structure

The catalog layer avoids a flat `catalog/base/core/` layout.

Instead, the catalog core is split into child modules, each of which follows the same pattern:

- `catalog/base/core/<child-module>/core/`
- `catalog/base/core/<child-module>/values/`

This applies to:

- `namespace/`
- `clusterclass/`
- `clusters/`
- `metal3-cluster-template/`
- `metal3-machine-template/`
- `kubeadm-controlplane-template/`
- `kubeadm-config-template/`

This keeps the catalog richer without turning `catalog/base/core/` into a dumping ground.


## Inventory child-module structure

The inventory layer avoids a flat `inventory/base/core/` layout.

Instead, inventory is split by NVIDIA hardware class, and each child module follows the same pattern:

- `inventory/base/core/<child-module>/core/`
- `inventory/base/core/<child-module>/values/`

This applies to:

- `namespace/`
- `hopper/`
- `blackwell/`
- `blackwell-ultra/`

That makes the inventory structure easier to grow as additional NVIDIA classes or
host examples are added later.

## WEKA virtio-fs overlay

This package now also includes a WEKA shared-POSIX overlay:

- `managed-addons/nvidia-ai-stack/overlays/weka/`
- `managed-addons/nvidia-ai-stack/overlays/prod-weka/`

It reuses the existing `snap-virtiofs` service and patches the backend-specific storage configuration for a WEKA POSIX client workflow.

See:

- `docs/weka-posix-virtiofs.md`

## WEKA performance profile

The WEKA virtio-fs path is now documented and packaged as a performance-oriented default:

- pinned to `arm64`
- fixed CPU and memory requests/limits in the service template
- explicit CPU-pinning style settings
- WEKA overlay mount options that bias toward dedicated-core execution

See:

- `docs/weka-posix-virtiofs.md`

## Explicit HBN / OVN model

The child-cluster addon stack now models HBN explicitly under `managed-addons/nvidia-ai-stack/base/core/hbn/` instead of burying it inside generic DPU services.

## Package intent

This repository models a supercontrolplane for NVIDIA-oriented bare-metal and child-cluster lifecycle management.

The management-plane side stays centered on Cluster API, CAPM3, inventory, catalog, onboarding, and, in the Kyverno variant, an explicit policy boundary. The child-cluster side keeps the addon stack explicit rather than collapsing DPF, HBN, and storage-oriented DPU services into a single opaque layer.

The goal is clarity of ownership and operational intent. Each layer should read as an architectural decision, not as a generated scaffold.

## Networking layout

The management-cluster Cilium baseline lives under `networking/management-cilium/`. Child-cluster networking addons such as Multus and SR-IOV remain under the NVIDIA AI addon stack because they belong to the child-cluster service boundary, not the management-plane networking boundary.

## Placeholder substitutions before deployment

The package intentionally uses placeholder values in the example manifests. Before using it in any real environment, replace at least the following:

- BMC endpoints and credentials references in the inventory layer
- image URLs and checksums used by inventory or Metal3 templates
- API server and cluster DNS placeholders used by bootstrap, app patches, and generated registration objects
- example OCI registries and storage backend endpoints
- OVN northbound and southbound database endpoints
- any example storage VIPs, NFS targets, or block targets

Treat these placeholders as review markers, not defaults.

## Kyverno policy numbering note

Policy numbering intentionally skips `04` in this package lineage. That gap is preserved on purpose and does not indicate a missing manifest.

## Placeholder audit note

Review `docs/placeholder-audit-before-deployment.md` for the placeholder values that must be substituted before any environment-specific use.

## Blockers-fixed variant

Review `docs/blockers-fixed-package-notes.md` for the package-specific summary of the first five blocker fixes.

## Minimal secret-management pattern

Review `docs/minimal-secret-management-pattern.md` for the package-side secret-management pattern and the environment-facing Secret templates under `secret-management/templates/`.

## Kyverno registration implementation note

Review `docs/fourth-iteration-notes.md` and `docs/kyverno-registration-path-assessment.md` for the registration-path notes used in this package.

## KubeVirt feature expansion

Review `docs/kubevirt-feature-expansion.md` for the KubeVirt feature-gate posture used in this package.

## MIG posture

Review `docs/mig-posture.md` for the hardware-family-oriented MIG model and the overlay routing used by the ApplicationSet.

## Family overlay activation chain

Review `docs/family-overlay-activation-chain.md` for the full activation chain that explains how family-specific overlays are selected and instantiated at runtime.

## CAPM3 version pinning

CAPM3 controller image pinning is defined in the CAPM3 overlays rather than the shared base, so environment-specific controller qualification stays at the overlay boundary.

## Ironic image pinning

The Ironic image pin is patched in the CAPM3 overlays so the validated runtime image stays environment-owned instead of being hardcoded in the shared base.

## DPF profile sizing

Review `docs/dpf-profile-sizing.md` for the family- and site-oriented DPF sizing and flavor model used by the overlays.

## GitOps and FedRAMP remediation plan

Review `docs/gitops-and-fedramp-remediation-plan.md` for the current GitOps control assessment and the prioritized FedRAMP-oriented remediation plan.


## Third-Party Components

See [THIRD_PARTY.md](./THIRD_PARTY.md) for details.


## Trademarks

NVIDIA, CUDA, GPUDirect, DOCA, BlueField and related marks are trademarks
of NVIDIA Corporation.

All other trademarks are the property of their respective owners.

This project is not affiliated with or endorsed by NVIDIA Corporation.



## Disclaimer

These repositories represent my personal reference architectures and open-source contributions.

They are not official NVIDIA or other vendor products.
