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
# SNAP block mode comparison

This package keeps two explicit DPU-side block-device emulation choices.

## `snap-block-nvme`
Use when you want:
- NVMe-emulated block presentation
- an NVMe-oriented remote-storage path
- a more storage-centric performance-focused block mode

## `snap-block-virtio-blk`
Use when you want:
- virtio-blk-emulated block presentation
- explicit guest-visible virtio-blk style semantics at the DPU service layer
- a simpler compatibility-oriented block mode

## Important distinction

These are DPU-side service choices.
They are separate from the KubeVirt VM disk bus decision.
