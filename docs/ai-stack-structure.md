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
# AI stack structure

This package organizes the child-cluster AI stack into focused folders so each
concern is readable on its own.

## Layout

- `base/namespaces/`
  - child-cluster namespaces only
- `base/core/`
  - KubeVirt, CDI, and GPU Operator CRs
- `base/dpf/`
  - DPF control-plane and provisioning objects
- `base/dpu-services/templates/`
  - reusable DPU service template definitions
- `base/dpu-services/configurations/`
  - service runtime configuration
- `base/dpu-services/deployments/`
  - deployment compositions
- `overlays/vast/`
  - VAST-specific storage patches
- `overlays/prod-vast/`
  - production entrypoint using the VAST overlay

## Review guidance

When reviewing changes:
1. check `dpf/` for control-plane/provisioning changes
2. check `dpu-services/templates/` for service-shape changes
3. check `dpu-services/configurations/` for behavior changes
4. check `dpu-services/deployments/` for rollout/composition changes
5. check overlays for backend-specific changes only
