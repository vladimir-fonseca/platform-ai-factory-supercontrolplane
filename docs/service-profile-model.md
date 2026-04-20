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
# Service Profile Model

## Intent

A service profile is a reusable addon bundle attached by cluster label.

This repo uses:

- `ApplicationSet` for selection and fan-out
- `ClusterClass` for reusable cluster shapes
- Git folders as the service catalog

## Example label flow

- cluster labeled `platform-profile=nvidia-ai`
- Argo CD `ApplicationSet` generates one application per child cluster
- that application deploys:
  - Cilium
  - GPU Operator
  - DPF
  - KubeVirt
  - CDI
