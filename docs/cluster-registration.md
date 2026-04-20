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
# Cluster Registration Pattern

## Intent

A managed cluster must be known to Argo CD before `ApplicationSet` can target it.

## Simplified flow

1. CAPI creates the child cluster.
2. A kubeconfig secret becomes available in the management cluster.
3. An onboarding Job or external controller reads that kubeconfig.
4. The onboarding flow creates an Argo CD cluster secret.
5. The cluster secret includes labels such as:
   - `platform-profile=nvidia-ai`
   - `gpu-family.nvidia.com/name=h100`
6. ApplicationSet sees the new cluster and renders the addon application.

## Why this repo uses a Job example

In real deployments, onboarding is often handled by:

- an external automation controller
- a GitOps controller with cluster-registration logic
- a pipeline step
- a custom operator

The Job pattern in this repo is intentionally simple and visible.
