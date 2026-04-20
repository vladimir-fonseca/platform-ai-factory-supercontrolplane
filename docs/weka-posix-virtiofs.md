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
# WEKA POSIX client over virtio-fs

This package includes an optional WEKA overlay for shared POSIX storage presented to guest workloads through the existing `snap-virtiofs` DPU service.

## Overlay paths

- `managed-addons/nvidia-ai-stack/overlays/weka/`
- `managed-addons/nvidia-ai-stack/overlays/prod-weka/`

## Design intent

The base package keeps `snap-virtiofs` backend-neutral so the same DPU service model can support different shared-filesystem providers.

The WEKA overlay specializes that generic configuration by setting:

- `backend: weka`
- WEKA management hosts
- WEKA filesystem name
- WEKA client mount path
- WEKA-specific mount options

## Main patched object

- `managed-addons/nvidia-ai-stack/overlays/weka/patch-snap-virtiofs-weka.yaml`

## What to replace for a real environment

Replace the placeholder values before deploying:

- `weka-mgmt-a.storage.example.com`
- `weka-mgmt-b.storage.example.com`
- `ai-shared`
- `/mnt/weka/ai-shared`
- any WEKA client mount options

## Relationship to other storage overlays

- `overlays/vast/` remains the VAST-specific shared POSIX overlay
- `overlays/weka/` is the WEKA-specific shared POSIX overlay
- both reuse the same `snap-virtiofs` service template and differ only in the backend-specific configuration patch

## Operational note

Validate the exact WEKA client packaging and supported mount options for the DPU service image and target operating environment before using this overlay in production.

## ARM-core pinning and performance defaults

The `snap-virtiofs` template is now explicitly biased toward DPU-side ARM execution and higher-performance shared POSIX service handling.

What is set in the template:

- `kubernetes.io/arch: arm64` node selection
- fixed CPU and memory requests/limits for the service daemonset
- `cpuPinning: true`
- `dedicatedCores: 4`
- `irqAffinity: true`

What is set in the WEKA overlay:

- `num_cores=4`
- `core_ids=0,1,2,3`
- `io_threads=4`
- `dedicated_mode=true`

These are package defaults intended to make the WEKA virtio-fs path clearly performance-oriented rather than minimal. Replace them with the exact values validated for your DPU core budget and WEKA client/runtime combination before production use.
