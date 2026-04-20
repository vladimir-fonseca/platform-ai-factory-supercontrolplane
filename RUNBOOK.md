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
# Runbook

This runbook covers the refined open-source super control plane pattern.

## Troubleshooting order

Start at the lowest dependency:

1. management-cluster Cilium
2. Cluster API core and kubeadm providers
3. CAPM3 / Metal3
4. inventory / host selection
5. ClusterClass and provider templates
6. cluster object reconciliation
7. Argo CD cluster registration
8. ApplicationSet service-profile rollout
9. child-cluster addon health

---

## Management cluster summary

```bash
kubectl get applications -n argocd
kubectl get pods -A | grep cilium
kubectl get deployments -A | grep cluster-api
kubectl get deployments -A | grep metal3
kubectl get baremetalhosts -A
kubectl get clusterclasses.cluster.x-k8s.io -A
kubectl get clusters.cluster.x-k8s.io -A
kubectl get applicationsets -n argocd
kubectl get secrets -n argocd | grep cluster
```

---

## Cluster API

```bash
kubectl get pods -n capi-system
kubectl get pods -n ca-kubeadm-bootstrap-system
kubectl get pods -n ca-kubeadm-control-plane-system
kubectl get clusters.cluster.x-k8s.io -A
kubectl describe cluster h100-dev -n fleet
kubectl get machinedeployments -A
kubectl get kubeadmcontrolplanes -A
```

Typical issues:

- provider CRDs missing
- control-plane template mismatch
- machine template mismatch
- bootstrap data never generated

---

## CAPM3 / Metal3

```bash
kubectl get pods -n capm3-system
kubectl get baremetalhosts -A
kubectl describe baremetalhost h100-a -n metal3-inventory
kubectl get metal3clusters -A
kubectl get metal3machines -A
```

Typical issues:

- BMC auth failures
- inventory labels do not match machine selectors
- inspection/provisioning stuck
- host state mismatch

---

## Onboarding / Argo CD cluster registration

```bash
kubectl get jobs -n argocd
kubectl logs -n argocd job/argocd-cluster-register
kubectl get secrets -n argocd | grep cluster
```

Typical issues:

- kubeconfig secret not found
- job cannot reach Argo CD API
- cluster secret not created
- label selectors do not match ApplicationSet expectations

---

## ApplicationSet / service profile

```bash
kubectl get applicationsets -n argocd
kubectl describe applicationset nvidia-ai-stack -n argocd
kubectl get applications -n argocd
```

Typical issues:

- cluster label mismatch
- child-cluster server value missing
- addon render failures

---

## Child cluster addon health

```bash
kubectl --context h100-dev get pods -A | grep cilium
kubectl --context h100-dev get clusterpolicy
kubectl --context h100-dev get pods -n gpu-operator
kubectl --context h100-dev get pods -n dpf-operator-system
kubectl --context h100-dev get kubevirt -n kubevirt
kubectl --context h100-dev get cdi
```

---

## Recovery

### Re-sync management apps

```bash
argocd app sync management-cilium
argocd app sync cluster-api
argocd app sync capm3
argocd app sync inventory
argocd app sync catalog
argocd app sync onboarding
argocd app sync services
```

### Roll back
Preferred rollback path:

1. revert the bad Git commit
2. push the revert
3. let Argo CD reconcile

## Onboarding validation

For the CAPI packages, do not stop validation at child-cluster creation.

Also validate the onboarding layer explicitly.

### Validation sequence

1. confirm the child cluster exists from the Cluster API point of view
2. confirm the onboarding resources ran successfully
3. confirm the child cluster is registered into Argo CD
4. confirm the expected labels are present for `ApplicationSet` targeting
5. confirm the generated `nvidia-ai-stack` application appears for that cluster
6. confirm addon sync starts in the expected order

### Typical checks

```bash
kubectl get cluster -A
kubectl get job -A
kubectl get configmap -A
kubectl get serviceaccount -A
kubectl get role,rolebinding -A
kubectl get secret -n argocd
kubectl get applications -n argocd
kubectl describe applicationset nvidia-ai-stack -n argocd
```

### Common onboarding failure pattern

A child cluster may exist and still not receive the AI stack.

When that happens, the missing step is often one of these:

- the onboarding job did not run successfully
- cluster registration into Argo CD did not occur
- the expected cluster labels were not applied
- the `ApplicationSet` selector does not match the registered cluster
## Argo CD sync-wave plan


## Why the sync-wave table matters

## Why the sync-wave table matters

The sync-wave table makes rollout order auditable by inspection. Without it, the reader has to infer ordering from filenames or from annotations scattered across the `apps/` directory.

Keep the table in the docs even though the Application annotations remain the source of truth.

## Kyverno registration policy validation

Because this package includes a Kyverno-based cluster-registration policy, validate both:

1. the source child-cluster kubeconfig secret
2. the generated Argo CD cluster secret in `argocd`

Typical checks:

```bash
kubectl get cpol generate-argocd-cluster-secret-from-child-kubeconfig
kubectl get secret -n argocd
kubectl describe secret <generated-cluster-secret> -n argocd
kubectl get applications -n argocd
```

If the child cluster exists but the generated Argo CD secret does not appear, review:
- source secret name matching `*-kubeconfig`
- namespace exclusions
- source secret data keys
- expected labels for ApplicationSet targeting


## Multus and SR-IOV validation

In addition to the existing addon validation, verify the richer networking path:

```bash
kubectl get ds -n multus-system
kubectl get ds -n sriov-system
kubectl get cm -n sriov-system sriov-device-plugin-config
kubectl get network-attachment-definitions.k8s.cni.cncf.io -A
kubectl get sriovnetworknodepolicies.sriovnetwork.openshift.io -A
```

Review points:

- Multus daemonset should be present on every child-cluster worker
- SR-IOV CNI and SR-IOV device plugin should be present on accelerated workers
- NAD objects should reference the expected `resourceName`
- node-policy examples should match your PF names, VF counts, and switch mode
```

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

## Site model

Use the site-specific app-of-apps and bootstrap entrypoints to route child Applications to the correct overlay and target cluster.

## Explicit HBN / OVN model

Review the dedicated HBN child module under `managed-addons/nvidia-ai-stack/base/core/hbn/` instead of treating HBN as just another generic DPU service.

## Kyverno policy numbering note

Policy numbering intentionally skips `04` in this package lineage. That gap is preserved on purpose and does not indicate a missing manifest.
