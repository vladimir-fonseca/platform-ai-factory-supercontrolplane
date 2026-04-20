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
# DPF profile sizing and flavor model

## Intent

This note explains why the shared DPF base keeps conservative fallback values while the active overlays replace them with family- and site-oriented examples.

## Shared-base posture

The shared base still carries:

- `maxNodes: 16`
- `dpuFlavor: default`

Those are fallback values only. They exist so the authored objects remain readable in isolation, but they are not the intended active values for the named overlays.

## Overlay posture

The active overlays now replace the base placeholders as follows:

```text
Overlay    maxNodes  dpuFlavor
---------  --------  -------------------
dev        8         hopper-dev
prod       32        blackwell-prod
prod-vast  32        blackwell-prod-vast
prod-weka  32        blackwell-prod-weka
```

## Why this is better

This is a better model than hardcoding a single `maxNodes` and `dpuFlavor` in the shared base because:

- development and production do not share the same scale target
- the flavor name should reflect the intended hardware family or operational profile
- backend-specific production overlays can carry their own profile identity
- the base stays neutral while the overlays express real deployment intent

## Environment note

These flavor names are example profile names, not a guarantee that the target DPF release already defines them.

Before deployment, replace them with the exact flavor names validated for the DPF release and BlueField firmware combination in use.
