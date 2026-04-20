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
# VAST multipath + Arm + DPDK notes

This variant makes three changes explicit.

## 1. VAST POSIX path uses multipath NFS

The VAST overlay now patches the virtio-fs service configuration to use VAST NFS
with multipath enabled via:

- `mountOptions:`
  - `vers=3`
  - `remoteports=dns`

That is based on the VAST NFS client multipath mechanism, where `remoteports=...`
or `remoteports=dns` allows a single mount to spread traffic across multiple
endpoint IPs.

## 2. DPU services are explicitly Arm

All DPU service templates now set:

- `serviceDaemonSet.nodeSelector.kubernetes.io/arch=arm64`

This keeps the service intent explicit in Git.

## 3. HBN includes explicit DPDK / OVS config

The HBN service configuration includes an `ovs.rawConfigScript` section that
models the DPDK-based OVS setup shown in NVIDIA's HBN zero-trust RDG.
