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
# KubeVirt feature expansion

## Intent

This note records the KubeVirt feature gates enabled in this package and the rationale for each one.

## Enabled feature gates

```text
GPU
DisableMDEVConfiguration
LiveMigration
Passt
```

## Rationale

- `GPU`
  - enables direct GPU device assignment for accelerated VM workloads

- `DisableMDEVConfiguration`
  - keeps the package aligned to direct device assignment rather than legacy mediated-device configuration

- `LiveMigration`
  - supports maintenance workflows and VM mobility across nodes

- `Passt`
  - enables a cleaner networking path for VM scenarios that benefit from passt-based networking support
