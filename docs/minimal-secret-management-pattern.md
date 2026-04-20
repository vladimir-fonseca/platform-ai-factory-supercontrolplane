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
# Minimal secret-management pattern

## Intent

This note records the minimum secret-management pattern expected by this package family.

The package does not attempt to solve enterprise secret management completely. It does provide a minimal, reviewable pattern so operators know which secret boundaries exist and where environment-specific material should be injected.

## Pattern

The package uses three secret classes:

1. management-plane Argo CD repository credentials
2. infrastructure credentials such as BMC access in the inventory layer
3. child-cluster pull or platform credentials such as OCI registry access

The recommended minimum pattern is:

- keep Git-tracked secret templates in `secret-management/templates/`
- never store real secret values in the package repository
- create environment-specific real Secrets out-of-band or through an external secret controller
- keep the Secret names stable so the authored manifests can reference them predictably

## Template coverage in this package

The package now ships templates for:

- Argo CD repository credentials
- an example BMC credential Secret
- an example OCI registry pull Secret

These templates are placeholders only. Replace all example values before any environment use.

## Recommended next step

For a stronger production posture, move from these templates to one of:

- External Secrets Operator
- Sealed Secrets
- Vault-backed secret injection
- an environment-specific secret creation pipeline

This document only establishes the minimum package-side pattern.
