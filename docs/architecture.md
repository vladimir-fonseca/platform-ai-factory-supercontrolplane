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
# Architecture

## Goal

Provide a more concrete open-source “super control plane” model for NVIDIA AI infrastructure.

## Core layers

### Management cluster
Runs:

- Argo CD
- Cilium
- Cluster API core
- kubeadm bootstrap/control-plane providers
- CAPM3 / Metal3
- inventory
- catalog
- onboarding automation
- addon catalog

### Catalog
Defines reusable cluster shapes with `ClusterClass`.

### Inventory
Defines bare-metal inventory and any selectors/pools used to bind cluster machines to specific hardware families.

### Onboarding
Registers created child clusters into Argo CD.

### Service profile
Attaches a standard NVIDIA AI bundle to matching child clusters.

## Why this resembles k0rdent

The operational pattern is the same in spirit:

- one management plane
- reusable templates
- reusable services
- declarative cluster lifecycle
- central fleet operations

The implementation here is upstream/open-source.

## Site model

Use the site-specific app-of-apps and bootstrap entrypoints to route child Applications to the correct overlay and target cluster.

## Explicit HBN / OVN model

Review the dedicated HBN child module under `managed-addons/nvidia-ai-stack/base/core/hbn/` instead of treating HBN as just another generic DPU service.

## Design posture

The package is intentionally opinionated.

The management plane is modeled as the source of lifecycle truth. Inventory, catalog, onboarding, and policy are separated because they answer different operational questions. On the child-cluster side, DPF, HBN / OVN, and storage-oriented DPU services are separated for the same reason: they have different ownership, different failure domains, and different review criteria.

That separation is not cosmetic. It is what keeps the package understandable when reviewed by platform, networking, storage, and security stakeholders.
